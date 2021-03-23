---
title:  "[MySQL/MariaDB] MySQL/MariaDB에서 사용할 사설 SSL 인증서 생성하기"
excerpt: "MySQL/MariaDB 서버에서 사용할 사설 SSL 인증서 생성 (OpenSSL / GnuTLS)"
header:
  overlay_color: "#333"
  actions:
    - label: "OpenSSL 공식사이트"
      url: "https://www.openssl.org/"
categories:
  - mysql
tags:
  - OpenSSL
  - GnuTLS
  - 서버보안
last_modified_at : 2021-03-23
last_modified_at_2 : 2021-03-23
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## OpenSSL을 이용한 사설 인증서 생성

사설 인증서는 MySQL 서버나 클라이언트 어디에서 생성하든 상관없다.

다만 사설 인증서 생성할 때 필요한 OpenSSL이 설치되어 있어야 한다.

### 테스트 환경

- CentOS 7.9
- OpenSSL 1.1.1i

### 인증서 저장 위치 생성
```
[root@localhost work]# mkdir /etc/ssl/mysql
[root@localhost work]# cd /etc/ssl/mysql
```

### 사설 CA 인증서 생성

특정 옵션값을 제외한 나머지는 추가 입력없이 Enter

Common Name 에 'mysql-admin' 입력

```
[root@localhost mysql]# /usr/local/openssl/bin/openssl genrsa 2048 > ca-key.pem
[root@localhost mysql]# /usr/local/openssl/bin/openssl req -new -x509 -nodes -days 365000 -key ca-key.pem -out ca.pem
~
Common Name (e.g. server FQDN or YOUR name) []:mysql-admin
```

### 서버 인증서 생성

특정 옵션값을 제외한 나머지는 추가 입력없이 Enter

Common Name 에 'mysql-server' 입력

```
[root@localhost mysql]# /usr/local/openssl/bin/openssl req -newkey rsa:2048 -days 365000 -nodes -keyout server-key.pem -out server-req.pem
~
Common Name (e.g. server FQDN or YOUR name) []:mysql-server
	
[root@localhost mysql]# /usr/local/openssl/bin/openssl rsa -in server-key.pem -out server-key.pem
[root@localhost mysql]# /usr/local/openssl/bin/openssl x509 -req -in server-req.pem -days 365000 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
	
[root@localhost mysql]# /usr/local/openssl/bin/openssl verify -CAfile ca.pem server-cert.pem 
server-cert.pem: OK

[root@localhost mysql]# ls -al
drwxr-xr-x  2 root root  172  7월 18 11:35 .
drwxr-xr-x. 3 root root   32  7월 18 11:17 ..
-rw-r--r--  1 root root 1675  7월 18 11:26 ca-key.pem
-rw-r--r--  1 root root 1314  7월 18 11:27 ca.pem
-rw-r--r--  1 root root 1168  7월 18 11:27 server-cert.pem
-rw-------  1 root root 1679  7월 18 11:27 server-key.pem
-rw-r--r--  1 root root  993  7월 18 11:27 server-req.pem
```

### 클라이언트 인증서 생성

클라이언트 인증서는 다음과 같은 용도로 사용한다.

  - DB 서버와 별도로 존재하는 웹 서버에서 DB 서버로 SSL 통신을 할 때 웹 서버에 적용
	- 접속하고자 하는 DB 서버와 별도의 리눅스 환경에서 mysql 클라이언트 프로그램으로 DB 서버에 접속할 때 클라이언트에 적용
	- Replication 에서 Master와 Slave 간의 SSL 통신을 하고자 할 때 Slave 서버에 적용

특정 옵션값을 제외한 나머지는 추가 입력없이 Enter

Common Name 에 'mysql-client' 입력

