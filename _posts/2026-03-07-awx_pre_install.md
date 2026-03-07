---
title:  "[AWX] AWX 설치 환경 구성"
excerpt: "AWX 컨테이너 이미지를 빌드하기 위한 환경을 생성하는 방법"
header:
  overlay_color: "#333"
categories:
  - CI/CD
tags:
  - Ansible
  - AWX
last_modified_at : 2026-03-07
last_modified_at_2 : 2026-03-07
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# 개요

## 주의 사항
- 2026.02.28 기준 설치 및 실행 확인
- awx 설치 시 18.0 버전부터 AWX Operator 를 이용하는 것을 권장한다. 이 경우 쿠버네티스 환경에서만 설치가 가능하다.
- 다행히도 docker compose 를 이용하여 도커 환경과 minikube 환경에서도 설치할 수 있도록 지원해준다. 다만 이 경우 개발/테스트 용도로 사용하는 것을 권장하며 해당 환경들을 공식적으로 배포하진 않고 있다.
  - [참고](https://github.com/ansible/awx/blob/devel/INSTALL.md)
- 도커 환경에서 AWX 설치 시 가장 문제가 되는 것은 AWX 컨테이너 빌드 시 필요한 패키지들끼리의 호환성 문제로 인해, 빌드하는 시점에 따라 내용물이 달라지면서 동작이 제대로 안 되는 상황이 발생한다.
  - AWX 설치 시 여러 패키지 및 라이브러리를 사용하는데 pip를 이용하여 설치하는 파이썬 패키지를 제외한 대부분의 패키지들이 고정된 버전을 사용하지 않는다.
  - git을 사용하여 특정 브랜치 (devel) 를 통해 소스를 다운로드하는 경우
    - 최신 버전의 소스이므로 빌드 시 AWX 버전이 낮을 경우 호환성 문제가 발생한다.
    - ansible/system-certifi
    - ansible/ansible-runner
    - ansible/python3-saml
    - ansible/django-ansible-base
  - AWX의 기본 컨테이너 이미지인 CentOS Stream 9의 근본적인 문제도 있다.
    - CentOS 의 정책이 변경되면서 CentOS Stream 으로 이름 변경과 함께 RHEL 얼리 액세스 느낌으로 변경되었다.
    - AWX 빌드 시 가장 문제가 되는 건 특정 버전의 패키지를 dnf 를 통해 설치할 수 있어야 하는데 CentOS Stream의 리포지토리에서는 최신 버전의 rpm 파일만 제공한다. (AWX 빌드 시 openssl, rsyslog 의 경우 특정 버전을 요구하게 됨)
  - 최대한 호환이 맞는 버전들을 특정해서 컨테이너 이미지를 빌드해야만 이후 AWX 사용에 문제가 없어진다.
- docker 환경에서 AWX 컨테이너 이미지를 생성을 위해 ansible 과 docker compose 를 이용한다. 여기에서는 이미지를 빌드하기 위한 환경을 구성하는 방법을 정리한다.


## 빌드 환경
- Rocky Linux 9.7
- Docker 26.1.4
- Docker-compose 2.35.1
- Python 3.9.25 (dnf로 설치)

- ansible 7.5.10
- git

# 환경 구성

## 필수 패키지 설치
```bash
dnf install python3 python3-pip git
```

## 필수 파이썬 패키지 설치
```bash
pip install virtualenv
pip install virtualenvwrapper
```

## virtualenv 를 사용하기 위한 환경 설정
- ~/.bashrc 에 아래의 내용을 추가

```bash
# 파이썬을 yum 또는 dnf 로 설치했을 경우
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3.9
source /usr/local/bin/virtualenvwrapper.sh
```

- bashrc 설정 이후 변수 적용을 위해 ssh 재접속 진행

## 가상 환경 생성
```bash
# 파이썬을 yum 또는 dnf 로 설치했을 경우
mkvirtualenv -p python3.9 awx
```

## awx 빌드 경로 생성
```bash
(awx) [jslim@jslim106 ~]$ mkdir awx-builder
(awx) [jslim@jslim106 ~]$ cd awx-builder
(awx) [jslim@jslim106 awx-builder]$ setvirtualenvproject
```

## awx 빌드 시 필요한 패키지 설치
```bash
(awx) [ahrah@jslim106 awx-builder]$ pip install ansible==7.5.0
(awx) [ahrah@jslim106 awx-builder]$ pip install docker
```