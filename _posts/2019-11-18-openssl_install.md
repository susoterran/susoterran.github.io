---
title:  "OpenSSL 설치하기"
header:
  image: https://file.bodnara.co.kr/logo/insidelogo.php?image=%2Fhttp%3A%2F%2Ffile.bodnara.co.kr%2Fwebedit%2Fnews%2F2015%2F1558324780-halo_mcc_343.jpg
categories:
  - websrv
tags:
  - OpenSSL
  - Linux
last_modified_at: 2019-11-18 T

toc: true
toc_label: "On this page"
toc_sticky: true
classes: wide
---
이번 페이지에서는 리눅스에 OpenSSL을 컴파일 설치하는 방법에 대해 알아본다.

## OpenSSL 이란?

- 네트워크를 통한 데이터 통신에 쓰이는 프로토콜인 SSL/TLS의 오픈 소스 구현판이다.

- C언어로 작성되어 있는 라이브러리 안에는, 기본적인 암호화 기능 및 여러 유틸리티 함수들이 구현되어 있다다.

- 거의 모든 버전의 유닉스 계열 운영체제 (솔라리스, 맥 OS X, 리눅스, BSD 포함) 및 OpenVMS, 윈도우에서 이용 가능하다.

- 2014년 하트블리드 (Heartbleed) 사태로 인해 위기를 맞긴 했지만 패치 이후 지금까지도 현업에서도 많이 사용하고 있다.
  - 해당 취약점으로 인해 가급적이면 1.0.2 버전 이상의 OpenSSL을 설치할 것을 권장하고 있다.


## 다운로드 및 설치

### 설치 환경
- CentOS 5,6,7
- Ubuntu 14.04, 16.04
- OpenSSL 1.0.2s

yum으로 설치힌 openssl의 경우 yum 이나 rpm을 이용하여 업데이트를 진행할 수 있다. 하지만 최신 버전의 openssl은 rpm 패키지가 아닌 소스 형태로 지원되므로 yum을 이용하여 최신 버전으로 업데이트를 진행할 수 없다.

openssl은 소스 파일 (정확히는 openssl 소스코드들의 압축 파일) 형태로 배포되며, gcc 등의 컴파일 도구를 이용하여 설치할 수 있다.


openssl을 소스 컴파일로 설치한 이후, 설치 한 openssl의 라이브러리를 이용하여 설치한 프로그램이 있다면 해당 프로그램은 openssl 라이브러러에 의존성을 가지게 된다. 한마디로 설치한 openssl이 없으며 정상적으로 구동되지 않는다는 의미이다. 

openssl을 재컴파일하게 되면 의존성 문제가 발생하여 이후에 설치한 프로그램들이 정상적으로 동작하지 않게 된다. 이럴 경우 해당 프로그램들도 재컴파일 해주어야 한다.

	
### 다운로드

```
 [root@localhost work]# wget https://www.openssl.org/source/openssl-1.0.2s.tar.gz
```

### 설치
```
[root@localhost work]# tar zxvf openssl-1.0.2s.tar.gz
[root@localhost work]# cd openssl-1.0.2s
```

우분투에서는 처음부터 빌드에 필요한 패키지들이 설치되어 있지 않을 수 있다. 아래의 명령을 통해 필요한 패키지들을 설치한다.
```
[root@ubuntu:/home/gihcuser]# apt-get install build-essential
```

OS가 32bit인지 64bit 인지에 따라 옵션이 다르므로 주의한다.
컴파일 옵션에 대한 자세한 사항은 아래 링크를 참고한다.
- https://github.com/openssl/openssl/blob/master/Configure

#### [OS 32bit]
```
[root@localhost openssl-1.0.2s]# ./config --prefix=/usr/local/openssl
```
#### [OS 64bit]
```
[root@localhost openssl-1.0.2s]# ./config -fPIC --prefix=/usr/local/openssl
```	

```
[root@localhost openssl-1.0.2s]# make
[root@localhost openssl-1.0.2s]# make install
```

### 설치 버전 확인
```
[root@localhost ~]# /usr/local/openssl/bin/openssl 
OpenSSL> version
OpenSSL 1.0.2s  20 Nov 2018
```