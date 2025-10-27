---
layout: post
title: "[GitLab-CI/CD] GitLab start"
date: 2025-09-23 00:17:00 +0900
categories: [ DevOps, CI/CD ]
tags: [ ci/cd-tools, gitlab-ci/cd ]
---

&nbsp; GitLab 서버에 해야하는 작업은 `.gitlab-ci.yml`로 정의한다. 실제 작업하는 Runner의 설정과 Job 실행 환경은 `config.toml`에 정의한다.

<br>

## GitLab 서버: "인력소"

---
&nbsp; GitLab은 SaaS 형태로도 제공되고, 기업 내부망에 직접 설치해서 사용하는 On-premises 방식으로도 제공된다.
SaaS의 경우 GitLab에서 기본적으로 제공하는 Shared Runner를 바로 사용할 수 있고, On-premise는 Runner를 직접 설치하고 등록해야한다.

&nbsp; Gitlab 저장소에 `.gitlab-ci.yml`을 포함한 소스를 push하면, GitLab이 해당 파일을 읽어 CI/CD 파이프라인을 트리거한다.
`.gitlab-ci.yml` 파일 안에는 Job(해야 할 작업)과 실행 순서가 정의된다.

<br>

## GitLab Server ↔︎ GitLab Runner 연결

---
&nbsp; 서버와 러너는 register(등록)을 통해 연결할 수 있다.

&nbsp; 등록 이후 Runner는 Job 요청을 long-poll(주기적 조회) 방식으로 받는다.
Runner가 요청을 보내면 Server는 Job이 없으면 연결을 일정 시간 유지하며, Job이 생기면 즉시 전달한다.<br>
&nbsp; 즉, GitLab Server가 Runner에 명령을 “직접 push”하는 게 아니라,
Runner가 “할 일 있으면 주세요” 하고 GitLab Server에 물어보는 구조다.

&nbsp; Runner를 설치한 후 `gitlab-runner register` 명령어로 GitLab 서버와 연결하고, `config.toml`에 해당 Runner의 설정을 저장할 수 있다.

- `Coordinator URL`: 어느 GitLab 서버랑 통신할지
- `Registration token`: 어떤 프로젝트/그룹에 연결될 지 결정하는 토큰
- `Tags`: 특정 Runner에서만 실행할 Job을 구분하기 위한 라벨
- `executor`: Job 실행 환경: shell, docker, kubernetes 등

| Executor         | 설명                                            |
|------------------|-----------------------------------------------|
| `shell`          | 로컬 머신에서 바로 스크립트 실행 (격리/보안 문제)                 |
| `docker`         | 이미 존재하는 Docker 환경에서 컨테이너를 띄워 Job 실행           |
| `docker+machine` | 필요 시 새로운 VM을 자동 생성 후, 그 VM 안에서 Docker 컨테이너 실행 |
| `kubernetes`     | 쿠버네티스 클러스터에 Pod을 생성해서 Job 실행                  |

<br>

## GitLab Runner: "인부"

---
&nbsp; GitLab Server에서 Job 실행 요청이 확인되면 현재 Job을 실행할 수 있는 Runner가 동작한다.
Runner는 코드를 clone하고, 빌드/테스트/배포 등 `.gitlab-ci.yml`에 정의된 스크립트를 실행한 후,
Job의 실행 상태(success/fail)와 로그(output)를 GitLab에 반환한다.

> 이 때, executor 설정에 따라, pod를 띄워 각 job을 실행할 수도, 직접 실행할 수도 있다.

&nbsp; `.gitlab-ci.yml`에서 tags를 지정하면, 해당 태그를 가진 Runner만 그 Job을 실행할 수 있다.
조건에 맞는 Runner 중에서 현재 대기 중(idle)인 Runner가 Job을 가져간다.
태그가 지정되지 않은 Job은 태그 없는 Runner에서만 실행될 수 있다.

<br>

## 주의 사항

---

### 다른 프로젝트에 push

&nbsp; ArgoCD를 사용해 자동 배포를 구성한 경우, Helm chart가 있는 별도의 배포 리포지토리에 태그를 변경해 push해야 한다.
이런 경우, 해당 리포지토리에 접근할 수 있는 Personal Access Token을 CI/CD 변수에 등록해야 하며, <br>
Runner가 해당 리포지토리에 접근 가능한 네트워크 환경에 있어야 한다.

### rollback

&nbsp; GitLab CI/CD는 자동 롤백 기능을 기본으로 제공하지 않는다.
만약 롤백이 필요하다면, 다음과 같은 방법이 있다.

- when: on_failure 옵션을 쓰면, 이전 단계가 실패했을 때 이 Job이 실행된다.
- deploy.sh 스크립트 내에서 핼스체크를 수행하고, 문제 발생 시 롤백 처리를 직접 구현할 수 있다.

<br>

## CI/CD DSL(configuration keyword)

---

- https://docs.gitlab.com/ci/yaml/

<br>

---
<p> 
  <strong>👀 참고: </strong>
  <span itemprop="keywords">
    <a href="https://docs.gitlab.com/ci" class="page__taxonomy-item p-category">GitLab Docs</a><span class="sep">&nbsp; </span>
  </span>
</p>
