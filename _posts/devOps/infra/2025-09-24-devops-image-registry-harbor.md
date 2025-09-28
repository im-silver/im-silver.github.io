---
layout: post
title: "[Harbor] 구축하려면 helm을 알아야 하네요"
date: 2025-09-24 23:53:38 +0900
categories: [ DevOps, infra ]
tags: [ image-registry, harbor ]
---

사실 Harbor는 구축까지 깊게 팔 필요는 없을 것 같읍니다. But, Helm/k8s 예습할 겸 정리해보겠습니다.

<br>

## Harbor?

---
&nbsp; Harbor는 OCI 호환 컨테이너 레지스트리 소프트웨어로, 쉽게 말해 사내에서 쓰는 Docker Hub라고 볼 수 있다.
보통 프라이빗 이미지 저장소 용도로 많이 사용한다.

&nbsp; 하나의 프로그램이 아니라 여러 컨테이너로 구성되어 있으며, 컨테이너 이미지들은 Docker Hub에 `goharbor/*` 형태로 공개되어 있다.
단순한 이미지 저장소 뿐만 아니라 다음의 기능도 제공한다.

- 프로젝트 단위 권한 관리
- 이미지 서명/취약점 스캐닝
- 리플리케이션 (다른 레지스트리와 동기화)
- 웹 UI 제공

<br>

## 구축: container Orchestration

---
&nbsp; Docker Hub에 이미지로 저장되어있는 Harbor container image를 무작정 내려받으면 될까?
아니다. 다음과 같은 이유로 보통 vm에 `tarball` 같은 패키지를 다운받아 설치하거나, helm chart를 사용해 쿠버네티스에 띄운다.

- Harbor는 여러 개의 마이크로서비스(컨테이너)로 구성
  - `Container Orchestration`이 필요
  - 컨테이너 런타임 환경(Docker, Docker Compose) 필요
- 해당 컨테이너들은 Linux 기반으로 빌드
  - OS: Linux

&nbsp; 단일 프로그램이 아니라 단순히 `docker run goharbor/harbor-core:latest` 처럼 개별적으로 실행하기는 어려울 듯 하다.
패키지를 다운받아 설치하는 방법과 쿠버네티스에 띄우는 방법을 간략하게 알아보겠다.

> 참고로 컨테이너는 nginx, core API, jobservice, registry, DB, redis 등이 있다.

<br>

### docker-compose 방식

