---
title:  "[Ansible] Ansible 설치하기"
excerpt: "Ansible, Python, Ansible-code, AWX 간 버전 호환 비교"
header:
  overlay_color: "#333"
categories:
  - CI/CD
tags:
  - Ansible
last_modified_at : 2023-05-14
last_modified_at_2 : 2025-11-02
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---


# Ansible 설치하기

현재 Python이 설치된 환경에서 어떤 Ansible 버전을 설치할 수 있는지 확인해봤다.

이번엔 실제로 Ansible을 설치하는 과정에 대해 알아볼 것이다.

Ansible을 설치하는 방법은 2가지이다.
- 각 OS의 Package 매니저를 이용하여 설치하는 방법
- Python의 PIP를 이용하여 설치하는 방법

기본적으로 현재 환경에 파이썬이 반드시 있어야 함을 전제로 한다.

## OS의 Package 매니저를 이용한 방법

yum이나 apt를 이용하여 설치할 수 있으나 추천하진 않는다. 이럴 경우 현재 배포판에서 지원하는 Ansible 버전에 종속되기 때문에 원하는 Ansible 버전을 설치할 수 없기 때문이다.

- CentOS
```
yum install ansible
```

- Rocky Linux 9
```
dnf install ansible
```

- Ubuntu
```
apt install ansible
```


## Python의 Package 매니저를 이용한 방법

Ansible에서 필요로 하는 버전의 파이썬이 이미 설치되어 있어야 한다는 전제가 필요하다.

virualenv 로 가상 환경을 만드는 이유는 시스템에서 사용하는 파이썬과 구별하기 위함이다.

가상 환경없이 pip로 특정 모듈을 설치한다면 이는 전체 시스템에 적용된다.
만약 가상 환경을 만든다면 python, pip 등의 실행 파일과 메타 데이터 및 설치한 모듈들의 저장 위치가 기존 위치와 전혀 다른 곳에 생성된다.

가상환경 내에서 pip로 모듈을 설치하던 설정값을 변경해도 전체 시스템에는 영향이 미치지 않는다.


설치 환경
- Rocky Linux 9.6
- Python 3.9.23
- Ansible 7.5.10

Python의 virtualenv 모듈을 통해 Ansible 만을 위한 가상의 개발환경을 만들게 된다.

```
### 필요한 파이썬 패키지 설치
pip install virtualenv

### virualenv로 가상 환경 생성
mkdir /usr/local/ansible
cd /usr/local/ansible
python3 -m virtualenv ansible-7.5.0

### 가상 환경 실행
source ansible-7.5.0/bin/activate

# ansible 설치
pip install ansible==7.5.0
```

source 명령어 이후 부터 쉘 프롬프트에 (ansible-7.5.0)이 붙어있을 것이다.
이렇게 되어있다면 ansible-7.5.0이 설치된 가상의 개발환경에 사용자가 접근하게 된 것이다.


<br>
<br>

