---
layout: post
title: "[Gitlab-CI/CD] gitlab-ci.yml 작성하기"
date: 2025-10-26 15:07:29 +0900
categories: [ Devops, CI/CD ]
tags: [ ci/cd-tools, gitlab-ci/cd ]
---


&nbsp; 크게 전체 흐름(Pipeline)과 각 작업(Job)을 정의할 수 있다.


- `stages`: 실행 순서를 정의 (ex. build → test → deploy)
- `job`: 각 stage 안에서 수행할 작업
- `script`: 실제 실행 명령
- `artifacts`: 다음 job으로 넘길 결과물
- `only/rules`: 특정 브랜치/조건에서만 실행

<br>

## 전체 흐름: 루트 `.gitlab-yml`

---




## 옵션

---

### Job

| 그룹                | 옵션                               | 설명                                           |
|-------------------|----------------------------------|----------------------------------------------|
| 기본                | `stage`                          | Job이 속하는 stage 지정                            |
|                   | `script`                         | 실제 실행 명령어                                    |
|                   | `tags`                           | 특정 Runner 지정                                 |
|                   | `rules` / `only` / `except`      | 조건부 실행                                       |
| artifacts / cache | `artifacts`                      | Job 완료 후 저장할 파일/디렉토리                         |
|                   | `artifacts:paths`                | 저장할 경로 지정                                    |
|                   | `artifacts:expire_in`            | 보관 기간 설정                                     |
|                   | `cache`                          | Job 간 공유할 캐시 디렉토리                            |
|                   | `dependencies`                   | 이전 Job artifacts 가져오기                        |
| pipeline 제어       | `needs`                          | 다른 Job 완료 후 실행                               |
|                   | `allow_failure`                  | 실패해도 pipeline 계속 진행 여부                       |
|                   | `retry`                          | 실패 시 재시도 횟수                                  |
|                   | `timeout`                        | Job 최대 실행 시간                                 |
|                   | `when`                           | 실행 시점 (`on_success`, `on_failure`, `always`) |
| 환경                | `image`                          | Docker 컨테이너 이미지 지정                           |
|                   | `services`                       | 함께 실행할 서비스 컨테이너 지정                           |
|                   | `before_script` / `after_script` | Job 실행 전/후 명령어                               |
|                   | `variables`                      | Job 전용 환경 변수                                 |
| 병렬 / 전략           | `parallel`                       | 동일 Job 병렬 실행                                 |
|                   | `resource_group`                 | Job 간 리소스 공유 / 직렬 실행                         |
|                   | `interruptible`                  | 새 pipeline 실행 시 중단 여부                        |
| 기타                | `extends`                        | 다른 Job 상속                                    |
|                   | `environment`                    | 배포 환경 (staging/prod) 지정                      |
|                   | `release`                        | 릴리즈 메타데이터 생성                                 |
|                   | `coverage`                       | 테스트 커버리지 추출                                  |


### pipeline

---
- root `.gitlab-ci.yml`에 정의한다
- `stages` 순서 정의는 필수, 나머지는 필요에 따라 선택적
- `include` 는 같은 이름의 job이 다른 설정을 가지고 있으면 override 함.
- 
| 옵션                               | 설명                                                        |
|----------------------------------|-----------------------------------------------------------|
| `stages`                         | Pipeline에 존재하는 stage 순서 정의                                |
| `default`                        | 모든 Job에 공통 적용할 기본 옵션 설정 (ex: image, before_script, retry) |
| `variables`                      | 전체 Pipeline에 적용할 환경 변수                                    |
| `workflow`                       | Pipeline 실행 조건 (ex: 특정 브랜치/태그에서만 실행)                      |
| `include`                        | 외부 YAML 파일 포함 (다른 .yml 파일 재사용)                            |
| `image`                          | Pipeline 전체 기본 이미지 (Job별 override 가능)                     |
| `services`                       | 전체 Pipeline 기본 서비스 컨테이너                                   |
| `cache`                          | 전체 Pipeline 공통 캐시                                         |
| `before_script` / `after_script` | 전체 Job 공통 스크립트                                            |
| `schedules`                      | 정기 Pipeline 실행 스케줄링 (GitLab UI에도 가능)                      |


<br>


