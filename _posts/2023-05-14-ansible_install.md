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
last_modified_at_2 : 2023-05-14
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# Ansible이란 무엇인가?

의외로 Ansible이 Hot한거 치곤 출판되어 나온 책은 많지 않다. 다행히도 Ansible이 계속 버전업을 해왔지만, 내재된 기본적인 개념이나 문법은 크게 변하진 않았다.

개인적으로 Ansible을 처음 공부할 때 "앤서블 철저 입문 (위키북스, 2017)"을 통해 기본 개념들을 알아나갔으며, 그 이후부턴 Ansible 공식 가이드 문서를 통해 각종 문법과 모듈 사용법을 공부했다.

본론으로 들어가서 Ansible에 대해 막상 설명하려고 보니 "자동화 설치 Tool" 이란 생각 밖에 안 떠오른다 (딱히 틀린 말은 아닌데, 그렇다고 맞다고 하면 그것도 좀...)

따라서 Ansible이 무엇이냐에 대한 내용은 내가 참고했던 자료들의 도움을 받고자 한다.

아래의 문서를 읽으면, Ansible의 대략적인 개념을 보는데 도움이 될 것이다.

[프로비저닝 자동화를 위한 Ansible AWX, 설치부터 엔터프라이즈 환경 적용까지 - 1](https://engineering.linecorp.com/ko/blog/ansible-awx-for-provisioning-1)

<br>
<br>

# Ansible과 Ansible-Core의 관계

Ansible을 Ansible 환경을 하나로 묶은 패키지로 본다면 Ansible-Core는 Ansible 언어, 런타임, 내장 모듈 세트 등을 가지고 있는 Ansible 내의 핵심 프레임워크라고 볼 수 있다.

같은 Ansible 친구인 듯 보여도 버전업은 따로 돌고 있다. 마치 커널과 리눅스 배포판의 버전이 다르게 관리되는 것과 비슷하다고 할까.

따라서 Ansible 버전에 따라 이에 포함된 core 버전이 달라진다.

추가적으로 core는 Python 기반으로 동작하는데, 설치 된 Python 버전에 따라 사용할 수 있는 ansible 및 ansible-core 버전이 달라진다.

이 버전 관계가 조금 복잡해보이긴 하는데 하나씩 정리해보겠다.

<br>
<br>

첫 번째는 각 ansible-core 에서 요구하는 Python 버전이다.

- Control node Python : Ansible 
- Managed node Python : 

|ansible-core Version|Control node Python|Managed node Python|
|---|---|---|
|2.11|Python 2.7, Python 3.5 - 3.9|Python 2.6 - 2.7, Python 3.5 - 3.9|
|2.12|Python 3.8 - 3.10|Python 2.6 - 2.7, Python 3.5 - 3.10|
|2.13|Python 3.8 - 3.10|Python 2.7, Python 3.5 - 3.10|
|2.14|Python 3.9 - 3.11|Python 2.7, Python 3.5 - 3.11|

단순하게 설명하면 2번째 열은 Ansible 환경이 구축된 노드의 파이썬 버전을 의미하며, 3번째 열은 Ansible 작업이 실제 동작하게 되는 노드의 파이썬 버전을 의미한다.

<br>
<br>

두 번째는 각 Ansible 버전 별 호환되는 ansible-core의 버전이다.

|Ansible 커뮤니티 릴리스|상태|핵심 버전 종속성|
|---|---|---|
|8.0.0|개발 중(미공개)|2.15
|7.x|현재|2.14|
|6.x|Ansible 6.7.0 이후 유지 관리되지 않음(수명 종료)|2.13|
|5.x|유지 관리되지 않음(수명 종료)|2.12|
|4.x|유지 관리되지 않음(수명 종료)|2.11|
|3.x|유지 관리되지 않음(수명 종료)|2.10|
|2.10|유지 관리되지 않음(수명 종료)|2.10|


CentOS 7에 기본 설치되는 파이썬 버전은 3.6.8인데, 이럴 경우 설치할 수 있는 최신 Ansible 버전은 4.10.0이 된다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2023-05-14-ansible_install/centos7-ansible.png?raw=true">

<br>
<br>

세 번째는 각 Ansible-core 별 릴리스 계획이다.

|버전|Release 일자|EOL|컨트롤러 파이썬|대상 Python/PowerShell|
|---|---|---|---|---|
|2.15|GA: 2023년 5월 22일<br>중요: 2023년 11월 6일<br>보안: 2024년 5월 20일|2024년 11월|파이썬 3.9 - 3.11|파이썬 2.7<br>파이썬 3.5 - 3.11<br>파워셸 3 - 5.1|
|2.14|GA: 2022년 11월 7일<br>긴급: 2023년 5월 22일<br>보안: 2023년 11월 6일|2024년 5월 20일|파이썬 3.9 - 3.11|파이썬 2.7<br>파이썬 3.5 - 3.11<br>파워셸 3 - 5.1|
|2.13|GA: 2022년 5월 23일<br>중요: 2022년 11월 7일<br>보안: 2023년 5월 22일|2023년 11월 06일|파이썬 3.8 - 3.10|파이썬 2.7<br>파이썬 3.5 - 3.10<br>파워셸 3 - 5.1|
|2.12|GA: 2021년 11월 8일<br>중요: 2022년 5월 23일<br>보안: 2022년 11월 7일|2023년 5월 22일|파이썬 3.8 - 3.10|파이썬 2.6<br>파이썬 3.5 - 3.10<br>파워셸 3 - 5.1|
|2.11|GA: 2021년 4월 26일<br>중요: 2021년 11월 8일<br>보안: 2022년 5월 23일|2022년 11월 7일|파이썬 2.7<br>파이썬 3.5 - 3.9|파이썬 2.6 - 2.7<br>파이썬 3.5 - 3.9<br>파워셸 3 - 5.1|
|2.10|GA: 2020년 8월 13일<br>중요: 2021년 4월 26일<br>보안: 2021년 11월 8일|2022년 5월 23일|파이썬 2.7<br>파이썬 3.5 - 3.9|파이썬 2.6 - 2.7<br>파이썬 3.5 - 3.9<br>파워셸 3 - 5.1|
|2.9|GA: 2019년 10월 31일<br>중요: 2020년 8월 13일<br>보안: 2021년 4월 26일|2022년 5월 23일|파이썬 2.7<br>파이썬 3.5 - 3.8|파이썬 2.6 - 2.7<br>파이썬 3.5 - 3.8<br>파워셸 3 - 5.1|

<br>
<br>

# Ansible 설치하기

현재 Python이 설치된 환경에서 어떤 Ansible 버전을 설치할 수 있는지 확인해봤다.

이번엔 실제로 Ansible을 설치하는 과정에 대해 알아볼 것이다.

CentOS 기준으로 yum으로 설치하는 방법도 있긴하나 별로 추천하진 않는다. 이럴 경우 CentOS에 기본 설치된 파이썬 버전에 종속되면서 Ansible 버전을 올릴 수 없는 문제가 발생하기 때문이다.

설치 환경
- CentOS 7.9. 2009
- Python 3.6.8

Python의 virtualenv 모듈을 통해 Ansible 만을 위한 가상의 개발환경을 만들게 된다.

```
##### virualenv 환경 생성
mkdir /home/ansible
cd /home/ansible
python3 -m virtualenv ansible-2.10.7
source ansible-2.10.7/bin/activate
pip install --upgrade pip

##### ansible 설치
pip install ansible==2.10.7
```

source 명령어 이후 부터 쉘 프롬프트에 (ansible-2.10.7)이 붙어있을 것이다.
이렇게 되어있다면 ansible-2.10.7이 설치된 가상의 개발환경에 사용자가 접근하게 된 것이다.

virualenv 로 가상 환경을 만드는 이유는 시스템에서 사용하는 파이썬과 구별하기 위함이다.

가상환경없이 pip로 특정 모듈을 설치한다면 이는 전체 시스템에 적용된다.
만약 가상 환경을 만든다면 python, pip 등의 실행 파일과 메타 데이터 및 설치한 모듈들의 저장 위치가 기존 위치와 전혀 다른 곳에 생성된다.

가상환경 내에서 pip로 모듈을 설치하던 설정값을 변경해도 전체 시스템에는 영향이 미치지 않는다.

이를 이용하면 시스템에 기본적으로 설치된 파이썬과의 간섭없이 최신 파이썬을 설치하고 이를 이용한 가상환경 생성도 가능해지는데 이는 다음에 정리하도록하겠다.



<br>
<br>


# 참고자료
[Ansible 버전 호환 비교](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#control-node-requirements)

[Ansible-core 버전 호환 비교](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#release-and-maintenance)