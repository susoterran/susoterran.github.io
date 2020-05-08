---
title:  "OpenSSL 설치하기"
excerpt: "리눅스에 OpenSSL을 컴파일 설치하는 방법"
header:
  overlay_color: "#333"
  actions:
    - label: "OpenSSL공식사이트"
      url: "https://www.openssl.org/"
categories:
  - websrv
tags:
  - OpenSSL
  - Linux
last_modified_at: 2020-05-06
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---
## OpenSSL 이란?

- 네트워크를 통한 데이터 통신에 쓰이는 프로토콜인 SSL/TLS의 오픈 소스 구현판이다.

- C언어로 작성되어 있는 라이브러리 안에는, 기본적인 암호화 기능 및 여러 유틸리티 함수들이 구현되어 있다다.

- 거의 모든 버전의 유닉스 계열 운영체제 (솔라리스, 맥 OS X, 리눅스, BSD 포함) 및 OpenVMS, 윈도우에서 이용 가능하다.

- 2014년 하트블리드 (Heartbleed) 사태로 인해 위기를 맞긴 했지만 패치 이후 지금까지도 현업에서도 많이 사용하고 있다.
  - 해당 취약점으로 인해 가급적이면 1.0.2 버전 이상의 OpenSSL을 설치할 것을 권장하고 있다.


## 설치 환경
- CentOS 5,6,7
- Ubuntu 14.04, 16.04
- OpenSSL 1.0.2s

yum으로 설치힌 openssl의 경우 yum 이나 rpm을 이용하여 업데이트를 진행할 수 있다. 하지만 최신 버전의 openssl은 rpm 패키지가 아닌 소스 형태로 배포되므로 yum을 통해서 최신 버전으로 업데이트를 진행할 수 없다.

openssl은 소스 파일 (정확히는 openssl 소스코드들의 압축 파일) 형태로 배포되며, gcc 등의 컴파일 도구를 이용하여 설치할 수 있다.


openssl을 소스 컴파일로 설치한 이후, 설치 한 openssl의 라이브러리를 이용하여 설치한 프로그램이 있다면 해당 프로그램은 openssl 라이브러러에 의존성을 가지게 된다. 한마디로 설치한 openssl이 없으면 정상적으로 구동되지 않는다는 의미이다.

openssl을 재컴파일하게 되면 의존성 문제가 발생하여 이후에 설치한 프로그램들이 정상적으로 동작하지 않게 된다. 이럴 경우 해당 프로그램들도 재컴파일 해주어야 한다.

	
## 다운로드

```
 [root@localhost work]# wget https://www.openssl.org/source/openssl-1.0.2s.tar.gz
```

## OpenSSL 설치
```
[root@localhost work]# tar zxvf openssl-1.0.2s.tar.gz
[root@localhost work]# cd openssl-1.0.2s
```

우분투에서는 처음부터 빌드에 필요한 패키지들이 설치되어 있지 않을 수 있다. 아래의 명령을 통해 필요한 패키지들을 설치한다.
```
[root@ubuntu:/home/gihcuser]# apt-get install build-essential
```

OS가 32bit인지 64bit 인지에 따라 옵션이 다르므로 주의한다.

32bit OS일 경우
```
[root@localhost openssl-1.0.2s]# ./config --prefix=/usr/local/openssl
```
64bit OS일 경우
```
[root@localhost openssl-1.0.2s]# ./config -fPIC --prefix=/usr/local/openssl
```	

```
[root@localhost openssl-1.0.2s]# make
[root@localhost openssl-1.0.2s]# make install
```

### 컴파일 옵션 설명

컴파일 옵션에 대한 자세한 사항은 아래 링크를 참고한다.
- https://github.com/openssl/openssl/blob/master/Configure

* &#45;&#45;prefix=/path/to/openssl
  * /path/to/openssl 아래에 bin, lib, include/openssl 디렉토리가 설치 될 경로 설정
  * OpenSSL에 의해 사용되는 설정 파일은 /path/to/openssl/ssl 또는 &#45;&#45;openssldir 옵션으로 설정한 경로 안에 위치할 것이다.
  * 해당 옵션을 입력하지 않으면 &#45;&#45;openssldir 옵션에 설정한 경로에 설치된다.

* -fPIC
  * [fPIC 옵션의 의미](https://susoterran.github.io//websrv/openssl-fpic/ "fPIC 옵션의 의미")
 
* &#45;&#45;openssldir=/path/to/openssl
  * OpenSSL 관련 파일들 (설정 파일, 서버 인증서 등)을 위치시킬 디렉토리 설정
  * /path/to/openssl 아래에 ssl 디렉터리가 추가된다.
		
* no-threads
  * 멀티 쓰레드 어플리케이션을 위한 라이브러리를 생성하지 않는다.
		
* threads
	* 멀티 쓰레드 어플리케이션을 위한 라이브러리를 생성한다 (기본값)
		
* no-zlib
	* TLS 통신을 압축하기 위한 zlib 기능 지원을 추가하지 않는다 (기본값)
		
* zlib
	* TLS 통신을 압축하기 위해 zlib 기능 지원을 추가한다
	* 참고자료 : https://securitygrind.com/building-openssl-with-zlib-support/
		
* zlib-dynamic
	* 필요할 때 동적으로 zlib 라이브러리를 로드한다.
	* shared 라이브러리가 로딩된 시스템에서만 사용한다.
	* 해당 옵션을 추가하면 자동으로 zlib 옵션이 활성화된다.
	* 기본값으로 비활성화 되어 있다.
		
* no-shared
	* shared 라이브러리를 생성하지 않는다 (기본값)
	* /path/to/openssl/lib 안에 정적 라이브러리 생성 (libcrypto.a, libssl.a)
		
* shared
	* /path/to/openssl/lib 경로에 공유 라이브러리 (libcrypto.so, libssl.so) 생성. 
  * openssl-devel 패키지를 설치하는 것 동일

## 설치 버전 확인
```
[root@localhost ~]# /usr/local/openssl/bin/openssl 
OpenSSL> version
OpenSSL 1.0.2s  20 Nov 2018
```