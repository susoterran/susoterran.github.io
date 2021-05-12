---
title:  "[PowerDNS] PowerAdmin 설치하기"
excerpt: "PowerAdmin 설치하기"
header:
  overlay_color: "#333"
  actions:
    - label: "PowerAdmin 공식사이트"
      url: "http://www.poweradmin.org/"
categories:
  - other
tags:
  - PowerDNS
last_modified_at : 2021-05-12
last_modified_at_2 : 2021-05-12
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## PowerAdmin

- PHP로 제작된 PowerDNS Web 프론트엔드
- PowerDNS의 백엔드가 MySQL로 구성되어 있어야 사용이 가능하다.
- GPL 라이선스를 사용
- 공식사이트 : <a href="http://www.poweradmin.org/">PowerAdmin</a>

- PowerDNS에서 공개한 Web 프론트엔드의 목록은 아래의 링크에서 확인할 수 있다.
    <a href="https://github.com/PowerDNS/pdns/wiki/WebFrontends">https://github.com/PowerDNS/pdns/wiki/WebFrontends
    </a>

- PowerAdmin의 특징은 PowerDNS의 마스터/슬레이브 기능을 설정할 수 있다는 점이다. 하지만 PowerDNS의 마스터/슬레이브 기능은 완전히 구현되어 있지 않아 공식적으로 사용하지 말라고 하며, DBMS를 백엔드로 쓸 것을 권장하고 있다. 해당 기능에 대해서는 뒤에서 설명하도록 하겠다.


## 테스트 환경

- CentOS 7
- httpd 2.4
- MySQL 5.6
- PHP 5.6
- PowerDNS 4.1

- APM 구성은 아래의 링크를 참고하여 설치하였다.

    <a href="https://susoterran.github.io/websrv/openssl_install/">[APM설치] OpenSSL 설치하기</a>

    <a href="https://susoterran.github.io/websrv/httpd_install/">[APM설치] httpd 설치하기</a>

    <a href="https://susoterran.github.io/websrv/php_install/">[APM설치] PHP 5.6 설치하기</a>

    <a href="https://susoterran.github.io/mysql/mysql5.6_install/">[MySQL] MySQL 5.6 / 5.7 설치하기</a>

## PHP 모듈 설치

- PowerAdmin 설치를 위해 아래의 모듈을 추가로 설치한다.
  - gettext
  - mcrypt
  - MDB2 (PowerAdmin 2.1.6 설치 시 필요)
  - pdo_mysql (PowerAdmin 2.1.7 설치 시 필요)

### gettext 설치

gettext를 설치하지 않으면 아래와 같은 에러 메시지가 나타난다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/01_gettext_fail.jpg?raw=true"></center>
<br>

gettext 모듈 설치는 다음과 같이 진행한다.

```
[root@localhost work]# cd /home/work/php-5.6.31/
[root@localhost php-5.6.31]# cd ext/gettext
[root@localhost gettext]# /usr/local/php/bin/phpize
	
[root@localhost work]# cd /home/work/php-5.6.31/
[root@localhost php-5.6.31]# ./configure --help | grep gettext
--with-gettext=DIR      Include GNU gettext support
	
[root@localhost gettext]# ./configure --with-php-config=/usr/local/php/bin/php-config
[root@localhost gettext]# make
[root@localhost gettext]# make install
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-zts-20131226/

[root@localhost gettext]# vi /usr/local/php/lib/php.ini
~
[gettext]
extension=/usr/local/php/lib/php/extensions/no-debug-zts-20131226/gettext.so
~

[root@localhost gettext]# /usr/local/apache2/bin/apachectl restart

[root@localhost gettext]# /usr/local/php/bin/php -m | grep gettext
gettext
```

### MDB2 설치

MDB2를 설치하지 않으면 PowerAdmin 설치 시 아래와 같은 에러 메시지가 나타난다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/02_mdb2_fail.jpg?raw=true"></center>
<br>

MDB2 모듈 설치는 다음과 같이 진행한다.

```
[root@localhost inc]# cd /usr/local/php/bin/
[root@localhost inc]# ./pear install MDB2
[root@localhost inc]# ./pear install MDB2_Driver_mysql
[root@localhost inc]# /sbin/ldconfig
[root@localhost inc]# /usr/local/apache2/bin/httpd -t
[root@localhost inc]# /usr/local/apache2/bin/apachectl restart
```

MDB2 모듈 설치 중 아래와 같은 에러가 발생할 수 있다.

[root@localhost bin]#  ./pear install MDB2
No releases available for package "pear.php.net/MDB2"
install failed

