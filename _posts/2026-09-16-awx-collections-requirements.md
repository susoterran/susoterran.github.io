---
title:  "[AWX] 프로젝트에 컬렉션 추가하기 - collections/requirements.yml"
excerpt: "AWX 프로젝트에 특정 컬렉션이 필요할 경우 추가하는 방법"
header:
  overlay_color: "#333"
categories:
  - CI/CD
tags:
  - Ansible
  - AWX
last_modified_at : 2026-09-16
last_modified_at_2 : 2026-09-16
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# collections/requirements.yml은 무엇인가

`collections/requirements.yml` 은 Ansible Galaxy의 의존성 선언 파일이며, "이 저장소의 코드를 실행하려면 어떤 컬렉션이 어느 버전 이상 필요한가"를 담고 있다. <br>
 파이썬의 `requirements.txt`나 Node.js의 `package.json` 를 떠올리면 쉽다. <br>
사용 목적은 재현성이다. 코드와 의존성 선언을 같이 리포지토리에 커밋하면, 개발 환경이 다른 곳에서도 컬렉션 버전을 동일하게 맞출 수 있다.

> `Collection` 은 플레이북, 롤, 모듈, 플러그인 등을 함께 묶어 배포하고 재사용할 수 있게 해주는 ansible의 표준 패키지 단위를 말한다. Python의 package 를 생각하면 된다.



# 용도
 
- 여러 사람이 `git clone` 을 통해 코드를 공유하는 상황에서 개발 환경 내 collection 을 동일하게 맞추고 싶을 때
- AWX-EE를 다시 생성하지 않고 collection 을 추가하거나 버전을 고정하고 싶을 때
- AWX-EE는 공통으로 쓰되 현재 프로젝트에서 다른 collection 조합이 필요한 경우

> AWX-EE (Execution Environment)는 AWX에 ansible을 구동하기 위한 컨테이너 이미지를 말한다.
> 자세한 설명은 다음 포스트에서 설명할 예정이다.



# 파일 포맷

최상위에 `collections`와 `roles` 를 사용할 수 있다.

```yaml
---
collections:
  - name: community.general
    version: ">=9.5.0"
    source: https://galaxy.ansible.com

roles:
  - name: geerlingguy.java
    version: "1.9.6"
```

버전을 따지지 않고 최신 버전만 설치하겠다고 한다면 아래와 같이 축약할 수도 있다 (공급망 공격에 취약할 수 있으니 권장하지 않음)
```yaml
---
collections:
  - community.general
```


## 각 항목별 사용하는 키의 의미

- name
  - FQCN (Fully Qualified Collection Name)의 첫 번째와 두 번째 항목을 사용한다. 
    - FQCN 이란 프로그래밍에서 클래스,함수,변수 등의 이름이 겹치지 않도록 전체 패키지명을 포함하여 고유하게 지정하는 이름을 말한다.
    - ansible에서는 다음의 형태를 사용한다 `<namespace>.<collection>.<module_name>`
  - ex: `community.general`, `community.docker`
- version: 버전을 지정한다. 생략하면 최신 버전으로 간주한다.
  - 다음과 같은 연산자를 사용하여 버전을 지정할 수 있다.
    - `*`: 최신 버전
    - `>=` · `>` · `<=` · `<`: 범위. 쉼표로 범위를 조합할 수 있다.
    - `!=`: 제외
    - `==` 또는 버전 문자열 단독: 해당 버전 그대로
- source: Galaxy 서버 URL 또는 `galaxy_server_list`에 정의한 서버 이름
- type: `galaxy`(기본), `git`, `file`, `url`, `dir`, `subdirs`
- signatures: GPG 서명 파일 URL 목록. 설치 시 무결성 검증에 사용

```yaml
---
# type 사용 예제
collections:
  # git 저장소에서 직접 설치. version 에는 브랜치·태그·커밋 해시를 쓴다
  - name: https://github.com/organization/repo_name.git
    type: git
    version: devel

  # 미리 내려받은 tarball 에서 설치 (폐쇄망 반입)
  - name: /tmp/community-crypto-2.22.0.tar.gz
    type: file

  # 로컬 디렉터리에서 설치
  - name: ./my_namespace/my_collection/
    type: dir
```

<br>


# 어떻게 사용하는가

## Ansible

- 현재 ansible 프로젝트 내에 아래의 명령어를 이용하여 `collections/requirements.yml` 에 선언한 컬렉션을 설치할 수 있다 (python의 `pip install -r requirements.txt` 과 사용법은 동일하다)
```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## AWX

AWX에서 ansible-playbook 의 실행 단위를 job이라고 하며, 모든 job은 Execution Environment(EE) 컨테이너 안에서 실행한다. <br>
job이 실행되는 시점에 collection이 컨테이너 안에 존재해야 하는데, 이를 설치하는 방법은 2가지다. <br>

1. EE 이미지를 직접 빌드하여 설치하는 방법
  - `ansible-builder` 로 이미지를 만들 때 `requirements.yml`을 읽어 설치한다.
  - 결과물이 이미지 레이어에 포함된다.
  - 다음 포스트에서 소개할 예정이다.
2. AWX 프로젝트를 동기화하는 시점에 설치하는 방법
  - AWX가 SCM에서 프로젝트를 받아올 때 `collections/requirements.yml`을 읽어 설치한다.
    - EE 이미지와는 별개로 AWX 내부의  임시폴더에 설치한다.
  - job 실행 직전에 job의 임시 폴더로 복사해서 사용한다.

<br>
두 가지 방법을 다 사용할 수도 있다 (앞서 `용도` 파트 중 2,3번이 이에 해당)
- 기존 EE 이미지와 `collections/requirements.yml` 에 동일한 컬렉션이 들어가 있는 경우 후자를 우선한다.
- 따라서 EE 이미지에 있는 컬렉션의 버전을 변경해서 사용하고 싶다면 ansible 프로젝트에 `collections/requirements.yml` 추가한 후 사용할 컬렉션을 선언하면 된다.
<br>

# 참고 자료

- https://docs.ansible.com/projects/ansible/latest/galaxy/user_guide.html
- https://docs.ansible.com/projects/ansible/latest/collections_guide/index.html
