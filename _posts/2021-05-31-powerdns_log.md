---
title:  "[PowerDNS] 로그 설정"
excerpt: "PowerDNS 로그 설정"
header:
  overlay_color: "#333"
  actions:
    - label: "PowerDNS 공식사이트"
      url: "https://www.powerdns.com/"
categories:
  - other
tags:
  - PowerDNS
last_modified_at : 2021-05-31
last_modified_at_2 : 2021-05-31
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## DNS 로그

- log-dns-details : DNS 관련 로그를 자세하게 출력한다. 'no'로 설정 시 DNS 세부 정보가 syslog로 전송되지 않아 성능이 향상된다.
- log-dns-queries : 모든 DNS 쿼리에 대해 로그를 출력한다. 로그를 보기 위해 loglevel을 최소 5 이상 설정해야 한다.
- loglevel : syslog 수준 값에 해당. error = 3, warning = 4, notice = 5, info = 6. 절대 3미만의 값을 입력하면 안된다.

```
[root@localhost ~]# vi /etc/pdns/pdns.conf
~~~ (생략) ~~~
log-dns-details=yes
log-dns-queries=yes
loglevel=6
~~~ (생략) ~~~
[root@localhost ~]# syetemctl restart pdns.service
```

/var/log/message 로그 확인

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-31-powerdns_log/01_message_log.jpg?raw=true"></center>
<br>


## Syslog 사용

기본적으로 PowerDNS의 로그는 /var/log/messages 에 쌓이게 된다. 해당 파일에는 dns 뿐만 아니라 시스템 로그도 쌓이기 때문에 dns의 로그만 따로 쌓는 것이 관리 측면에서 더 편하다.

```
[root@localhost ~]# mkdir /var/log/pdns

[root@localhost ~]# vi /etc/pdns/pdns.conf
disable-syslog=no (기본값이므로 해당 옵션이 no로 되어 있는지 확인)
logging-facility=0

[root@localhost ~]# vi /etc/rsyslog.conf
# PDNS log
local0.=info                                             /var/log/pdns/pdns.info
local0.=warn                                             /var/log/pdns/pdns.warn
local0.=err                                              /var/log/pdns/pdns.err
```
<b>CentOS 6.x</b>
```
[root@localhost ~]# /etc/init.d/pdns restart
[root@localhost ~]# /etc/init.d/rsyslog restart
```
<b>CentOS 7.x</b>

service 파일에 "--disable-syslog" 추가

```	
[root@localhost ~]# vi /usr/lib/systemd/system/pdns.service 
~~~ (생략) ~~~
ExecStart=/usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --write-pid=no

[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl restart pdns.service
[root@localhost ~]# systemctl restart rsyslog.service

```

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-31-powerdns_log/02_pdns_log.jpg?raw=true"></center>
<br>


## systemd 환경에서 로그 설정

### 문제점
기존의 syslog를 이용한 로깅 설정을 했을 시 문제점은 로그가 rsyslog.conf에서 설정한대로 여러 로그 파일에 쌓이는 것과 동시에 message 파일에도 저장된다는 것이다.

rsyslog.conf에 messages 파일에 대해 local0.none 설정을 추가해도 마찬가지로 pdns 로그가 messages 파일에도 쌓이게 된다.

```
*.info;mail,local1.none;authpriv.none;cron.none;local0.none     /var/log/messages
```

### 원인

systemd 환경에서는 로그가 system-journald 에 의해 관리된다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-31-powerdns_log/03_systemd_log_01.jpg?raw=true"></center>
<br>

pdns.service 에서 발생하는 로그는 system-journald를 거쳐 journal 파일에 로깅이되며, rsyslogd로 전달된 로그는 /var/log 아래의 로그 파일에 로깅된다.

또한 systemd 유닛 파일에 syslog 관련 옵션이 없을 경우 daemon.info 형식으로 rsyslog에 전달된다.

따라서 pdns.conf에 지정한 local0.info / local0.warn / local0.err 와는 별도로 message 파일에도 로그가 쌓이게 된다.

### 해결 방법

systemd에 의해 생성된 로그가 rsyslog로 넘어갈 때 syslog facility와 Level 값을 붙여 로그가 쌓여야 하는 파일을 지정할 수 있다.

```
[root@localhost ~]# vi /usr/lib/systemd/system/pdns.service
~~~ (생략) ~~~
[Service]
ExecStart=/usr/sbin/pdns_server --guardian=no --daemon=no --log-timestamp=no --write-pid=no
~~~ (생략) ~~~
SyslogFacility=local0
SyslogLevel=debug
-> 위 두줄을 Service 구역에 추가
```

```
[root@localhost ~]# vi /etc/rsyslog.conf
~~~ (생략) ~~~
*.info;mail,local1.none;authpriv.none;cron.none;local0.none     /var/log/messages
~~~ (생략) ~~~
# PDNS log
local0.=info                                             /var/log/pdns/pdns.info
local0.=warn                                            /var/log/pdns/pdns.warn
local0.=err                                              /var/log/pdns/pdns.err
local0.=debug                                         /var/log/pdns/pdns.debug
-> messages 파일에 쌓이던 로그는 /var/log/pdns/pdns.debug에 쌓이게 된다.

[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl restart pdns.service
[root@localhost ~]# systemctl restart rsyslog.service
```

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-31-powerdns_log/04_systemd_log_02.jpg?raw=true"></center>
<br>


## 참고 자료
- systemd 유닛 설명

	<a href="https://www.freedesktop.org/software/systemd/man/systemd.exec.html">
	https://www.freedesktop.org/software/systemd/man/systemd.exec.html
	</a>

- systemd 유닛 옵션

	<a href="https://fmd1225.tistory.com/93">
	https://fmd1225.tistory.com/93
	</a>

- 리눅스 로그 관리

	<a href="https://m.blog.naver.com/PostView.nhn?blogId=nahejae533&logNo=221270596126&proxyReferer=https:%2F%2Fwww.google.com%2F">
	https://m.blog.naver.com/PostView.nhn?blogId=nahejae533&logNo=221270596126&proxyReferer=https:%2F%2Fwww.google.com%2F
	</a>

