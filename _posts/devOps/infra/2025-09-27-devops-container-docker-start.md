---
layout: post
title: "[Docker] 백엔드는 언어라면, 인프라는 도커,,"
date: 2025-09-27 16:18:01 +0900
categories: [DevOps, infra]
tags: [container, docker]
---

## OCI?

- Docker가 처음 세상에 나왔을 때는 “컨테이너 이미지/런타임/레지스트리 포맷”이 다 Docker 독자 포맷

- “모두가 쓸 수 있는 개방형 표준을 만들자” 해서 2015년에 리눅스 재단 산하에 **OCI(Open Container Initiative)**라는 단체


OCI에서 정한 주요 표준

OCI Image Spec
→ 컨테이너 이미지 포맷을 표준화 (어떻게 계층 구조로 저장되고 메타데이터는 어떻게 기록되는지)

OCI Runtime Spec
→ 이미지를 실행할 때 컨테이너 런타임이 따라야 할 규칙 (예: runc, containerd 같은 런타임이 지켜야 할 API)

OCI Distribution Spec
→ 컨테이너 이미지를 레지스트리에 저장하고 주고받는 방식(HTTP API)


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


