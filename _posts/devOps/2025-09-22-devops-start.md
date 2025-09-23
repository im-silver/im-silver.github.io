---
layout: post  
title: "[devops] 인프라 & CI/CD 흐름 정리"  
date: 2025-09-22 23:54:00 +0900
categories: [DevOps] 
tags: [overview]
---
## 서론

---
&nbsp; 내가 담당하는 서비스의 스펙은 다음과 같았다.
- 언어: Java, PHP
- DB: Altibase, MySQL, Elasticsearch, Redis
- 스케쥴: Crontab, Java 스케쥴링
- 메시징: RabbitMQ
- CI/CD: Shell Script, Jenkins(수동 빌드)
- 환경: Linux

<br>

&nbsp; 서비스가 구분되지 않고 퍼져있었으며, 프로젝트의 구조가 불분명해 소스 코드 파악이 어려웠다.<br>
- 추상화가 부족한 상태에서 수많은 정책을 주먹구구식으로 추가했고, 곳곳에 하드코딩이 남발되었다.
- 캡슐화가 되어 있지 않아 객체가 어디서 변경되는지 추적하기가 어려웠다.
- 테스트 코드는 관리되지 않아 오류 투성이고, 테스트 라이브러리가 아닌 단순 로그 출력으로 직접 확인한다.
- Redis는 캐시가 아닌 DB처럼 사용되었고, 다양한 DB를 하나의 트랜잭션으로 묶기 위해 코드가 매우 복잡했다.
- 배포 전 자동화된 테스트가 없어, 배포 이후에는 반드시 모니터링을 통해 장애 여부를 확인해야 했다.

<br>

&nbsp; 하울의 움직이는 성과 다를 바가 없다.
이 때문에 신규 입사자(2년 차 = 나ㅋ)는 수많은 규칙과 꼬여 있는 정책을 이해하느라 진이 빠진다. 
새로오신 본부장님은 이 성을 더 발전시키는 건 불가능 하다고 보시고, 완전히 갈아 엎자는 목표를 세우셨다.

인프라 관련 신규 스펙은 다음과 같다.
- CI/CD: GitLab CI/CD, ArgoCD
- 컨테이너 오케스트레이션: Kubernetes(K8s)
- 패키지 관리 및 배포: Helm
- 이미지 레지스트리: Harbor
- 서비스 메시: Istio

&nbsp; 유지보수 업무를 하며 안주했던 만큼 신규 스펙을 따라가는 게 쉽지 않다.
먼저 전체 흐름을 큰 그림으로 대충이라도 잡아두고, 부족한 부분은 차근차근 채워 나가려 한다.
신규 플랫폼의 인프라 흐름을 따라가보자.

<br>

## 1. gitlab-ci.yml: “Runner가 할 일 목록”

---
&nbsp; GitLab 이 해당 파일을 읽어 해야할 작업(Job)을 정의한다.
해당 Job을 실행할 수 있는 Runner가 `.gitlab-ci.yml`에 정의된 빌드, 테스트, 배포 스크립트를 실행 후 로그와 성공 여부를 반환한다.

&nbsp; GitLab은 SaaS 형태로도 제공되지만, 기업 내부망에 직접 설치해서 사용하는 On-premise 방식으로도 제공된다.
SaaS의 경우 GitLab에서 기본적으로 제공하는 Shared Runner를 사용할 수 있지만, 우리는 내부망에 설치했기 때문에 Runner 따로 설치해 사용한다.

회사의 `.gitlab-ci.yml`에 정의된 파이프라인은 대략 다음과 같다.
- Test
- Gradle Build
- Docker BUild
  - Harbor에 push
- Manifest 프로젝트에 Push

## 2. Harbor: "사내 이미지 저장소"

---
내부 컨테이너 이미지 레지스트리 (Docker Hub 같은 역할)
