---
layout: post
title: "[GitLab-CI/CD] GitLab start"
date: 2025-09-23 00:17:00 +0900
categories: [ DevOps, CI/CD ]
tags: [ ci/cd-tools, gitlab-ci/cd ]
---

&nbsp; GitLab 서버에서 해야하는 작업은 `.gitlab-ci.yml`, 실제 작업하는 Runner의 스펙은 `config.toml`에 정의한다.

<br>

## GitLab 서버

---
&nbsp; GitLab은 SaaS 형태로도 제공되고, 기업 내부망에 직접 설치해서 사용하는 On-premises 방식으로도 제공된다.
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
태그가 지정되지 않은 Job은 태그 없는 Runner에서만 실행될 수 있다.

<br>

## 🐳 이미지 버전 관리 전략

---
&nbsp; CI 파이프라인에서는 도커 이미지를 빌드하고 레지스트리에 저장하는 Job이 많다.
보통 이미지 태그를 통해 배포와 릴리즈를 관리하며, 운영에서는 SHA + SemVer, 테스트서버에서는 SHA + branch 조합으로 많이 사용한다.<br>
SHA, SemVer, branch 태깅 전략을 알아보자.

### 1. 커밋 SHA 태깅 (:$CI_COMMIT_SHA)

&nbsp; Git 커밋 해시 앞 7자리를 태그명으로 사용한다.
특정 커밋과 이미지가 1:1 매핑이 되어 디버깅과 롤백에 용이하며, GitLab CI/CD는 편의를 위해 $CI_COMMIT_SHA 변수를 제공한다.

  > SHA? "Secure Hash Algorithm" <br>
  > 입력 데이터를 고정 길이의 문자열(해시)로 변환하는 암호화 함수며 Git에서는 커밋 식별자로 사용한다.

### 2. SemVer

&nbsp; Semantic Versioning, 버전 번호 자체가 변경 내용을 추강화하기 때문에 의미 있는 버전 관리가 가능하다.
일반적으로는 `MAJOR.MINOR.PATCH`형식을 따른다.

| 구분        | 의미                  | 예시                         |
|-----------|---------------------|----------------------------|
| **MAJOR** | 큰 변경, 이전 버전과 호환성 깨짐 | 2.0.0 → 큰 기능 구조 변경         |
| **MINOR** | 새로운 기능 추가, 호환성 유지   | 1.2.0 → 기능 추가, 기존 기능 유지    |
| **PATCH** | 버그 수정               | 1.2.1 → 기존 기능에 영향 없는 버그 수정 |

QA/기획/운영팀 등 비개발자도 쉽게 이해할 수 있어야 하기 때문에, 운영에서는 SemVer 태그로도 관리하는 경우가 많다.

### 3. Branch

&nbsp; 브랜치 이름으로 태깅한다. 스테이징/QA 환경에서 브랜치별 컨테이너 독립 배포가 가능해 충돌 위험이 적다.

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
