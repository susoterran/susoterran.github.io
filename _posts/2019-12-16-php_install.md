---
title:  "PHP 설치하기"
excerpt: "이번 페이지에서는 리눅스에 PHP를 컴파일 설치하고 httpd 연동하는 방법에 대해 알아본다."
header:
  overlay_color: "#333"
  actions:
    - label: "php공식사이트"
      url: "https://www.php.net/"
categories:
  - websrv
tags:
  - Linux
last_modified_at: 2019-12-16 T
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
---

### PHP 란?

- Professional Hypertext Preprocessor
- 동적 웹 페이지 생성 스크립트 언어
- C, C++, Perl, Java 등의 언어를 기반으로 하며 C언어 문법과 비슷
- 오픈 소스이지만, GNU가 아닌 PHP 프로젝트 라이선스에 적용
- 초기에는 절차지향 형태의 언어였지만 PHP5에서부터 객체지향을 지원


### 다운로드 및 설치

#### 설치 환경
- CentOS 5,6,7
- httpd-2.4.39, pcre 8.42, MySQL

#### 필수 라이브러리 설치

#### libiconv 설치

- 문자 인코딩 라이브러리

```
[root@localhost work]# wget https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.15.tar.gz --no-check-certificate
[root@localhost work]# tar zxvf libiconv-1.15.tar.gz
[root@localhost work]# cd libiconv-1.15

#### OS가 32bit 인 경우
[root@localhost libiconv-1.15]# ./configure --prefix=/usr/local --libdir=/usr/local/lib

#### OS가 64bit인 경우
[root@localhost libiconv-1.15]# ./configure --prefix=/usr/local --libdir=/usr/local/lib64
	
[root@localhost libiconv-1.15]# make
[root@localhost libiconv-1.15]# make install
```

#### libmcrypt

- 다양한 암호화 알고리즘을 지원하는 mcrypt 라이브러리
	
```
[root@localhost work]# wget https://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz --no-check-certificate
[root@localhost work]# tar zxvf libmcrypt-2.5.8.tar.gz
[root@localhost work]# cd libmcrypt-2.5.8

#### OS가 32bit 인 경우
[root@localhost libmcrypt-2.5.8]# ./configure --prefix=/usr/local --libdir=/usr/local/lib

#### OS가 64bit 인 경우
[root@localhost libmcrypt-2.5.8]# ./configure --prefix=/usr/local --libdir=/usr/local/lib64

[root@localhost libmcrypt-2.5.8]# make
[root@localhost libmcrypt-2.5.8]# make install
[root@localhost libmcrypt-2.5.8]# /sbin/ldconfig
	
[root@localhost libmcrypt-2.5.8]# cd libltdl/

#### OS가 32bit 인 경우
[root@localhost libltdl]# ./configure --enable-ltdl-install --libdir=/usr/local/lib

#### OS가 64bit 인 경우
[root@localhost libltdl]# ./configure --enable-ltdl-install --libdir=/usr/local/lib64

[root@localhost libltdl]# make
[root@localhost libltdl]# make install
```

#### mhash
	
- hash 암호화 알고리즘

```
[root@localhost work]# wget https://sourceforge.net/projects/mhash/files/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz --no-check-certificate
[root@localhost work]# tar zxvf mhash-0.9.9.9.tar.gz
[root@localhost work]# cd mhash-0.9.9.9

#### OS가 32bit 인 경우
[root@localhost mhash-0.9.9.9]# ./configure --prefix=/usr/local --libdir=/usr/local/lib

#### OS가 64bit 인 경우
[root@localhost mhash-0.9.9.9]# ./configure --prefix=/usr/local --libdir=/usr/local/lib64
	
[root@localhost mhash-0.9.9.9]# make
[root@localhost mhash-0.9.9.9]# make install
```

#### mcrypt

- 다양한 암호화 알고리즘을 지원하는 mcrypt 라이브러리에 대한 인터페이스
- PHP 7.1.x 버전에서 해당 기능은 비활성화되며 (PHP 소스를 통해 설치되진 않으나 PECL 저장소를 통해 동적 모듈 형태로 설치 가능), 7.2.x 버전 이상에서는 아예 사용할 수 없다.
  - openssl 등 다른 라이브러리를 사용해야 함.
	