&nbsp; VM이나 서버에 직접 구축하는 경우, [harbor-offline-installer-vX.Y.Z.tgz](https://github.com/goharbor/harbor/releases)같은
`tarball`을
다운로드 받아야 한다.

1. 압축 해제
   - `tar -xvzf <다운한 tarball>`
2. 설치 준비(설정 파일 준비)
   - harbor 패키지 내 `harbor.yml.tmpl` 수정해 `harbor.yml`로 저장 <br>
     _(hostname, harbor_admin_password, ssl 설정 등)_
   - `./prepare` 실행 후 docker-compose.yml 확인 <br>
     _(harbor.yml을 읽어서 docker-compose.yml 등을 실제 값으로 치환)_
3. Harbor 설치
   - install.sh 스크립트 실행 <br>
     _(내부적으로 docker-compose up -d를 실행)_
4. 접속 확인

&nbsp; 관리는 docker로 하면 되며, offline installer는 Harbor 이미지까지 포함하고 있어 인터넷 없어도 설치 가능하다고 한다.

<br>

### (권장) Kubernetes + Helm 방식

&nbsp; 먼저 쿠버네티스 클러스터를 준비하고, helm CLI를 설치한다. `kubectl`로 클러스터에 접근할 수 있어야 한다.

1. Harbor Helm Repo 추가
   ```shell
   # helm repo add <REPO_ALIAS> <repository>
   helm repo add goharbor https://helm.goharbor.io
   helm repo update
   ```
   - helm 차트는 `<repo-alias>/<chart-name>` 형식을 따른다. 해당 repo 별칭을 goharbor로 지정했다.
   - 이제 goharbor/harbor를 통해 harbor repository에서 harbor chart를 내려받을 수 있다. <br>
2. Helm 및 K8s 설정
   - values.yaml 수정해 기본 설정을 커스터마이징한다.<br>
     _(expose.ingress.hosts.core, harborAdminPassword)_
   - 네임스페이스를 생성한다.
     - ex. `kubectl create namespace harbor`
3. Helm Chart로 Harbor 설치
   ```shell
   # helm install <RELEASE_NAME> <repo-alias>/<chart-name> -n <namespace> -f <values-file>
   helm install myHabor goharbor/harbor -n harbor -f my-values.yaml
   ```
   - 최종 Kubernetes manifest(Deployment, Service, PVC 등)를 생성한다.
   - manifest에 정의된 Docker Hub의 goharbor/* 이미지를 내려받아 Pod가 실행된다.
   - helm option:
     - `-n`: namespace
     - `-f`: file
4. 설치 확인 및 접속 확인

<br>

## 사용

---

### push

1. 로그인
   ```shell
   docker login harbor.mycompany.com -u im-silver -p actuallyGold
   ```
   - Harbor는 Docker Registry v2 API + OCI Distribution Spec을 준수하므로, docker CLI로 로그인이 가능하다.
   - port는 기본적으로 443(HTTPS), 80(HTTP)이 할당된다.

2. 이미지 push
   ```shell
   # 이미지에 push할 수 있는 태그 추가
   docker tag api-server:1.0 harbor.mycompany.com/backend/api-server:1.0
   
   # push
   docker push harbor.mycompany.com/backend/api-server:1.0
   ```
  
   - Docker는 registry에 push할 때, 이미지 태그명이 다음 형식을 따라야 한다.
     - `<registry>/<namespace>/<image>:<tag>` <br>
       _<레지스트리 주소>/<조직|사용자|프로젝트 단위>/<이미지 이름>:<태그>_
   - 태그(push할 경로 지정) 추가 후, push한다.

<br>

### pull

1. 로그인
   ```shell
   docker login harbor.mycompany.com -u im-silver
   ```

2. 이미지 pull
   ```shell
   docker pull harbor.mycompany.com/backend/api-server:1.0
   ```
   - 도메인은 `<harbor-domain>/<project>/<repository>:<tag>` 형식을 따른다
   - 이미지 push 할 때 경로와 같다. Harbor의 domain이 registry 주소, project는 Harbor에서 만든 프로젝트다.
    repository가 프로젝트 안에 실제 이미지 이름이 된다.

<br>

### CI/CD & 쿠버네티스 연동

1. CI/CD 연동
   - GitLab CI/CD 

      ```yaml
      build:
      stage: build
      script:
        - echo "$HARBOR_PASSWORD" | docker login -u "$HARBOR_USER" --password-stdin harbor.mycompany.com
        - docker build -t harbor.mycompany.com/backend/api-server:$CI_COMMIT_SHA .
        - docker push harbor.mycompany.com/backend/api-server:$CI_COMMIT_SHA
      ```
     - 로그인 <br>
       _`GitLab 프로젝트 → Settings > CI/CD > Variables`에서 id, pw 등 환경 변수를 설정할 수 있다._
     - 현재 GitLab 소스 디렉토리에 있는 Dockerfile을 읽어서 새 이미지 생성<br>
        _`.`: Runner가 체크아웃한 소스 디렉토리_
     - Harbor에 push

2. 쿠버네티스 연동
   - Deployment

      ```yml
     #spec.template.spec.
      containers:
        - name: api-server
          image: harbor.mycompany.com/backend/api-server:1.0.0
      imagePullSecrets:
        - name: harbor-secret
      ```
   - Secret 생성:
      ```shell
      kubectl create secret docker-registry harbor-secret \
      --docker-server=harbor.mycompany.com \
      --docker-username=<user> \
      --docker-password=<pass>
      ```

---


