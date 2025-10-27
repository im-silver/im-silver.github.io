---
layout: post
title: "[Gitlab-CI/CD] gitlab-ci 잘 활용하기"
date: 2025-10-26 15:56:10 +0900
categories: [ Devops, CI/CD ]
tags: [ ci/cd-tools, gitlab-ci/cd, tip ]
---

실무용 포인트

민감 정보(Harbor, ArgoCD 토큰) → GitLab Variables로 관리

Job별 Runner 지정 가능 (tags)

실패 Job은 retry로 재시도

pipeline 흐름 시각화 용이 (needs, stages)


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