```
[root@localhost mysql]# /usr/local/openssl/bin/openssl req -newkey rsa:2048 -days 365000 -nodes -keyout client-key.pem -out client-req.pem
~
Common Name (e.g. server FQDN or YOUR name) []:mysql-client
	
[root@localhost mysql]# /usr/local/openssl/bin/openssl rsa -in client-key.pem -out client-key.pem
[root@localhost mysql]# /usr/local/openssl/bin/openssl x509 -req -in client-req.pem -days 365000 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem

[root@localhost mysql]# /usr/local/openssl/bin/openssl verify -CAfile ca.pem client-cert.pem 
client-cert.pem: OK

[root@localhost mysql]# ls -al
drwxr-xr-x  2 root root  172  7월 18 11:35 .
drwxr-xr-x. 3 root root   32  7월 18 11:17 ..
-rw-r--r--  1 root root 1675  7월 18 11:26 ca-key.pem
-rw-r--r--  1 root root 1314  7월 18 11:27 ca.pem
-rw-r--r--  1 root root 1176  7월 18 11:35 client-cert.pem
-rw-------  1 root root 1679  7월 18 11:35 client-key.pem
-rw-r--r--  1 root root  997  7월 18 11:31 client-req.pem
-rw-r--r--  1 root root 1168  7월 18 11:27 server-cert.pem
-rw-------  1 root root 1679  7월 18 11:27 server-key.pem
-rw-r--r--  1 root root  993  7월 18 11:27 server-req.pem
```

## GnuTLS를 이용한 사설 인증서 생성

사설 인증서는 MySQL 서버나 클라이언트 어디에서 생성하든 상관없다.

다만 사설 인증서 생성할 때 필요한 OpenSSL이 설치되어 있어야 한다.

### 테스트 환경

- CentOS 7.9
- gnutls-utils-3.3.29-9.el7_6.x86_64

### GnuTLS 설치

```
[root@localhost work]# yum -y install gnutls-utils
	
[root@localhost mysql]# rpm -qa | grep gnutls-utils
gnutls-utils-3.3.29-9.el7_6.x86_64
```

### 인증서 저장 위치 생성
```
[root@localhost work]# mkdir /etc/ssl/mysql
[root@localhost work]# cd /etc/ssl/mysql
```

### 사설 CA 인증서 생성

특정 옵션값을 제외한 나머지는 추가 입력없이 Enter

Common Name 에 'mysql-admin' 입력

```
[root@localhost mysql]# certtool --generate-privkey --outfile ca-key.pem
[root@localhost mysql]# certtool --generate-self-signed --load-privkey ca-key.pem --outfile ca.pem
~
Common name: mysql-admin
~
Country name (2 chars): KR
~
The certificate will expire in (days): 365000
~
Is the above information ok? (y/N): y
```

### 서버 인증서 생성

특정 옵션값을 제외한 나머지는 추가 입력없이 Enter

Common Name 에 'mysql-server' 입력

```
[root@localhost mysql]# certtool --generate-privkey --outfile server-key.pem
[root@localhost mysql]# certtool --generate-certificate --load-privkey server-key.pem --outfile server-cert.pem --load-ca-certificate ca.pem --load-ca-privkey ca-key.pem
~
Common name: mysql-server
~
Country name (2 chars): KR
~
The certificate will expire in (days): 365000
~
Is the above information ok? (y/N): y
```

### 클라이언트 인증서 생성

특정 옵션값을 제외한 나머지는 추가 입력없이 Enter

Common Name 에 'mysql-client' 입력

```
[root@localhost mysql]# certtool --generate-privkey --outfile client-key.pem
	
[root@localhost mysql]# certtool --generate-certificate --load-privkey client-key.pem --outfile client-cert.pem --load-ca-certificate ca.pem --load-ca-privkey ca-key.pem
~
Common name: mysql-client
~
Country name (2 chars): KR
~
The certificate will expire in (days): 365000
~
Is the above information ok? (y/N): y

[root@localhost mysql]# ls -al
drwxr-xr-x  2 root  root   128  3월 22 22:25 .
drwxr-xr-x. 4 root  root    46  3월 22 22:25 ..
-rw-------  1 root root 5823  3월 22 01:10 ca-key.pem
-rw-r--r--  1 root root 1123  3월 22 01:11 ca.pem
-rw-r--r--  1 root root 1159  3월 22 01:14 client-cert.pem
-rw-------  1 root root 5816  3월 22 01:13 client-key.pem
-rw-r--r--  1 root root 1159  3월 22 01:14 server-cert.pem
-rw-------  1 root root 5813  3월 22 01:13 server-key.pem
```