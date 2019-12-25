---
title:  "httpd 설치하기"
excerpt: ""
header:
  overlay_color: "#333"
  actions:
    - label: "httpd공식사이트"
      url: "http://httpd.apache.org/"
categories:
  - websrv
tags:
  - httpd
  - Linux
last_modified_at: 2019-12-15 T
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
---
이번 페이지에서는 리눅스에 아파치 httpd를 컴파일 설치하는 방법에 대해 알아본다.

## httpd 란?

Apache HTTP Server (약칭 "http")는 오픈소스 소프트웨어 그룹인 아파치 소프트웨어 재단에서 만드는 웹 서버 프로그램이다.

모듈을 통해 많은 기능을 추가할 수 있으며, 다른 프로그램과의 연동도 가능하다. 이 때문에 여러가지 서버 사이드 프로그래밍 언어나 DBMS와도 궁합이 잘 맞는다.

PHP 모듈 (libphp.so)을 통해 PHP를 실행할 수도 있고, 속도 향상을 위해 PHP-FPM을 통해 PHP를 연결해서 사용한다.
- libphp.so 를 사용할 경우 httpd에서 PHP 언어 해석 및 응답 등을 진행
- PHP-FPM을 사용할 경우 httpd와는 별도의 php-fpm 프로세스가 동작하여 PHP 언어 해석 및 응답 등을 진행하며 httpd는 프록시의 역할을 수행한다. httpd와 php-fpm 사이의 통신은 포트번호나 소켓을 이용한다.

JSP의 경우에는 설치 및 설정이 조금 귀찮아지지만 톰캣과 연동하여 돌릴 수 있다.

라이선스는 GPL이 아닌 자체 라이선스를 쓴다. 아파치 라이선스 2.0을 따르는데, 아파치 소프트웨어 재단에서 만들었다는 사실을 밝히고 아파치 라이선스를 따르면 자유롭게 수정 및 재배포가 가능하다. 소스 공개 강제 사항도 없는 자유로운 라이선스다.

## 다운로드 및 설치

### 설치 환경

- CentOS 5,6,7
- Ubuntu 14.04, 16.04
- httpd-2.4.39, pcre 8.42, apr-1.6.5, apr-util 1.6.1, expat-devel-2.0.1-13.el6_8.x86_64

httpd 설치 시 필요한 라이브러리가 많다. 해당 라이브러리들에 대해서 살짝 알아보고 설치를 진행하겠다.

### APR (Apache Portable Runtime) 

OS 플랫폼 종류와 관계없이 아파치 웹서버 (정확히는 apr을 이용하는 프로그램들)를 구동시켜주는 라이브러리

OS 레벨에서 프로그램을 개발하는 것은 시간과 기술력이 많이 필요하다 (네트워크 소켓 관리, 파일 I/O 핸들링, 메모리 할당과 같은 OS 영역에서 동작하는 기능들도 같이 개발해야 한다면 많은 시간과 기술력이 필요해질 것이다. 게다가 리눅스와 윈도우, Mac OS를 다 지원하게 만든다면...)사용하는 하드웨어와 OS에 상관 없이 상위 레벨의 구현에만 신경쓸 수 있도록 만든 라이브러리가 APR이다. 

APR-util은 APR의 보조 라이브러리이다.

### PCRE

정규 표현식 C 라이브러리

한마디로 httpd에서 정규 표현식을 사용할 수 있게 해주느 라이브러리이다.

대표적으로 mod_rewrite 에서 정규표현식을 구현하기 위해 사용한다.

### expat

XML 파서 (parser) 라이브러리

XML 문서를 읽어들이고, XML문법에 맞게 작성되었는지 검사하고, 의미를 해석하는 기능을 가지고 있다.

apr / apr-util 1.6 이상을 설치할 시 반드시 설치해야 하는 라이브러리이다 (이전 버전까지는 expat이 apr 에 포함되었지만 1.6부터 포함되지 않는다)


### 다운로드

```
[root@localhost work]# wget http://archive.apache.org/dist/apr/apr-1.6.5.tar.gz
[root@localhost work]# wget http://archive.apache.org/dist/apr/apr-util-1.6.1.tar.gz
[root@localhost work]# wget https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
[root@localhost work]# wget http://archive.apache.org/dist/httpd/httpd-2.4.39.tar.gz
```

### 설치

#### [PCRE 설치]
```
[root@localhost work]# tar zxvf pcre-8.41.tar.gz
[root@localhost work]# cd pcre-8.41
[root@localhost pcre-8.41]# ./configure --prefix=/usr/local/pcre
[root@localhost pcre-8.41]# make
[root@localhost pcre-8.41]# make install
```

#### [expat 설치]

#### CentOS 일 경우
```
[root@localhost work]# yum -y install expat-devel
[root@localhost work]# rpm -qa | grep expat
expat-devel-2.1.0-10.el7_3.x86_64
```

#### Ubuntu 일 경우
```
root@ubuntu:/home/work# apt-get install libexpat1-dev
```


#### [httpd 설치]
```
[root@localhost work]# tar zxvf apr-1.6.3.tar.gz
[root@localhost work]# mv -f apr-1.6.3 ./httpd-2.4.39/srclib/apr
[root@localhost work]# tar zxvf apr-util-1.6.1.tar.gz
[root@localhost work]# mv -f apr-util-1.6.1 ./httpd-2.4.39/srclib/apr-util

[root@localhost work]# tar zxvf httpd-2.4.39.tar.gz
[root@localhost work]# cd httpd-2.4.39
[root@localhost httpd-2.4.39]# ./configure --prefix=/usr/local/apache2 --with-pcre=/usr/local/pcre --enable-mods-shared=most --enable-so --enable-modules=ssl --enable-modules=so --enable-modules=most --enable-ssl --with-ssl=/usr/local/openssl --with-mpm=event --with-included-apr --enable-lib64 --libdir=/usr/lib64
[root@localhost httpd-2.4.39]# make
[root@localhost httpd-2.4.39]# make install
```

#### 부팅 이후 자동 실행되게 설정
#### CentOS 6 이하 버전
```
[root@localhost ~]# ln -s /usr/local/apache2/bin/apachectl /etc/rc.d/rc3.d/S99apache
[root@localhost ~]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost ~]# /usr/local/apache2/bin/apachectl start
```
#### CentOS 7, Ubuntu 16.04
```
CentOS 7 -> [root@localhost ~]# vi /usr/lib/systemd/system/httpd.service
Ubuntu 16.04 -> root@ubuntu:~# vi /etc/systemd/system/httpd.service

[Unit]
Description=apache2 Service
After=syslog.target
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/apache2/bin/apachectl start
ExecStop=/usr/loacal/apache2/bin/apachectl stop
ExecReload=/usr/local/apache2/bin/apachectl graceful
PrivateTmp=true
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target

[root@localhost ~]# systemctl enable httpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.

[root@localhost ~]# systemctl list-unit-files | grep http
httpd.service                                 enabled

[root@localhost ~]# systemctl restart httpd.service
```
