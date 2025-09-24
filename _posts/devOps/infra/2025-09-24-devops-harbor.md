---
layout: post
title: "[Harbor] 구축하려면 helm을 알아야 하네요"
date: 2025-09-24 23:53:38 +0900
categories: [DevOps, infra]
tags: [image-registry, harbor]
---
사실 Harbor는 구축까지 깊게 팔 필요는 없을 것 같읍니다. But, Helm/k8s 예습할 겸 정리해보겠습니다.

## Harbor?

---
&nbsp; Harbor는 컨테이너 이미지 레지스트리 소프트웨어로, Docker Hub 역할을 한다고 볼 수 있다.
보통 사내에서 프라이빗하게 도커 이미지 레지스트리로 많이 사용한다.

&nbsp; 하나의 프로그램이 아니라 여러 컨테이너로 구성되어 있는데, 컨테이너 이미지들은 Docker Hub에 `goharbor/*` 형태로 공개되어 있다.
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
&nbsp; VM이나 서버에 직접 구축하는 경우는 [harbor-offline-installer-vX.Y.Z.tgz](https://github.com/goharbor/harbor/releases)같은 `tarball`을 
다운로드 받아야 한다.

1. 압축 해제
   - `tar -xvzf [다운한 tarball]`
2. 설치 준비
   - harbor 패키지 내 `harbor.yml.tmpl` 수정해 `harbor.yml`로 저장 <br>
      _(hostname, harbor_admin_password, ssl 설정 등)_
   - `./prepare` 실행 후 docker-compose.yml 확인 <br>
       _(harbor.yml을 읽어서 docker-compose.yml 등을 실제 값으로 치환)_
3. Harbor 설치
   - install.sh 스크립트 실행 <br>
     _(내부적으로 docker-compose up -d를 실행)_
4. 접속 확인

&nbsp; 관리는 docker로 하면 되며, offline installer는 Harbor 이미지까지 다 포함돼 있어서 인터넷 없어도 설치 가능하다고 한다.

<br>

### (추천) Kubernetes + Helm 방식

&nbsp; 먼저 쿠버네티스 클러스터를 준비하고, helm CLI를 설치한다. `kubectl`로 클러스터에 접근할 수 있어야 한다.
  
1. Harbor Helm Repo 추가
   ```shell
   helm repo add goharbor https://helm.goharbor.io
   helm repo update
   ```
   - helm 차트는 보통 <repo alias>/<chart name> 형식을 따른다. 해당 repo 별칭을 goharbor로 지정했다.
   - 이제 goharbor/harbor를 통해 harbor repository에서 harbor chart를 내려받을 수 있다. <br>
2. Helm 및 K8s 설정
   - values.yml 수정
      - 기본 설정을 커스터마이징한다.<br>
        _(expose.ingress.hosts.core, harborAdminPassword)_
   - 네임스페이스 생성
     - ex. `kubectl create namespace harbor`
3. Helm Chart로 Harbor 설치
   ```shell
   helm install [k8s 생성될 release 명] goharbor/harbor -n harbor -f my-values.yaml
   ```
   - 최종 Kubernetes manifest(Deployment, Service, PVC 등)를 생성
   - manifest에 정의된 Docker Hub의 goharbor/ 이미지를 내려받아 Pod 실행
4. 설치 확인 및 접속 확인

<br>

## 사용

---