---
layout: post
title: "[Docker] 백엔드는 언어라면, 인프라는 도커,,"
date: 2025-09-27 16:18:01 +0900
categories: [DevOps, infra]
tags: [container, docker]
---

## OCI? 런타임과 레지스트리가 호환 가능하게 해준 햄들
- 2015년, 리눅스 재단 산하에 설립된 단체
- 목적: 컨테이너 이미지와 런타임, 레지스트리 포맷을 표준화
- 배경:
  - 초기에는 컨테이너 기술과 이미지 포맷이 Docker 고유 포맷 중심
  - 여러 벤더와 생태계가 확장되면서, 모두가 쓸 수 있는 개방형 표준 필요
  - Docker 포맷과 아키텍처가 유용하여 표준으로 채택됨
  - 컨테이너 생태계가 표준화 + 확장성을 갖추게 됨

### OCI 주요 표준

| 표준                        | 역할                                                                   |
| ------------------------- | -------------------------------------------------------------------- |
| **OCI Image Spec**        | 컨테이너 이미지 포맷 표준화<br>– 이미지가 계층 구조로 어떻게 저장되는지, 메타데이터 구조 정의              |
| **OCI Runtime Spec**      | 컨테이너 실행 시 런타임이 지켜야 할 규칙<br>– 예: `runc`, `containerd` 같은 런타임이 따르는 API |
| **OCI Distribution Spec** | 컨테이너 이미지를 레지스트리에 저장/전송하는 방식 표준화<br>– HTTP API 기반                     |



## 메모

### login
```shell
docker login <harbor-hostname>[:port] [-u <username> {-p <password> | --password-stdin}]
```
- `-p <PASSWORD>`: 비밀번호가 히스토리나 프로세스 목록(ps 등) 에 노출될 수 있어서 보안에 취약
- `--password-stdin`: 비밀번호를 표준 입력(stdin)으로 전달

### tag
- 이미지에 이름 추가
```shell
docker tag <기존이미지[:태그]> <새이미지[:태그]>
```
- push 하려면 새이미지 이름은 다음 형식을 따라야 함
```shell
docker tag <image_name[:tag]> <registry>/<namespace or project>/<image>:<tag>
```

### push
- 도커 레지스트리에 push하려면 정해진 주소 형식을 따라야 함
```shell
docker push <registry>/<namespace or project>/<image>:<tag>
```