```
[root@localhost work]# wget https://sourceforge.net/projects/mcrypt/files/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz --no-check-certificate
[root@localhost work]# tar zxvf mcrypt-2.6.8.tar.gz
[root@localhost work]# cd mcrypt-2.6.8
	
[root@localhost mcrypt-2.6.8]# vi /etc/ld.so.conf
include ld.so.conf.d/*.conf
/usr/local/lib64                   <- 추가 후 저장
[root@localhost mcrypt-2.6.8]# /sbin/ldconfig

#### OS가 32bit 인 경우
[root@localhost mcrypt-2.6.8]# ./configure --prefix=/usr/local --libdir=/usr/local/lib

#### OS가 64bit 인 경우
[root@localhost mcrypt-2.6.8]# ./configure --prefix=/usr/local --libdir=/usr/local/lib64
	
[root@localhost mcrypt-2.6.8]# make
[root@localhost mcrypt-2.6.8]# make install
```

#### 기타 의존성 라이브러리 설치

- libxml2 : XML 문서의 구문을 분석하기 위한 소프트웨어 라이브러리
- libcurl : cURL에서 사용하는 라이브러리. 
  - cURL : 다양한 통신 프로토콜을 이용하여 데이터를 전송하기 위한 라이브러리와 명령 줄 도구를 제공하는 소프트웨어
- libjpeg : JPEG 이미지 데이터 포맷을 다루기 위한 기능을 가진 무료 라이브러리
- libpng : 공식 PNG 참조 라이브러리
- Freetype2 : font 라이브러리

```
[root@localhost work]# yum -y install libxml2 libxml2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel
```

#### PHP 다운로드 설치

[PHP 5.6.39]
```
[root@localhost work]# wget http://www.php.net/distributions/php-5.6.39.tar.gz
[root@localhost work]# tar zxvf php-5.6.39.tar.gz
[root@localhost work]# cd php-5.6.39

[root@localhost php-5.6.39]# LIBS="-liconv"
[root@localhost php-5.6.39]# export LIBS

[root@localhost php-5.6.39]# ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-mysql=/usr/local/mysql --with-gd --with-png-dir --with-zlib-dir --with-jpeg-dir --with-kerberos --with-freetype-dir --enable-mbstring --enable-sockets --disable-debug --enable-gd-native-ttf --enable-dba=shared --with-iconv-dir=/usr/local/lib64 --with-mhash --with-curl --enable-opcache --with-mcrypt --with-openssl=/usr/local/openssl --with-mysqli=/usr/local/mysql/bin/mysql_config

-> OS가 32bit인 경우에는 --with-iconv-dir 옵션의 경로를 제거해준다.

[root@localhost php-5.6.39]# make
[root@localhost php-5.6.39]# make install
```

#### PHP 기본 설정
```
[root@localhost php-5.6.39]# vi /usr/local/apache2/conf/httpd.conf
<IfModule dir_module>
    DirectoryIndex index.html index.php   -> index.php를 추가
</IfModule>
~~~
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz
-> 아래의 옵션 2개를 추가
AddType application/x-httpd-php .php .htm
AddType application/x-httpd-php-source .phps
```
```
[root@localhost php-5.6.39]# cp php.ini-production /usr/local/php/lib/php.ini
[root@localhost php-5.6.31]# vi /usr/local/php/lib/php.ini
; Production Value: Off
; http://php.net/short-open-tag
short_open_tag = On     (Off -> On 변경)
-> php 태그 단순화 (<?php ?> -> <? ?>)

; http://php.net/expose-php
expose_php = On          (On -> Off 변경)
-> 아파치 PHP 버전을 숨기기 위한 설정
```
- so 파일을 찾을 수 있도록 ld.so.conf의 맨 아래줄에 mysqllib의 디렉터리를 지정해준다.

```
[root@localhost php-5.6.39]# vi /etc/ld.so.conf
/usr/local/mysql/lib       (OS가 32bit일 경우)
/usr/local/mysql/lib64    (OS가 64bit일 경우)
	

[root@localhost php-5.6.39]# /sbin/ldconfig

[root@localhost php-5.6.39]# /usr/local/apache2/bin/httpd -t
Syntax OK
[root@localhost php-5.6.39]# /usr/local/apache2/bin/apachectl restart
```