다음과 같이 조치 후 위의 설치 방법을 다시 진행한다.

```
[root@localhost bin]# ./php -r "print_r(openssl_get_cert_locations());"
Array
(
    [default_cert_file] => /usr/local/openssl/ssl/cert.pem
    [default_cert_file_env] => SSL_CERT_FILE
    [default_cert_dir] => /usr/local/openssl/ssl/certs
    [default_cert_dir_env] => SSL_CERT_DIR
    [default_private_dir] => /usr/local/openssl/ssl/private
    [default_default_cert_area] => /usr/local/openssl/ssl
    [ini_cafile] => 
    [ini_capath] => 
)

[root@localhost bin]# cd /usr/local/openssl/ssl/
[root@localhost ssl]# ls
certs  man  misc  openssl.cnf  private
-> 실제 경로에 cert.pem 파일이 없다

[root@localhost bin]# cp /etc/pki/tls/cert.pem /usr/local/openssl/ssl/
```

### pdo_mysql 설치

pdo_mysql를 설치하지 않으면 PowerAdmin 설치 시 아래와 같은 에러 메시지가 나타난다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/03_pdomysql_fail.jpg?raw=true"></center>
<br>

pdo_mysql 모듈 설치는 다음과 같이 진행한다.

```
[root@localhost ~]# cd /home/work/php-5.6.31/ext/pdo_mysql
[root@localhost pdo_mysql]# /usr/local/php/bin/phpize
[root@localhost pdo_mysql]# ./configure --with-php-config=/usr/local/php/bin/php-config --with-pdo-mysql=/usr/local/mysql/bin/mysql_config
[root@localhost pdo_mysql]# make
[root@localhost pdo_mysql]# make install
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-zts-20131226/

[root@localhost pdo_mysql]# ls -al /usr/local/php/lib/php/extensions/no-debug-zts-20131226/pdo_mysql.so 
-rwxr-xr-x 1 root root 174000  3월 23 19:24 /usr/local/php/lib/php/extensions/no-debug-zts-20131226/pdo_mysql.so

[root@localhost pdo_mysql]# vi /usr/local/php/lib/php.ini 
~
[pdo_mysql]
extension=/usr/local/php/lib/php/extensions/no-debug-zts-20131226/pdo_mysql.so
위의 내용 추가 후 저장

[root@localhost pdo_mysql]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost pdo_mysql]# /usr/local/apache2/bin/apachectl restart

[root@localhost pdo_mysql]# /usr/local/php/bin/php -m | grep pdo_mysql
pdo_mysql
```

## PowerAdmin 설치

### 아파치 설정

```
[root@localhost etc]# vi /usr/local/apache2/conf/extra/httpd-vhosts.conf
~
<VirtualHost *:80>
    DocumentRoot "/home/httpd/html/poweradmin"
    ServerName domain.30test.com
    ErrorLog "|/usr/local/apache2/bin/rotatelogs /var/log/apachelog/domain.30test.com-error_log.%Y%m%d 1000M"
	  CustomLog "|/usr/local/apache2/bin/rotatelogs /var/log/apachelog/domain.30test.com-access_log.%Y%m%d 1000M" common env=!IMAGE
	  <Directory "/home/httpd/html/poweradmin">
	      Options FollowSymLinks
	      AllowOverride None
	      Require all granted
	  </Directory>
</VirtualHost>
~
	
[root@localhost etc]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost etc]# /usr/local/apache2/bin/apachectl restart
```

### 다운로드

[root@localhost etc]# cd /home/work
	
<b>poweradmin 2.1.6</b>

```
[root@localhost work]# wget https://downloads.sourceforge.net/project/poweradmin/poweradmin-2.1.6.tgz --no-check-certificate
	
[root@localhost work]# tar zxvf poweradmin-2.1.6.tgz
[root@localhost work]# mv poweradmin-2.1.6 /home/httpd/html/poweradmin
```


<b>poweradmin 2.1.7</b>

```
[root@localhost work]# wget https://jaist.dl.sourceforge.net/project/poweradmin/poweradmin-2.1.7.tgz
	
[root@localhost work]# tar zxvf poweradmin-2.1.7.tgz
[root@localhost work]# mv poweradmin-2.1.7 /home/httpd/html/poweradmin
```

### 설치 전 필요사항

- 웹 소스 파일을 옮기고 접속해 보면 아래와 같은 페이지가 나타난다. 
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/18_config_error1.jpg?raw=true"></center>
<br>

