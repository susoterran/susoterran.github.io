---
title:  "[Certbot] 와일드카드 도메인 인증서 발급/갱신"
excerpt: "Certbot을 이용한 와일드카드 도메인 인증서 발급/갱신"
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
last_modified_at : 2021-06-27
last_modified_at_2 : 2021-06-27
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## 개요

certbot을 이용하여 와일드카드 도메인 인증서를 발급받는 방법을 설명한다.

certbot에서 와일드카드 도메인 인증서를 발급받는 과정은 아래와 같다.
  - certbot에서 TXT 레코드에 추가할 값을 발급
  - 발급받은 값을 도메인의 TXT 레코드에 추가
  - 발급 서버에서 TXT 레코드를 확인함으로써 도메인에 대한 소유권 유무를 확인
  - 인증서 발급

위 과정 중 두 번째는 자동이나 수동으로 진행할 수 있다.

자동으로 발급받는 경우, 사용하는 DNS 서비스에서 API를 지원해주는지를 확인해야 한다. 아래의 링크에서 API 지원 여부를 확인한다.

<a href="https://certbot.eff.org/docs/using.html#dns-plugins">
https://certbot.eff.org/docs/using.html#dns-plugins
</a>

여기서는 PowerDNS로 네임서버를 구축한 환경에서 certbot을 통한 와일드카드 도메인 인증서를 자동으로 발급받아 볼 것이다.

수동으로 발급받는 경우에는 certbot에서 등록하라고 준 값을 네임 서버에 TXT 레코드로 추가한 후 certbot 명령어를 계속 진행해서 발급 받기 때문에 여기서 따로 설명하진 않겠다.

사용하는 Apache는 아래의 링크에서 설명하는 방법으로 설치한 버전이다.

<a href="https://susoterran.github.io/websrv/httpd_install/">
[APM설치] httpd 설치하기
</a>


## 테스트 환경

- CentOS 7.9
- httpd 2.4.46 (event)
- OpenSSL 1.1.1k
- PowerDNS 4.3

## 플러그인 설치 및 설정

인증서 자동 발급을 위해 certbot에서 네임 서버에 TXT 레코드를 등록할 수 있도록 해야 한다.

이 때 필요한 것은 아래의 세 가지이다.

  - 네임 서버에서 Dynamic DNS Update 지원할 수 있도록 설정되어 있어야 한다.
  - 네임 서버에서 TSIG 키를 생성한다.
  - certbot 플러그인 (certbot-dns-rfc2136 : Dynamic DNS Update를 이용하여 TXT 레코드를 등록시켜주는 플러그인)

<br>
첫 번째와 두 번째의 경우 이전에 작성한 포스트를 참고하면 된다.

<a href="https://susoterran.github.io/other/powerdns_dynamic_update/">
[PowerDNS] DNS Dynamic Update
</a>

<br>
이제 플러그인 설치와 기본 설정을 진행하도록 한다.

```
[root@localhost ~]# snap set certbot trust-plugin-with-root=ok
	
[root@localhost ~]# snap install certbot-dns-rfc2136
certbot-dns-rfc2136 1.15.0 from Certbot Project (certbot-eff✓) installed

[root@localhost ~]# vi /etc/letsencrypt/rfc2136.ini
# Target DNS server
dns_rfc2136_server = 127.0.0.1
# Target DNS port
dns_rfc2136_port = 53
# TSIG key name
dns_rfc2136_name = test1234
# TSIG key secret
dns_rfc2136_secret = Dynamic DNS Update에 사용할 시크릿 값
# TSIG key algorithm
dns_rfc2136_algorithm = HMAC-SHA512
	
[root@localhost ~]# chmod 600 /etc/letsencrypt/rfc2136.ini 
```

## PowerDNS에 Dynamic DNS Update 설정



## 와일드카드 인증서 발급

```
[root@localhost letsencrypt]# certbot certonly --dns-rfc2136 --dns-rfc2136-credentials /etc/letsencrypt/rfc2136.ini -d *.h2terran.site -d h2terran.site
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator dns-rfc2136, Installer None
Requesting a certificate for *.h2terran.site and h2terran.site
	
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
  /etc/letsencrypt/live/h2terran.site/fullchain.pem
  Your key file has been saved at:
  /etc/letsencrypt/live/h2terran.site/privkey.pem
  Your certificate will expire on 2021-08-11. To obtain a new or
  tweaked version of this certificate in the future, simply run
  certbot again. To non-interactively renew *all* of your
  certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:
	
  Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
  Donating to EFF:                    https://eff.org/donate-le
```

해당 작업을 진행하면 TXT 레코드가 생성된다. 삭제는 수동으로 진행한다.


## 인증서 확인

```
[root@localhost ~]# certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: h2terran.site
    Serial Number: 3269f64a4838afda5a1c86866962b82e775
    Key Type: RSA
    Domains: *.h2terran.site h2terran.site
    Expiry Date: 2021-08-11 07:12:15+00:00 (VALID: 44 days)
    Certificate Path: /etc/letsencrypt/live/h2terran.site/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/h2terran.site/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

## 인증서 갱신

발급 받은 인증서의 유효기간은 발급 받은 이후로 90일까지이며, 기본적으로 인증서 만료일이 한달 이내로 남은 시점일 때부터 갱신이 가능하다.

아래의 파일에서 갱신 가능일을 수정할 수 있다.
```
[root@localhost ~]# vi /etc/letsencrypt/renewal/www10.h2terran.xyz.conf 
# renew_before_expiry = 30 days
-> 아래와 같이 변경
renew_before_expiry = 90 days
```
<br>
crond를 이용하여 매월 1일에 인증서를 재발급받고 아파치 적용을 위해 재실행을 진행하게 설정한다.

```
[root@localhost ~]# crontab -l
0 0 1 * * certbot renew --apache-ctl=/usr/local/apache2/bin/apachectl
```

## 참고 자료

- 플러그인 설치

  <a href="https://certbot.eff.org/lets-encrypt/centosrhel7-nginx#wildcard">
	https://certbot.eff.org/lets-encrypt/centosrhel7-nginx#wildcard
	</a>
	
- certbot-dns-rfc2136 플러그인 사용 설명서

  <a href="https://certbot-dns-rfc2136.readthedocs.io/en/latest/">
	https://certbot-dns-rfc2136.readthedocs.io/en/latest/
	</a>	

- certbot-dns 플러그인 목록

  <a href="https://certbot.eff.org/docs/using.html#dns-plugins">
	https://certbot.eff.org/docs/using.html#dns-plugins
	</a>
  