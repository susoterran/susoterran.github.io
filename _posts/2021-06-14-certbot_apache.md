---
title:  "[Certbot] 싱글 도메인 인증서 발급/갱신"
excerpt: "Certbot을 이용한 싱글 도메인 인증서 발급/갱신"
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
last_modified_at : 2021-06-14
last_modified_at_2 : 2021-06-14
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## 개요

certbot을 이용하여 싱글 도메인 인증서를 발급하고 자동으로 Apache 웹 서버에 적용하는 방법을 설명한다.

여기서 사용하는 Apache는 아래의 링크에서 설명하는 방법으로 설치한 버전이다.

<a href="https://susoterran.github.io/websrv/httpd_install/">
[APM설치] httpd 설치하기
</a>


## 테스트 환경

- CentOS 7.9
- httpd 2.4.46 (event)
- OpenSSL 1.1.1k

## 싱글 도메인 인증서 발급

certbot을 이용하여 SSL 인증서를 발급받게 되는데 이때, apache 플러그인을 이용하여 apache에 적용된 (httpd-vhost.conf 파일 안의) 도메인 주소들을 확인한다.

httpd-vhost.conf 파일 안의 도메인 주소 중 SSL 인증서 발급을 원하는 도메인들을 일부분 또는 전체를 고를 수 있다.


certbot에서 apache 플러그인을 이용하여 SSL 인증서를 발급/적용/갱신 할 때 apachectl 스크립트를 사용한다. 

아파치를 컴파일 설치했을 경우 certbot이 apachectl 스크립트 위치를 찾지 못하므로 아래와 같이 두 가지 방법을 통해 인식할 수 있게 해야 한다.

<b>[심볼릭 링크 생성]</b>
```
[root@localhost ~]# ln -s /usr/local/apache2/bin/apachectl /sbin/apachectl
```
<br>
<b>[certbot 옵션 사용]</b>
```
certbot 실행 시 다음의 옵션을 추가 : --apache-ctl=/usr/local/apache2/bin/apachectl
```

여기서는 2번째 방법을 사용하여 인증서 발급/설치/갱신을 진행한다.

```
[root@localhost ~]# certbot certonly --apache --apache-server-root=/usr/local/apache2 --apache-challenge-location=/usr/local/apache2 --apache-bin=/usr/local/apache2/bin --apache-ctl=/usr/local/apache2/bin/apachectl

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache
	
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: domain.h2terran.xyz
2: pinpoint.h2terran.xyz
3: redi.h2terran.xyz
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 2 (인증서 발급할 도메인 선택. 전체 도메인 발급도 가능)
Requesting a certificate for pinpoint.h2terran.xyz
Performing the following challenges:
http-01 challenge for pinpoint.h2terran.xyz
Waiting for verification...
Cleaning up challenges
	
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
	  /etc/letsencrypt/live/pinpoint.h2terran.xyz/fullchain.pem
	  Your key file has been saved at:
	  /etc/letsencrypt/live/pinpoint.h2terran.xyz/privkey.pem
	  Your certificate will expire on 2021-07-07. To obtain a new or
	  tweaked version of this certificate in the future, simply run
	  certbot again. To non-interactively renew *all* of your
	  certificates, run "certbot renew"
	- If you like Certbot, please consider supporting our work by:
	
	  Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
	  Donating to EFF:                    https://eff.org/donate-le

```

인증서를 발급 받은 후 httpd-ssl.conf 파일은 수동으로 설정하고, apache 프로세스를 재실행하면 인증서 적용이 완료된다.


## SSL 인증서 상태 확인

아래의 명령어를 통해 certbot을 이용하여 발급받은 SSL 인증서의 정보와 현재 상태를 확인할 수 있다.

```
[root@localhost apachelog]# certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log
	
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: domain.h2terran.xyz
    Serial Number: 300f5b9ca1243fa7dea897bc7b899191902
    Key Type: RSA
    Domains: domain.h2terran.xyz
    Expiry Date: 2021-07-07 14:38:37+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/domain.h2terran.xyz/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/domain.h2terran.xyz/privkey.pem
  Certificate Name: pinpoint.h2terran.xyz
    Serial Number: 3a8bd82d5b21cf934648d92e1ea9d7e2dca
    Key Type: RSA
    Domains: pinpoint.h2terran.xyz
    Expiry Date: 2021-07-07 06:07:23+00:00 (VALID: 88 days)
    Certificate Path: /etc/letsencrypt/live/pinpoint.h2terran.xyz/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/pinpoint.h2terran.xyz/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

## SSL 인증서 갱신

### 수동 갱신

발급 받은 인증서의 유효기간은 발급 받은 이후로 90일까지이며, 기본적으로 인증서 만료일이 한달 이내로 남은 시점일 때부터 갱신이 가능하다. 

아래의 파일에서 갱신 가능일을 수정할 수 있다.

```	
[root@localhost ~]# vi /etc/letsencrypt/renewal/www10.h2terran.xyz.conf 
# renew_before_expiry = 30 days
-> 아래와 같이 변경
renew_before_expiry = 90 days
```
	
아래의 명령어로 인증서 수동 갱신 및 아파치 프로세스 재실행을 진행할 수 있다.
	
```
[root@localhost ~]# certbot renew --apache-ctl=/usr/local/apache2/bin/apachectl

Saving debug log to /var/log/letsencrypt/letsencrypt.log
	
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/domain.h2terran.xyz.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert is due for renewal, auto-renewing...
Plugins selected: Authenticator apache, Installer apache
Renewing an existing certificate for domain.h2terran.xyz
	
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of apache server; fullchain is
/etc/letsencrypt/live/domain.h2terran.xyz/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/pinpoint.h2terran.xyz.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert is due for renewal, auto-renewing...
Plugins selected: Authenticator apache, Installer apache
Renewing an existing certificate for pinpoint.h2terran.xyz

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of apache server; fullchain is
/etc/letsencrypt/live/pinpoint.h2terran.xyz/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all renewals succeeded: 
  /etc/letsencrypt/live/domain.h2terran.xyz/fullchain.pem (success)
  /etc/letsencrypt/live/pinpoint.h2terran.xyz/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

### 자동 갱신

수동 갱신 파트에 있는 renew_before_expiry 옵션을 변경하고 진행한다.

crond를 이용하여 매월 1일에 인증서를 재발급받고 아파치 적용을 위해 재실행을 진행하게 설정한다.

```
[root@localhost ~]# crontab -e
0 0 1 * * certbot renew --apache-ctl=/usr/local/apache2/bin/apachectl
```	




## 참고 자료
- Certbot 사용자 가이드

	<a href="https://certbot.eff.org/docs/using.html#apache">
	https://certbot.eff.org/docs/using.html#apache
	</a>