---
title:  "[PowerDNS] PDNS Manager 설치하기"
excerpt: "PDNS Manager 설치하기"
header:
  overlay_color: "#333"
  actions:
    - label: "PowerAdmin 공식사이트"
      url: "https://pdnsmanager.org/"
categories:
  - other
tags:
  - PowerDNS
last_modified_at : 2021-05-21
last_modified_at_2 : 2021-05-21
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## PDNS Manager

- PHP로 제작된 PowerDNS Web 프론트엔드 (PHP 7.1 이상 버전 설치가 필요)
- PowerDNS의 백엔드가 MySQL로 구성되어 있어야 사용이 가능하다.
- Apache 2.0 라이선스를 사용
- 공식사이트 : <a href="https://pdnsmanager.org/">PDNS Manager</a>

- PowerDNS에서 공개한 Web 프론트엔드의 목록은 아래의 링크에서 확인할 수 있다.
    <a href="https://github.com/PowerDNS/pdns/wiki/WebFrontends">https://github.com/PowerDNS/pdns/wiki/WebFrontends
    </a>

- PowerAdmin과 비교하여 PDNS Manager의 특징은 UI가 깔끔하단 점이다. 다만 PowerAdmin과 달리 PowerDNS의 마스터/슬레이브 기능을 설정할 수 없지만 앞서 설명했던 것처럼 백엔드를 MySQL을 사용하는 시점에선 사용할 일 없는 기능이다.

- 인터넷 익스플로러에서는 사이트가 정상적으로 동작하지 않는 점에 주의한다.


## 테스트 환경

- CentOS 7
- httpd 2.4
- MariaDB 10.4
- PHP 7.3
- PowerDNS 4.3

- APM 구성은 아래의 링크를 참고하여 설치하였다.

    <a href="https://susoterran.github.io/websrv/openssl_install/">[APM설치] OpenSSL 설치하기</a>

    <a href="https://susoterran.github.io/websrv/httpd_install/">[APM설치] httpd 설치하기</a>

    <a href="https://susoterran.github.io/websrv/php_7.2-7.4_install/">[APM설치] PHP 7.2 / 7.4 설치하기</a>

    <a href="https://susoterran.github.io/mysql/mariadb10.4_install/">[MariaDB] MariaDB 10.4 / 10.5 설치하기</a>

## PHP 모듈 설치

- PDNS Manager 설치를 위해 아래의 모듈을 추가로 설치한다.
  - apcu
  - pdo_mysql


### APCU 설치

apcu를 설치하지 않으면 PDNS Manager 설치 시 아래와 같은 에러 메시지가 나타난다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/01_apcu_fail.jpg?raw=true"></center>
<br>

apcu 모듈 설치는 다음과 같이 진행한다.

```
[root@localhost work]# wget http://pecl.php.net/get/apcu-5.1.16.tgz
[root@localhost work]# tar zxvf apcu-5.1.16.tgz 
[root@localhost work]# cd apcu-5.1.16/
[root@localhost apcu-5.1.16]# /usr/local/php/bin/phpize	
[root@localhost apcu-5.1.16]# ./configure --with-php-config=/usr/local/php/bin/php-config
[root@localhost apcu-5.1.16]# make
[root@localhost apcu-5.1.16]# make install

[root@localhost apcu-5.1.16]# vi /usr/local/php/lib/php.ini 
~
[apcu]
extension=/usr/local/php/lib/php/extensions/no-debug-zts-20180731/apcu.so
-> 내용 추가
	
[root@localhost apcu-5.1.16]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost apcu-5.1.16]# systemctl restart httpd.service 
	
[root@localhost apcu-5.1.16]# /usr/local/php/bin/php -m | grep apcu
apcu
```


### pdo_mysql 설치

pdo_mysql를 설치하지 않으면 PDNS Manager 설치 시 아래와 같은 에러 메시지가 나타난다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/02_pdomysql_fail.jpg?raw=true"></center>
<br>

pdo_mysql 모듈 설치는 다음과 같이 진행한다.

