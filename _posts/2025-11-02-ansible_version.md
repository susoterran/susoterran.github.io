---
title:  "[Ansible] Ansible 버전별 호환 관계 정리"
excerpt: "ansible 버전 호환 관계"
header:
  overlay_color: "#333"
categories:
  - CI/CD
tags:
  - Ansible
last_modified_at : 2025-11-03
last_modified_at_2 : 2025-11-03
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# 개요

Ansible은 프로비저닝, 구성 관리, 애플리케이션 배포, 오케스트레이션 등 IT 인프라에 대한 자동화 기능을 지원해주는 툴이다. 이렇게 많은 기능을 지원하기 위해 Ansible 내부는 여러 개의 구성요소를 포함하고 있다. 이번 문서에서는 Ansible을 사용하면서 자주 보게 되는 Ansible 내부 구성요소와 각 버전별 호환 관계에 대해 알아본다.

<br>

# ansible과 ansible-core

Ansible을 Ansible 환경을 하나로 묶은 패키지로 본다면 Ansible-Core는 Ansible 언어, 런타임, 내장 모듈 세트 등을 가지고 있는 Ansible 내의 핵심 프레임워크라고 볼 수 있다.

마치 Ansible은 리눅스 배포판, Ansible-Core 를 커널에 비유할 수 있다.
리눅스 커널이 C/C++으로 작성된 것과 마찬가지로, Ansible-Core는 파이썬으로 작성되어 있다.
이는 리눅스 배포판의 버전에 따라 지원하는 리눅스 커널과 C 컴파일러 및 라이브러리의 버전이 달라지듯이 Ansible의 버전에 따라 내부적으로 사용하는 Ansible-Core 버전과 시스템에 요구하는 파이썬 버전이 달라진다.

<br>

# ansible과 python

Ansible 내부적으로 모든 로직은 파이썬으로 구현되어 있고, 원격 서버의 파이썬 환경을 통해 수행된다.

Control Node에서 Ansible 플레이북을 실행하면 SSH를 통해 Managed Nodes에 접속하고, 파이썬으로 구현된 모듈과 플러그인을 Managed Nodes로 복사 후 파이썬 환경을 통해 실행한다. 기본적으로 자주 사용되는 리눅스 배포판에는 SSH 서버와 파이썬이 설치되어 있기 때문에, Ansible이 agentless 의 이점을 가질 수 있다 (Windows의 경우 Windows에서 지원하는 WinRM과 PowerShell 환경을 사용하기 때문에 마찬가지로 agentless의 이점을 가진다)

따라서 Ansible과 Ansible-Core의 버전에 따라 Control Node 에 필요한 파이썬 버전과 Managed Nodes 에 필요한 파이썬 버전이 달라진다.

Ansible을 사용하기 전에 현재 인프라 환경에서 사용하고 있는 파이썬 버전들을 확인하고 이에 맞춰 사용할 Ansible 버전을 선택해야 한다.

<br>

# 버전별 호환 관계

현재 버전의 지원 상태와 신규 버전의 정보가 업데이트 될 수 있으니 아래의 공식 문서를 확인하는 것이 좋다.
- [https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#release-and-maintenance](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#release-and-maintenance)

| ansible community  version |      지원 상태      | ansible-core version |      Control node Python     |                  Managed node Python                  |
|:--------------------------:|:-------------------:|:--------------------:|:----------------------------:|:-----------------------------------------------------:|
| 12.0.0                     | 개발 중             | 2.19                 |                              |                                                       |
| 11.x                       | 현재                | 2.18                 | Python 3.11 - 3.13           | Python 3.8 - 3.13 PowerShell 5.1                      |
| 10.x                       | 2025/10/7 이후 중단 | 2.17                 | Python 3.10 - 3.12           | Python 3.7 - 3.12 PowerShell 5.1                      |
| 9.x                        | 2025/9/13 이후 중단 | 2.16                 | Python 3.10 - 3.12           | Python 2.7 Python 3.6 - 3.12 Powershell 5.1           |
| 8.x                        | EOL                 | 2.15                 | Python 3.9 - 3.11            | Python 2.7 Python 3.5 - 3.11 PowerShell 3 - 5.1       |
| 7.x                        | EOL                 | 2.14                 | Python 3.9 - 3.11            | Python 2.7 Python 3.5 - 3.11 PowerShell 3 - 5.1       |
| 6.x                        | EOL                 | 2.13                 | Python 3.8 - 3.10            | Python 2.7 Python 3.5 - 3.10 PowerShell 3 - 5.1       |
| 5.x                        | EOL                 | 2.12                 | Python 3.8 - 3.10            | Python 2.6 - 2.7 Python 3.5 - 3.10 PowerShell 3 - 5.1 |
| 4.x                        | EOL                 | 2.11                 | Python 2.7, Python 3.5 - 3.9 | Python 2.6 - 2.7 Python 3.5 - 3.9 PowerShell 3 - 5.1  |

<br>