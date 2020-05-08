---
title:  "Let`s Encrypt 설치 및 SSL 인증서 발급"
excerpt: "리눅스에 Let`s Encrypt를 설치하고 SSL 인증서를 발급하여 사이트에 적용하는 방법"
header:
  overlay_color: "#333"
  actions:
--    - label: "OpenSSL공식사이트"
--      url: "https://www.openssl.org/"
categories:
  - websrv
tags:
  - OpenSSL
  - Linux
  - ssl
last_modified_at: 2020-03-30
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---
## Let`s Encrypt 란?

- 무료 SSL 인증서를 발급해주는 프로젝트

- 기존 상용 SSL 인증서들의 유효 기간이 1년 단위인데 반해 Let`s Encrypt를 통해 발급받은 인증서의 최대 유효 기간은 3개월이다. 따라서 3개월에 한번씩 인증서를 갱신해야 한다.

- 멀티 인증서 (여러 도메인을 하나의 인증서로 인증하는 형태)는 지원하지 않으나, 여러 개의 서브 도메인들을 묶어서 인증서를 발급받을 수 있다 (그러나 와일드카드 인증서는 지원안됨).


## 설치 환경
- CentOS 7.7
- OpenSSL 1.0.2s
- Apache 2.4.41

## Let`s Encrypt 클라이언트 설치

- Let\`s Encrypt 를 통해 인증서를 발급받기 위해서는 Let`s Encrypt 클라이언트가 필요하며, Python 2.4 이상의 패키지가 설치되어 있어야 한다.

```
[Let`s Encrypt 클라이언트 다운로드]

[root@localhost ~]# cd /home/work/
[root@localhost work]# yum -y install git
[root@localhost work]# git clone https://github.com/letsencrypt/letsencrypt
[root@localhost work]# cd letsencrypt/
```

## 인증서 발급

- 인증서 발급 시 DocumentRoot 안에 hash값을 삽입한 html 파일을 생성한 후, 인증 서버에서 생성한 html 파일을 URL로 실행하여 hash값이 출력되는 것을 확인하는 방식으로 사이트 인증을 한다.

- 인증서 발급 시 주의 할 사항

> mod_rewrite를 이용하여 http 접속 시 https로 강제 리다이렉트되게 설정했을 경우 인증서가 정상적으로 발급되지 않는다 (인증서버가 "http://인증서요청도메인주소" 로 접속하기 때문)

> 아파치와 같은 WEB 서버나 톰캣과 같은 WAS를 통해 웹서비스를 제공하나 80번 포트를 사용하지 않는다면 이 역시 인증서가 정상적으로 발급되지 않는다. 인증서가 발급되는 상황에는 80번 포트로 웹 접속이 가능해야 한다 (동작 중인 웹 서비스를 중지 시킬 것 없이 아무 내용도 없는 사이트를 하나 만들고 80번 포트로 접속할 수 있게만 한다면 인증서 발급이 가능)

- 아파치, Nginx 등 웹 서버에 맞는 플러그인을 통해 인증서 발급부터 서버 설정까지 자동으로 SSL 인증서를 설치할 수 있다.

- 인증서 발급 방식은 자동, 수동으로 나뉜다. 여기서는 두 가지 방식 모두를 다룬다.

### 자동 인증서 발급

- 여러 서브 도메인을 사용할 경우 아래와 같이 -d 옵션을 사용하여 서브 도메인들을 입력한다.

- webroot의 경우 www.h2terran.co.kr 도메인의 DocumentRoot를 입력한다.

```
[root@localhost ~]# /home/work/letsencrypt/certbot-auto certonly --webroot -w /home/httpd/html -d www.h2terran.co.kr -d h2terran.co.kr
```

<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/auto-issue.jpg?raw=true"></center>

### 수동 인증서 발급

- 플러그인을 통한 인증서 발급에 제한이 있을 경우에는 --manual 옵션을 통해 수동으로 인증서를 발급하고 서버에 설정한다.

```
[root@localhost letsencrypt]# ./letsencrypt-auto certonly --manual --register-unsafely-without-email
```

- letsencrypt 서비스 동의

<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/menual-issue-1.jpg?raw=true"></center>

- 인증서 발급을 원하는 도메인 주소를 입력 (www.h2terran.co.kr)

* 만약 복수 개의 도메인 주소를 입력하고자 한다면 쉼표(,)나 빈칸으로 구분한다 (www.h2terran.co.kr h2terran.co.kr)

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/menual-issue-2.jpg?raw=true">

- 인증서를 발급받을 서버의 IP 정보를 공개적으로 수집해도 되는지 묻는다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/menual-issue-3.jpg?raw=true">

- 발급받는 인증서의 사이트가 실제 존재하는지를 확인하는 단계. 아래에 나온 안내대로 http://www.h2terran.co.kr/.well-known/acme-challenge/duQnmJJq1VzQes_-dHmnyFjRjExw1pcu21OD-AtHu5A URL로 요청을 보냈을 때 중간에 나온 duQnmJJq1VzQes_-dHmnyFjRjExw1pcu21OD-AtHu5A 을 반환해야 인증단계가 마무리된다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/menual-issue-4.jpg?raw=true">

- 루트디렉터리에서 위의 URL에 맞게 파일을 생성해준다.

```
[root@www ~]# cd /home/httpd/html
[root@www html]# mkdir .well-known
[root@www html]# cd .well-known/
[root@www .well-known]# mkdir acme-challenge
[root@www .well-known]# cd acme-challenge/
[root@www acme-challenge]# vi duQnmJJq1VzQes_-dHmnyFjRjExw1pcu21OD-AtHu5A
duQnmJJq1VzQes_-dHmnyFjRjExw1pcu21OD-AtHu5A.RGx8oO5GT-D9auOp8wE8YWbLcDn_jeS6TbAuim3TEeo

