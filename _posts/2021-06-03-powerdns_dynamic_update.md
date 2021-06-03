---
title:  "[PowerDNS] DNS Dynamic Update"
excerpt: "DNS Dynamic Update 설정"
header:
  overlay_color: "#333"
  actions:
    - label: "PowerDNS 공식사이트"
      url: "https://www.powerdns.com/"
categories:
  - other
tags:
  - PowerDNS
last_modified_at : 2021-06-03
last_modified_at_2 : 2021-06-03
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## 개요

앞서 PowerDNS 구성하면서 도메인 정보를 보관하는 백엔드로 MySQL과 같은 DBMS를 사용했으며, PDNS Manager와 같은 프론트엔드를 구성하여 웹 상에서 도메인 정보를 관리할 수 있도록 구성했다.

이와 별개로 Dynamic DNS Update (rfc2136)을 이용하여 도메인 정보를 관리할 수 있는데 이는 웹으로 구성된 프론트엔드를 대체할 수 있다.
	
여기서는 Dynamic Update와 tsig를 구성하는 방법에 대해 알아보고 nsupdate를 이용하여 도메인 정보를 수정하는 방법을 알아본다.

## pdns.conf 설정

```
[root@localhost ~]# vi /etc/pdns/pdns.conf 
~~ (생략) ~~
# allow-dnsupdate-from  A global setting to allow DNS updates from these IP ranges.
#
allow-dnsupdate-from=127.0.0.0/8,::1
~~ (생략) ~~
# dnsupdate     Enable/Disable DNS update (RFC2136) support. Default is no.
#
dnsupdate=yes
~~ (생략) ~~
	
[root@localhost ~]# systemctl restart pdns
```

"allow-dnsupdate-from" 옵션은 Dynamic Update를 사용할 수 있는 클라이언트를 설정하는 부분이다. 

Dynamic Update 가 편하긴 하지만 인증이나 전송 데이터에 대한 보안 요소가 부족하다. 따라서 Dynamic Update를 이용할 땐 네임서버 자체나 네임서버와 클라이언트 간 네트워크 구간이 안전할 경우에 사용하는 것이 좋다.

여기선 네임서버 자체에서만 Dynamic Update를 사용할 수 있게 설정한다.

"dnsupdate" 옵션은 PowerDNS의 Dynamic Update 기능을 활성화하는 설정이다.


## 도메인 메타 데이터 설정

pdnsutil를 이용하여 Dynamic Update 사용 시 필요한 메타 데이터를 생성한다.

해당 프로그램은 PowerDNS 설치 시 pdns 데몬과 같이 설치된다.

### ALLOW-DNSUPDATE-FROM

해당 메타 데이터는 특정 네트워크 대역에서 보낸 h2terran.site 도메인 주소에 대한 Dynamic Update 메시지만 수신하도록 설정하는 값이다.

```
[root@localhost ~]# pdnsutil set-meta h2terran.site ALLOW-DNSUPDATE-FROM 127.0.0.1/32
```

### TSIG-ALLOW-DNSUPDATE

해당 메타 데이터는 Dynamic Update를 수행할 때 필요한 TSIG 키를 생성한다.

```
[root@localhost ~]# pdnsutil generate-tsig-key test hmac-sha512
~~~ (생략) ~~~
Create new TSIG key test hmac-sha512 QeMkvUbf9hK+jgTEwbVN1MfyN2gplBIuaJFC36aqO5BShWtkCoDGvnRqy7zD2y0NP5lGn0OEEWL5fga1IcAW1g==
```

- generate-tsig-key : tsig key를 생성하기 위한 옵션
- test : 여기서 설정한 값을 뒤에서 설정할 해쉬함수를 이용하여 tsig key를 생성한다
- hmac-sha512 : 앞서 입력한 평문 데이터를 통해 tsig key를 생성하는데, 이때 사용할 해쉬 함수를 설정
	- hmac-md5의 경우 보안 취약점이 있기 때문에 절대 사용하지 말 것.

<br>
이렇게 생성한 키를 test라는 이름으로 DB에 추가한다.

