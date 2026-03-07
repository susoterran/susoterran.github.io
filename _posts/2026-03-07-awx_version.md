---
title:  "[AWX] AWX 란 무엇인가"
excerpt: "awx에 대한 설명과 ansible 과의 관계"
header:
  overlay_color: "#333"
categories:
  - CI/CD
tags:
  - Ansible
  - AWX
last_modified_at : 2026-03-06
last_modified_at_2 : 2026-03-06
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# 개요

ansible 프로젝트는 yaml 기반으로 작성되는 코드들의 집합이며 플레이북, 인벤토리, 역할을 적절히 조합하여 Managed Node를 구성하는데에 주로 쓰인다.
단순히 인프라 환경 구성 뿐만 아니라 handler를 이용하여 플레이북 실행 결과에 대한 알림을 받을 수도 있고, 정기적인 배치 작업을 수행하거나 애플리케이션 배포하는 용도 등 활용법은 무궁무진하다.
다만 ansible 자체는 CLI 텍스트 기반으로 동작하기 때문에 사용하기 불편한 점이 있다.

AWX는 ansble 프로젝트 관리를 위한 웹 기반 사용자 인터페이스, REST API 및 작업 엔진을 제공하는 오픈 소스 툴이다.
웹을 통해 프로젝트 관리 뿐만 아니라 배치, 알림, 인증, 감사 등의 기능들을 제공해주기 때문에 ansible을 보다 쉽게 사용할 수 있게 된다.

<br>

# awx와 ansible-core 호환 관계 
- 2026.03.07 기준으로 작성
- AWX 18 이후부터 컨테이너 내부는 크게 ansible-ee (Execution Environments) 와 ansible-runner 로 구성된다.

<br>

## awx-ee
- ansible 기능을 통합하여 컨테이너 이미지로 빌드한 후 사용한다.
- 기존 AWX의 경우 내부에 ansible 버전이 사실상 하나로만 사용했으나, EE를 사용하게 되면 AWX 내에서 여러 ansible 버전을 사용할 수 있게 되었다.
- AWX에 설정한 실행환경에 맞는 이미지를 리포지토리로부터 다운받아 컨테이너 내부에서 podman를 통해 ee 컨테이너를 추가로 실행하는 구조이다.
- redhat에서 제공하는 기본 EE를 사용할 수도 있고, 직접 EE 이미지를 빌드하여 별도의 리포지토리로 관리하여 사용할 수도 있다.
- awx 컨테이너 내부에서 awx-ee가 podman 컨테이너로 동작하는 모습
  ```bash
  bash-5.1$ podman ps -a
  CONTAINER ID  IMAGE                          COMMAND               CREATED         STATUS         PORTS       NAMES
  ff82156fa3b8  quay.io/ansible/awx-ee:latest  ansible-playbook ...  47 seconds ago  Up 47 seconds              ansible_runner_263
  ```

## ansible-runner
- ansible을 추상화한 인터페이스 역할을 한다.
- 파이썬 라이브러리 형태로 존재한다.

<br>

## awx 버전간 호환 관계

- awx-ee의 버전별로 내부에 설치된 파이썬과 ansible-core 버전이 다르다.

| awx-ee 버전 |  릴리즈 일자  | ansible-core version |  Python version  |
|:----------:|:------------:|:--------------------:|:----------------:|
| 24.6.1     | 2024-07-03   | 2.15.12              | Python 3.9.19    |
| 24.0.0     | 2024-03-13   | 2.15.9               | Python 3.9.18    |
| 23.0.0     | 2023-08-30   | 2.15.3               | Python 3.9.17    |
| 22.0.0     | 2023-04-04   | 2.14.4               | Python 3.9.16    |

<br>

# awx-ee를 커스텀 빌드해야 하는 경우
1. ansible과 awx-ee의 ansible-core 버전이 일치하지 않는 경우
- ansible 개발 환경에서 ansible 13.2.0을 설치한다면 ansible-core 버전은 2.20이다.
- 하지만 현재 최신 버전인 24.6.1에서의 ansible-core는 2.15.12을 사용한다.
- ansible-core 버전이 다를 경우 ansible 프로젝트가 AWX 내에서 정상적으로 동작한다는 보장을 할 수 없다. 특히 새로 추가된 기능들은 사용할 수 없게 된다.
- 필자의 경우 ansible 7.5.0을 사용하다가 `ansible.builtin.meta: end_role` 를 사용하고자 했지만 ansible-core 2.18부터 추가된 기능이라 사용할 수 없었고, 이참에 최신 버전을 쓰자고 생각하여 13.2.0을 사용하게 되었다.
  - 이로 인해 awx-ee 이미지를 새로 빌드해야 했는데 이부분은 추후에 소개하도록 하겠다.

2. 필요한 collection이 awx-ee에서 제공되지 않는 경우
- ansible 환경에서는 ansible-galaxy 를 이용하여 필요한 collection을 설치할 수 있었다.
- 하지만 awx-ee 에서는 기본 제공되는 collection 외에는 추가 사용할 수 없다는 문제가 있다.
- 최신 awx-ee 에 왠만한 collection 들이 포함되어 있지만 커스텀 collection 을 작성해서 사용한다면 마찬가지로 awx-ee에서 사용할 수 없다.
- 이 경우 awx-ee 이미지를 새로 빌드하면서 필요한 collection을 ansible-galaxy를 통해 설치할 수 있다.

<br>