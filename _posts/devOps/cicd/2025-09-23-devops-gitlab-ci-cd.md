---
layout: post  
title: "[GitLab CI/CD] GitLab start"  
date: 2025-09-23 00:17:00 +0900
categories: [DevOps,CI/CD]
tags: [gitLab-ci/cd,GitLab]
---

GitLab cicd를 공부하면, Github Action도 쉽게 파악할 수 있기를 기대합니다.ㅎ
<br><br>

# GitLab CI/CD
&nbsp; GitLab 서버에서 해야하는 작업은 `.gitlab-ci.yml`, 실제 작업하는 Runner의 스펙은 `config.toml`에 정의한다.


## GitLab 서버

---
&nbsp; GitLab은 SaaS 형태로도 제공되고, 기업 내부망에 직접 설치해서 사용하는 On-premise 방식으로도 제공된다.
SaaS의 경우 GitLab에서 기본적으로 제공하는 Shared Runner를 바로 사용할 수 있고, On-premise는 Runner를 직접 설치하고 등록해야한다.

&nbsp; Gitlab 저장소에 `.gitlab-ci.yml`을 포함한 소스를 push하면, GitLab이 해당 파일을 읽어 CI/CD 파이프라인을 트리거한다.
`.gitlab-ci.yml` 파일 안에는 Job(해야 할 작업)과 실행 순서가 정의된다.

<br>

## GitLab Runner

---
&nbsp; GitLab이 Job 실행 요청을 보내면 현재 Job을 실행할 수 있는 Runner가 동작한다.
Runner는 코드를 clone하고, 빌드/테스트/배포 등 `.gitlab-ci.yml`에 정의된 스크립트를 실행한 후, 
Job의 실행 상태(success/fail)와 로그(output)를 GitLab에 반환한다.

&nbsp; Runner를 설치한 후 `gitlab-runner register` 명령어로 GitLab 서버와 연결하고, `config.toml`에 해당 Runner의 설정을 저장할 수 있다.
- `Coordinator URL`: 어느 GitLab 서버랑 통신할지
- `Registration token`: 어떤 프로젝트/그룹에 연결될 지 결정하는 토큰
- `Tags`: 특정 Runner에서만 실행할 Job을 구분하기 위한 라벨
- `executor`: Job 실행 환경: shell, docker, kubernetes 등

&nbsp; `.gitlab-ci.yml`에서 tags를 지정하면, 해당 태그를 가진 Runner만 그 Job을 실행할 수 있다.
조건에 맞는 Runner 중에서 현재 대기 중(idle)인 Runner가 Job을 가져간다.
태그가 지정되지 않은 Job은 모든 태그 없는 Runner에서 실행될 수 있다.

<br>

## CI/CD DSL(configuration keyword)

---
- https://docs.gitlab.com/ci/yaml/

<br>

## 주의 사항

---
### 다른 프로젝트에 push
&nbsp; ArgoCD를 사용해 배포를 자동화해뒀다면 Helm chart가 있는 프로젝트에 태그를 변경하는 등 다른 프로젝트로 push 가 필요한 상황이 있다.
이 때, 권한이 있는 토큰을 CI/CD 변수에 넣어줘야하며, <br>
Runner가 내부망에 있다면 외부 GitLab.com이나 다른 GitLab 인스턴스에 접근 가능한 네트워크 환경이어야 한다.

### rollback
&nbsp; GitLab CI/CD는 자동 롤백을 지원하지 않는다.
만약 롤백이 필요하다면, 다음과 같은 방법이 있다.
- when: on_failure 옵션을 쓰면, 이전 단계가 실패했을 때 이 Job이 실행된다.
- deploy.sh 안에서 트랜잭션 처리나 헬스체크 후 롤백을 직접 구현할 수 있다.

---
<p> 
  <strong>👀 참고: </strong>
  <span itemprop="keywords">
    <a href="https://docs.gitlab.com/ci" class="page__taxonomy-item p-category">GitLab Docs</a><span class="sep">&nbsp; </span>
  </span>
</p>