- config.inc.php 파일을 만들고 설정해준다.
```
[root@localhost gettext]# cd /home/httpd/html/poweradmin/inc
[root@localhost inc]# ls -al
~
-rw-r--r--. 1 501 games  1589  5월  3  2012 config-me.inc.php
~
[root@localhost inc]# cp config-me.inc.php config.inc.php
```
<br>

- 새로고침(F5)하면 아래와 같은 페이지가 나타난다. install 링크를 눌러 설치 작업을 진행한다.
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/19_config_error2.jpg?raw=true"></center>
<br>


### PowerAdmin 설치

- 사용할 언어를 선택 (여기서는 영어를 선택)
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/04_poweradmin_install01.jpg?raw=true"></center>
<br>

- "Go to step 3' 클릭
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/05_poweradmin_install02.jpg?raw=true"></center>
<br>

- DB 커넥션 정보를 입력한다.
	- Username : PowerDNS에서 사용하는 데이터베이스 계정
	- Password : Username에 입력한 계정의 패스워드
	- Database type : 서버에 설치된 DBMS
	- Hostname : DBMS 접속 주소
	- DB Port : DBMS 접속 포트
	- Databases : PowerDNS에서 사용하는 데이터베이스 이름
	- Poweradmin administrator password : Poweradmin의 관리자 계정 (admin)의 패스워드
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/06_poweradmin_install03.jpg?raw=true"></center>
<br>

- poweradmin에서 사용할 관리자 계정 정보와 영역 생성 시 기본적으로 적용할 hostmaster와 1차/2차 네임서버를 입력한다.
	- Username : Poweradmin에서 DB에 접속하기 위해 사용하는 계정
	- Password : Username에서 입력한 계정의 패스워드
	- Hostmaster : 영역 생성 시 SOA 레코드에 기본적으로 적용할 영역 관리자의 메일 주소. '@' 대신에 '.' 을 사용한다 (입력하지 않아도 상관없다)
	- Primary nameserver : 영역 생성 시 SOA 레코드에 기본적으로 적용할 1차 네임서버의 도메인 주소 (입력하지 않아도 상관없다)
  - Secondary nameserver : 영역 생성 시 SOA 레코드에 기본적으로 적용할 2차 네임서버의 도메인 주소 (입력하지 않아도 상관없다)
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/07_poweradmin_install04.jpg?raw=true"></center>
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/08_poweradmin_install05.jpg?raw=true"></center>
<br>

- 6단계에서 아래와 같이 ../inc/config.inc.php 에 내용을 쓸 수 없다는 메시지가 나타난다. 해당 파일에 아래 내용을 수동으로 입력해준다.
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/09_poweradmin_install06.jpg?raw=true"></center>
<br>

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/10_poweradmin_install07.jpg?raw=true"></center>
<br>

- 설치가 완료되면 install 디렉터리를 삭제한다.
```
[root@localhost inc]# rm -rf /home/httpd/html/poweradmin/install
```
<br>

- 로그인 창
<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/11_poweradmin_login.jpg?raw=true"></center>
<br>

- 만일 로그인후 "Error: Default session encryption key is used, please set it in your configuration file." 메시지가 웹페이지 상단에 출력될 경우 아래의 과정은 실행하면 된다.
 
```
openssl 명령어를 이용하여 랜덤문자를 생성
$ openssl rand -base64 32
BGHfCJUh5VcB4qaRpjaXMBnXMFxyCesybeXx2h+4PCA=
	
config.inc.php파일에 아래의 내용으로 변경
$session_key = 'BGHfCJUh5VcB4qaRpjaXMBnXMFxyCesybeXx2h+4PCA=';
```

## DNS Zone 설정 테스트

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/12_test1.jpg?raw=true"></center>
<br>

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/13_test2.jpg?raw=true"></center>
<br>

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/14_test3.jpg?raw=true"></center>
<br>

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/15_test4.jpg?raw=true"></center>
<br>

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/16_test5.jpg?raw=true"></center>
<br>

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-12-poweradmin_install/17_test6.jpg?raw=true"></center>
<br>

- nslookup으로 도메인 정보 확인

```
[root@localhost inc]# nslookup
> server 127.0.0.1
Default server: 127.0.0.1
Address: 127.0.0.1#53

> ftp.30test.com
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   ftp.30test.com
Address: IP주소
```



## 참고 자료
- PowerAdmin 공식 사이트

    <a href="http://www.poweradmin.org/">http://www.poweradmin.org/
    </a>

- PDNS 설치

    <a href="https://idchowto.com/?p=33188">https://idchowto.com/?p=33188
    </a>