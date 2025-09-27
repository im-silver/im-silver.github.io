---
layout: post
title: "Bash Convention"
date: 2025-09-27 15:27:48 +0900
categories: [ etc ]
tags: [ convention ]
---

충격적인 사실 발견

## 문서에도 컨벤션이 있읍니다.

---
```shell
git commit -m "MESSAGE"
```

```shell
kubectl get {pod|service|deployment}
```

```shell
helm install <RELEASE_NAME> <CHART>
```

```shell
curl [options] <url>
ls [-al] [FILE...]
```

&nbsp; 생각 없이 보면서 사람 취향따라 바뀐다고 생각했다.
하지만 안일한 생각이었고? 가독성을 위해 컨벤션을 지켜 글을 작성하고자 한다.


## 대문자 vs 소문자

- 대문자: 사용자가 꼭 바꿔 넣어야 하는 PLACEHOLDER (Upper Snake Case)
  - ex. `git commit -m "MESSAGE"`
- 소문자: 실제 값이 소문자로 많이 쓰이거나 일반적인 이름
  - ex. `/home/<username>`
  - `username`은 리눅스 계정명으로 보통 소문자로 정의되어 있음

뭔가 주관적인 기준이 들어가는 게 맘에 들지 않아 꺾쇠나 중괄호 내부에 사용 시, 다른 방식으로 쓰고자 한다.

### MY_CONVENTION
- 대문자 (Upper Snake Case)
  - 의미: 사용자가 새로 정해야 하는 값 → PLACEHOLDER
  - 특징: 새로 작성하기 때문에 오탈자 상관 없음
  - ex. `git commit -m "MESSAGE"` → "MESSAGE"는 내가 작성하는 메시지

- 소문자
  - 의미: 이미 존재하고, 실제로 정해져 있는 이름을 틀리지 않고 작성
  - 특징: 존재하는 리소스/파일/차트 이름이므로 오탈자 나면 안됨
  - ex. `/home/<username>` → 계정명이 `im-silver`라면 반드시 정확히 써야 함

따라서, `helm install <RELEASE_NAME> <chart>`이런 식으로 정리하겠다.


## 표기법

| 표기법      | 의미        | 예시 (문법)                                  | 예시 (실제 사용)                               | 
|----------|-----------|------------------------------------------|------------------------------------------| 
| `< >`    | 필수 인자     | `helm install <RELEASE_NAME> <chart>`    | `helm install my-harbor goharbor/harbor` | 
| `[ ]`    | 선택적 인자(옵션) | `curl [options] <url>`                   | `curl https://example.com`               | 
| `{ \| }` | 여러 값 중 하나 선택 | `kubectl get {pod\|service\|deployment}` | `kubectl get pod`                        | 
| `...`    | 반복 가능     | `cp <source>... <directory>`             | `cp a.txt b.txt /tmp/`                   | 

> 때때로 [...] 안에 ... 붙어서 여러 개 가능하다는 의미로도 쓰임