```
[root@localhost ~]# cd /home/work/php-7.3.27/ext/pdo_mysql
[root@localhost pdo_mysql]# /usr/local/php/bin/phpize
[root@localhost pdo_mysql]# ./configure --with-php-config=/usr/local/php/bin/php-config --with-pdo-mysql=/usr/local/mysql/bin/mysql_config
[root@localhost pdo_mysql]# make
[root@localhost pdo_mysql]# make install
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-zts-20180731/

[root@localhost pdo_mysql]# ls -al /usr/local/php/lib/php/extensions/no-debug-zts-20180731/pdo_mysql.so 
-rwxr-xr-x 1 root root 174000  4월 12 12:21 /usr/local/php/lib/php/extensions/no-debug-zts-20180731/pdo_mysql.so

[root@localhost pdo_mysql]# vi /usr/local/php/lib/php.ini 
~
[pdo_mysql]
extension=/usr/local/php/lib/php/extensions/no-debug-zts-20180731/pdo_mysql.so
위의 내용 추가 후 저장

[root@localhost pdo_mysql]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost pdo_mysql]# /usr/local/apache2/bin/apachectl restart

[root@localhost pdo_mysql]# /usr/local/php/bin/php -m | grep pdo_mysql
pdo_mysql
```

## PDNS Manager 설치

### 다운로드

```
[root@localhost ~]# cd /home/work/
[root@localhost work]# wget https://dl.pdnsmanager.org/pdnsmanager-2.0.1.tar.gz
[root@localhost work]# tar zxvf pdnsmanager-2.0.1.tar.gz 
	
[root@localhost work]# mv pdnsmanager-2.0.1 /home/httpd/pdnsmanager

[root@localhost work]# chown -R root:daemon /home/httpd/pdnsmanager
```

### 아파치 설정

```
[root@localhost work]# vi /usr/local/apache2/conf/httpd.conf
~
LoadModule rewrite_module modules/mod_rewrite.so  (주석제거)
~
# Virtual hosts
#Include conf/extra/httpd-vhosts.conf  (주석제거)
```

```
[root@localhost work]# vi /usr/local/apache2/conf/extra/httpd-vhosts.conf 
<VirtualHost *:80>
  ServerName domain.30test.com
	DocumentRoot /home/httpd/pdnsmanager/frontend
	
	RewriteEngine On
	RewriteRule ^index\.html$ - [L]
	RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME} !-f
	RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME} !-d
	RewriteRule !^/api/\.* /index.html [L]
	
	ErrorLog "|/usr/local/apache2/bin/rotatelogs /var/log/apachelog/domain.30test.com-error_log.%Y%m%d 1000M"
	CustomLog "|/usr/local/apache2/bin/rotatelogs /var/log/apachelog/domain.30test.com-access_log.%Y%m%d 1000M" common env=!IMAGE
	
	Alias /api /home/httpd/pdnsmanager/backend/public
	<Directory /home/httpd/pdnsmanager/backend/public>
	  RewriteEngine On
	  RewriteCond %{REQUEST_FILENAME} !-f
	  RewriteCond %{REQUEST_FILENAME} !-d
	  RewriteRule ^ index.php [QSA,L]
	</Directory>
	<Directory /home/httpd/pdnsmanager>
	  Require all granted
	</Directory>

</VirtualHost>
```

```	
[root@localhost work]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost work]# systemctl restart httpd.service 
```

### 설치

http://도메인주소/setup 으로 접속

Database 접속 정보와 사용할 admin 계정 입력 후 "Setup" 버튼을 클릭하면 설치가 완료된다.

Host에 'localhost'를 입력할 경우 아래와 같이 소켓 오류가 발생할 수 있다. '127.0.0.1'로 설정하면 정상적으로 로컬 DB와 연결이 된다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/03_socket_fail.jpg?raw=true"></center>
<br>

PDNS Manager 설치 시 PowerDNS에서 사용하는 데이터베이스 안의 내용이 비어있어야 정상 설치된다.

이는 /home/httpd/pdnsmanager/backend/sql/setup.sql 을 실행하여 pdns 데이터베이스 안의 테이블을 생성하기 때문이다.

setup.sql에는 PowerDNS에서 사용하는 테이블들도 같이 저장되어 있다. 혹시나 버전에 따라 테이블 구조가 달라질 수 있기 때문에 각 PowerDNS 버전에 맞는 구조인지 확인할 필요가 있다.

기존에 PowerDNS에서 사용하는 데이터베이스에 내용이 있을 경우 아래와 같이 오류가 발생한다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/04_database_fail.jpg?raw=true"></center>
<br>




## DNS Zone 설정 테스트

http://도메인주소 으로 접속하고 로그인 진행

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/05_login.jpg?raw=true"></center>
<br>

도메인 영역 생성

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/06_zone_create.jpg?raw=true"></center>
<br>

레코드 생성

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/07_record_create.jpg?raw=true"></center>
<br>

nslookup으로 도메인 정보 확인

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-21-pdnsmanager_install/08_nslookup.jpg?raw=true"></center>
<br>


## 참고 자료
- PDNS Manager 공식 사이트

    <a href="https://pdnsmanager.org/">ttps://pdnsmanager.org/
    </a>