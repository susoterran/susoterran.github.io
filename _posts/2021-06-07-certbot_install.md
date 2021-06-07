---
title:  "[Certbot] Certbot 설치"
excerpt: "Certbot 설치"
header:
  overlay_color: "#333"
  actions:
    - label: "Certbot 공식사이트"
      url: "https://certbot.eff.org/"
categories:
  - websrv
tags:
  - Linux
  - 서버보안
last_modified_at : 2021-06-07
last_modified_at_2 : 2021-06-07
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## HTTPS와 SSL 인증서

HTTPS와 SSL 인증서와 관련된 내용은 아래의 링크를 참고하면 도움이 된다.

<a href="https://opentutorials.org/course/228/4894">
https://opentutorials.org/course/228/4894
</a>

## Certbot 이란?

Certbot은 Let`s Encrypt 인증서를 자동으로 발급하고 설치해주는 오픈소스 소프트웨어 도구이다.

기존엔 git clone 이나 패키지 관리자를 통해 certbot을 설치했으나 최신 버전에서는 snap을 이용하여 certbot을 설치하는 것을 권장하고 있다.
	
여기에선 snap을 이용하여 certbot을 설치하는 방법에 대해 알아본다.

### 테스트 환경

- CentOS 7.9

### 기존 certbot 삭제

```
[root@localhost ~]# yum -y remove certbot
```

### snapd 설치

```
[root@localhost ~]# yum -y install epel-release
[root@localhost ~]# yum -y install snapd
[root@localhost ~]# systemctl enable --now snapd.socket
Created symlink from /etc/systemd/system/sockets.target.wants/snapd.socket to /usr/lib/systemd/system/snapd.socket.
[root@localhost ~]# ln -s /var/lib/snapd/snap /snap

```


```
[root@localhost ~]# snap install core
```

위 명령어 사용 시 아래와 같은 에러 메시지가 나온다면 될때까지 명령어 실행을 반복한다

error: too early for operation, device not yet seeded or device model not acknowledged

```
[root@localhost ~]# snap refresh core
```

위 명령어 사용 시 아래와 같은 메시지가 나와도 상관없으니 넘긴다.
snap "core" has no updates available


### certbot 설치

```
[root@localhost ~]# snap install --classic certbot
[root@localhost ~]# ln -s /snap/bin/certbot /usr/bin/certbot
```



## 참고 자료
- Certbot 설치 공식 문서

	<a href="https://certbot.eff.org/lets-encrypt/centosrhel7-apache#default">
	https://certbot.eff.org/lets-encrypt/centosrhel7-apache#default
	</a>