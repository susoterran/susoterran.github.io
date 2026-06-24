---
title:  "[AWX] AWX 설치 이후 로그인 실패 조치하기"
excerpt: "설치 이후 발생하는 로그인 실패 현상 해결"
header:
  overlay_color: "#333"
categories:
  - CI/CD
tags:
  - Ansible
  - AWX
last_modified_at : 2026-06-24
last_modified_at_2 : 2026-06-24
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# 개요

* AWX 설치 직후 로그인이 계속 실패하는 현상이 발생한다. 분명히 정상적인 로그인 정보를 입력했음에도 계속 로그인이 실패한다.
* 환경
  * AWX 24.6.1
* 컨테이너 로그
  * 로그인 시 아래와 같은 로그가 출력된다.
```bash
2026-03-05 10:28:14,235 WARNING  [86d36004] django.security.csrf Forbidden (Origin checking failed - https://awx.test.com does not match any trusted origins.): /api/login/
[pid: 458|app: 0|req: 7/22] 192.168.0.13 () {64 vars in 1117 bytes} [Thu Mar  5 10:28:14 2026] POST /api/login/ => generated 2604 bytes in 3 msecs (HTTP/1.0 403) 7 headers in 268 bytes (1 switches on core 0)
[pid: 458|app: 0|req: 8/23] 192.168.0.13 () {58 vars in 963 bytes} [Thu Mar  5 10:28:31 2026] GET /api/login/ => generated 5886 bytes in 15 msecs (HTTP/1.0 200) 10 headers in 460 bytes (1 switches on core 0)
2026-03-05 10:28:31,356 WARNING  [6bdbf366] django.security.csrf Forbidden (Origin checking failed - https://awx.test.com does not match any trusted origins.): /api/login/
[pid: 458|app: 0|req: 9/24] 192.168.0.13 () {64 vars in 1117 bytes} [Thu Mar  5 10:28:31 2026] POST /api/login/ => generated 2604 bytes in 3 msecs (HTTP/1.0 403) 7 headers in 268 bytes (1 switches on core 0)
```

# 원인

* `http://192.168.0.13:8013/` 와 같이 AWX 서버의 IP주소 및 포트번호로 직접 접속을 할 때는 문제가 없다.
* 현재 AWX 앞단에 Nginx가 위치해 있으며 도메인(`https://awx.test.com`)을 통해 접속하도록 구성되어 있다.
* Django의 CSRF(크로스 사이트 요청 위조) 보안 정책에 의해 요청이 차단되었다.
  * 최신 AWX의 백엔드 프레임워크인 Django가 보안을 위해 미리 신뢰하도록 설정되지 않은 도메인(Origin)에서의 POST 요청(로그인 등)을 차단한다.


# 해결 방법
AWX 설정에 해당 도메인을 신뢰할 수 있는 출처(Trusted Origins)로 명시해 주면 된다.

```bash
$ docker exec -it tools_awx_1 bash
bash-5.1$ vi /etc/tower/conf.d/local_settings.py

# 아래 내용 추가
CSRF_TRUSTED_ORIGINS = ['https://awx.test.com']

# docker 컨테이너 재시작
$ docker restart tools_awx_1
```