위 내용을 입력 - 저장 후 종료
```
<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/menual-issue-5.jpg?raw=true">

- 인증이 완료 되었다. 인증서는 /etc/letsencrypt/live/www.h2terran.co.kr 디렉터리에서 확인할 수 있다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/menual-issue-6.jpg?raw=true">


```
[root@localhost letsencrypt]# cd /etc/letsencrypt/live/www.h2terran.co.kr
[root@localhost www.h2terran.co.kr]# ls -al
drwxr-xr-x 2 root root  93  2월  2 10:44 .
drwx------ 3 root root  46  2월  2 10:44 ..
-rw-r--r-- 1 root root 692  2월  2 10:44 README
lrwxrwxrwx 1 root root  42  2월  2 10:44 cert.pem -> ../../archive/www.h2terran.co.kr/cert1.pem
lrwxrwxrwx 1 root root  43  2월  2 10:44 chain.pem -> ../../archive/www.h2terran.co.kr/chain1.pem
lrwxrwxrwx 1 root root  47  2월  2 10:44 fullchain.pem -> ../../archive/www.h2terran.co.kr/fullchain1.pem
lrwxrwxrwx 1 root root  45  2월  2 10:44 privkey.pem -> ../../archive/www.h2terran.co.kr/privkey1.pem
```

- 동일 도메인 주소로 다시 인증서를 발급받게 되면 도메인-4자리수로 디렉터리가 새로 생성된다. 예를 들어 처음 인증서를 발급받게 되면 위와 같이 live 아래에 도메인 주소로 디렉터리가 생성되고, 2번째로 발급받을 시 www.h2terran.co.kr-0001 라는 이름의 디렉터리가 새로 생성된다.

## 인증서 설정 (아파치 기준)

```
[root@localhost www.h2terran.co.kr]# vi /usr/local/apache2/conf/httpd.conf
#LoadModule ssl_module modules/mod_ssl.so (주석 제거)
->
LoadModule ssl_module modules/mod_ssl.so

#LoadModule socache_shmcb_module modules/mod_socache_shmcb.so (주석 제거)
->
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so

# Secure (SSL/TLS) connections
#Include conf/extra/httpd-ssl.conf (주석 제거)
->
Include conf/extra/httpd-ssl.conf
```

```
[root@localhost www.h2terran.co.kr]# vi /usr/local/apache2/conf/extra/httpd-ssl.conf
~
<VirtualHost *:443>
DocumentRoot "/home/httpd/html"
ServerName www.h2terran.co.kr
ServerAlias h2terran.co.kr
ErrorLog "/var/log/apachelog/www.h2terra.co.kr_ssl_error_log"
TransferLog "/var/log/apachelog/www.h2terra.co.kr_ssl_access_log"
	
SSLCertificateFile "/etc/letsencrypt/live/www.h2terran.co.kr/cert.pem"
SSLCertificateKeyFile "/etc/letsencrypt/live/www.h2terran.co.kr/privkey.pem"
SSLCertificateChainFile "/etc/letsencrypt/live/www.h2terran.co.kr/chain.pem"
~

[root@localhost ~]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost ~]# /usr/local/apache2/bin/apachectl restart
```

- httpd-ssl.conf에 SSL 인증서 경로 입력 시 주의해야 할 점은 letsencrypt를 통해 발급받은 인증서는 /etc/letsencrypt/live/발급받은인증서의도메인명/ 디렉터리 아래에 생성된다 (정확히는 심볼릭 링크 파일). 갱신 시에도 이곳에 있는 인증서를 사용하게 되므로 임의의 경로에 인증서를 옮겨서 설정하지 않는 것이 좋다.


## 인증서 적용 후 확인

- 사이트에 적용한 인증서 정보 확인할 수 있는 사이트

    https://www.geocerts.com/ssl-checker

<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/letsencrypt/auto-issue-certification.jpg?raw=true"></center>


## 인증서 갱신

- Let`s encrypt를 통해 발급받은 인증서는 3개월에 한번씩 갱신이 필요하다. 인증서가 아예 만료된 상태에서는 갱신이 되지 않기 때문에 다시 발급 받아야 한다.
- letsencrypt로 발급받은 인증서의 유효기간은 90일이고, renew 명령어를 사용하면 자동으로 인증서의 기간을 확인해준다. 하지만 유효기간 만료 30일 이내일 때만 갱신이 가능하므로 특별한 사유가 없는 한 발급일 또는 갱신일로부터 60일이 지난 후부터 진행하길 권장한다.

```
[root@localhost letsencrypt]# ./letsencrypt-auto renew
```