```
[root@localhost ~]# pdnsutil import-tsig-key test hmac-sha512 'QeMkvUbf9hK+jgTEwbVN1MfyN2gplBIuaJFC36aqO5BShWtkCoDGvnRqy7zD2y0NP5lGn0OEEWL5fga1IcAW1g=='
~~~ (생략) ~~~
Imported TSIG key test hmac-sha512
```

<br>
test로 등록한 tsig 키를 이용하여 Dynamic Update가 가능하도록 TSIG-ALLOW-DNSUPDATE 메타 데이터에 추가한다.

여기서 추가한 tsig 키는 h2terran.site 도메인 주소에 대해 Dynamic Update를 할 때만 사용 가능하다.

```
[root@localhost ~]# pdnsutil set-meta h2terran.site TSIG-ALLOW-DNSUPDATE test
~~~ (생략) ~~~
Set 'h2terran.site' meta TSIG-ALLOW-DNSUPDATE = test
```

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-06-03-powerdns_dynamic_update/01_domainmetadata.jpg?raw=true"></center>
<br>


<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-06-03-powerdns_dynamic_update/02_tsigkey.jpg?raw=true"></center>
<br>

h2terran.site (domain_id = 2)의 TSIG-ALLOW-DNSUPDATE 인 test와 tsigkeys의 test와 연동되어 사용된다. 


## Dynamic Update 테스트

### 레코드 추가 테스트

nsupdate 명령을 이용하여 동적 업데이트를 진행한다. 

key 에 알고리즘을 명시하지 않으면 hmac-md5으로 인식하게 되니 주의한다.

```
[root@localhost letsencrypt]# nsupdate <<!
server 127.0.0.1 53
zone h2terran.site
update add test1.h2terran.site 300 A IP주소
key hmac-sha512:test QeMkvUbf9hK+jgTEwbVN1MfyN2gplBIuaJFC36aqO5BShWtkCoDGvnRqy7zD2y0NP5lGn0OEEWL5fga1IcAW1g==
send
!
```

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-06-03-powerdns_dynamic_update/03_record_add.jpg?raw=true"></center>
<br>


### 레코드 삭제 테스트

```
[root@localhost ~]# nsupdate <<!             
server 127.0.0.1 53
zone h2terran.site
update del test1.h2terran.site 300 A IP주소
key hmac-sha512:test QeMkvUbf9hK+jgTEwbVN1MfyN2gplBIuaJFC36aqO5BShWtkCoDGvnRqy7zD2y0NP5lGn0OEEWL5fga1IcAW1g==
send
!
```

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-06-03-powerdns_dynamic_update/04_record_del.jpg?raw=true"></center>
<br>


## 마무리

Dynamic Update는 PDNS Manager와 같은 웹 프론트엔드에 비해 보안이나 도메인 정보를 관리하는 측면에서 별로 도움이 되지 않는 기능이다.

하지만 묘한 곳에서 엄청난 활약을 할 수 있는데, Let`s Encrypt의 와일드카드 인증서를 이 기능을 이용하여 자동으로 발급받을 수 있다.

이와 관련된 내용은 앞으로 블로그에 추가할 예정이다.


## 참고 자료
- Dynamic Update (RFC2136)

	<a href="https://doc.powerdns.com/authoritative/dnsupdate.html#setting-up-powerdns">
	https://doc.powerdns.com/authoritative/dnsupdate.html#setting-up-powerdns
	</a>

- pdnsutil

	<a href="https://doc.powerdns.com/authoritative/manpages/pdnsutil.1.html">
	https://doc.powerdns.com/authoritative/manpages/pdnsutil.1.html
	</a>

- TSIG

	<a href="https://doc.powerdns.com/authoritative/tsig.html">
	https://doc.powerdns.com/authoritative/tsig.html
	</a>

- nsupdate 명령으로 원격지에서 마스터 네임서버 제어하기

	<a href="https://crazyit.tistory.com/entry/nsupdate-명령으로-원격지에서-마스터-네임서버-제어하기">
	https://crazyit.tistory.com/entry/nsupdate-명령으로-원격지에서-마스터-네임서버-제어하기
	</a>