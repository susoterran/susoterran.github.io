---
title:  "[PowerDNS] PowerDNS의 기본 동기화 기능의 문제점"
excerpt: "PowerDNS의 기본 동기화 기능의 문제점"
header:
  overlay_color: "#333"
  actions:
    - label: "PowerDNS 공식사이트"
      url: "https://www.powerdns.com/"
categories:
  - other
tags:
  - PowerDNS
last_modified_at : 2021-08-01
last_modified_at_2 : 2021-08-01
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## 개요

DNS 서비스를 제공하는 Name 서버를 구축할 때 주/보조 형태의 구성을 사용하게 되는데, 이는 주 서버에 저장되어 있는 도메인 영역 정보를 일정한 주기에 따라 보조 서버에 동기화 시킴으로써 DNS 쿼리에 대한 트래픽 분산과 서버 장애에 대한 서비스 내구성을 높인다.

DNS 영역 전송 (DNS Zone Transfer)은 크게 3가지 방법으로 주 서버의 영역 데이터를 보조 서버로 동기화시킨다.

- AXFR (Authoritative Transfer) : 보조 서버가 지정된 주기(SOA 레코드의 refresh)에 따라 최신 정보를 요청하고, 최신 정보가 있을 시 주 서버가 보조 서버에 영역 데이터를 전달한다. 이때 주 서버는 요청 받은 영역의 모든 정보를 전달한다.

- IXFR (Incremental Zone transfer) : AXFR과 달리 영역에 대해 변경된 내용만 보조 서버에 전송하는 방식이다.

- Notify 동기화 : 주 서버의 영역 데이터에 변화가 발생 (SOA 레코드의 serial 값 증가)하면 주 서버가 보조 서버에 즉시 변화를 알린다. 변화를 감지한 보조 서버는 AXFR 또는 IXFR 방식으로 데이터를 주 서버로 부터 전달받는다.

PowerDNS에서는 기본적으로 주-보조 서버 구성을 지원한다. 이때 사용하는 것이 PowerDNS DB내에 있는 supermasters 테이블이다.

해당 테이블의 값을 변경하는 방법은 DB를 통해 변경하는 방법과 FrontEnd를 통해 변경하는 방법이 있다. 후자의 경우 poweradmin에서 변경이 가능하다.

이번 포스트에서는 PowerDNS에서 주-보조 서버간 데이터 동기화 설정을 하는 방법을 알아볼 것이며, 공식적으로 DBMS의 replication 기능을 사용할 것을 권장하는 이유에 대해 알아본다.

## 주-보조 서버 동기화 설정

### 주 서버에서의 설정

```
[root@localhost ~]# vi /etc/pdns/pdns.conf 
~
allow-axfr-ips=보조서버IP주소/32
disable-axfr=no
master=yes

[root@localhost ~]# systemctl restart pdns
```

### 보조 서버에서의 설정

```
[root@localhost ~]# vi /etc/pdns/pdns.conf 
~
slave=yes

[root@localhost ~]# systemctl restart pdns
```

### supermasters 설정

supermasters 설정은 보조 서버의 DB에서 진행한다.

```
[root@localhost ~]# /usr/local/mysql/bin/mysql -u root -p
mysql> use pdns;

mysql> INSERT INTO supermasters (ip, nameserver, account) VALUES ('주네임서버IP주소','ns1.30test.com','admin');
```

### 동기화 확인

주 서버에서 allow-axfr-ips에 보조 서버의 IP 주소를 설정하고, 보조 서버에서 supermasters에 주 서버의 IP를 설정하면, AXFR을 이용하여 영역 데이터를 동기화한다.

<b>[보조 서버에서 확인한 로그]</b>

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-08-01-powerdns_sync/01_axfr_log.jpg?raw=true">

주 서버와 보조 서버간의 동기화 주기는 보통 SOA 레코드의 refresh 에 설정된 값에 맞추어 진행된다. 하지만 PowerDNS의 경우 pdns.conf의 slave-cycle-interval 값에 맞추어 동기화가 진행되며 기본값은 60초이다.

## 동기화 설정의 문제점

주 서버에서 새로 영역을 생성하거나 영역 정보를 변경했을 경우는 정상적으로 동기화된다.

하지만 주 서버에서 영역을 삭제했을 경우 이 내용이 보조 서버에는 적용되지 않는데, AXFR 프로토콜은 영역 삭제에 대한 동기화를 지원하지 않기 때문이다.

공식적으로 이 문제를 해결하기 위한 방법으로 PHP로 작성한 스크립트를 사용할 것을 제시하고 있다.

해당 스크립트를 실행함으로써 주 서버에서 삭제된 영역을 검색하여, 보조 서버에 이를 적용하는 과정을 통해 영역 삭제에 대한 동기화 작업을 진행한다.

아래는 스크립트의 내용이며, 보조 서버에서 실행시키면 된다. 중간에 보조 서버의 MySQL 접속 정보만 입력하면 된다.

```
<?php
############################### LICENSE ###################################
# 1. You are allowed to do everything with it you like, but with the exceptions below:
# 1a. Don't use it to do illegal things.
# 1b. You didn't create it (Mark Scholten, mark@mscholten.eu, did create it), so don't say to others that you did create it.
# 2. This script/software comes without guaranteed support or any other guarantee; use it on your own risk! The creator isn't responsible for using it and doesn't accept any responsibility
# 3. Support might be available (look at mscholten.eu for the ways to get support and to see if support is available, you can also try to email to mark@mscholten.eu if support on mscholten.eu is not available).
# 4. Commercial use is allowed.
# 5. Distribution is allowed, asking money for it is allowed. Using it in other scripts is allowed as long as you respect point 1/1a/1b in this license.
############################### END LICENSE ################################

// configuration section
mysql_connect(  'localhost', // host
	'mysql username', //username
	'mysql pass'); //pass
mysql_select_db('mysql database');
$test = 0; // change to 0 to delete records/domains, if set to something else it will not delete anything
$verbose = 1; // set to 1 to be verbose (output domainnames that are deleted/are to be deleted (depending on the $test setting)

// no configuration below this line

	function is_stil_active($domain,$server){
		$axfr = shell_exec("dig AXFR ".$domain." @".$server."");
		$explode = explode("XFR size:",$axfr);
		if(isset($explode['1'])){
			return TRUE;
		}else{
			return FALSE;
		}
	}

$sql3 = "SELECT `id`,`name`,`master` FROM domains WHERE `type`='SLAVE'";
$query = mysql_query($sql3) or die(mysql_error());
if(mysql_num_rows($query) == FALSE){
}else{
	while($record = mysql_fetch_object($query)){
		if(!is_stil_active($record->name,$record->master)){
			if($test === 0){
				mysql_query("DELETE FROM domains WHERE id='".$record->id."'") or die(mysql_error());
				mysql_query("DELETE FROM records WHERE domain_id='".$record->id."'") or die(mysql_error());
			}
			if($verbose === 1){
				echo $record->name."
";
	}
	}
	}
	if($verbose === 1){
		echo "Done
";
	}
}
?>
```

Bind가 각 영역 정보를 파일 형태로 관리하는 것과 달리 PowerDNS는 MySQL과 같은 DBMS를 이용하여 영역 정보를 관리하도록 설계되었다.

DBMS를 통해 PowerDNS를 구축하였다면 DBMS에서 제공하는 Replication 기능을 사용하여 데이터 동기화를 진행하는 것이 기존의 IXFR/AXFR 동기화 방식에 비해 데이터 유실 (DBMS에서 트랜잭션을 보장하기 때문)과 유출 (Replcation SSL 사용할 경우)에 대비할 수 있다.


## 참고자료

<a href="https://github.com/PowerDNS/pdns/wiki/FAQ">
https://github.com/PowerDNS/pdns/wiki/FAQ
</a>

<a href="https://marc.info/?l=pdns-users&m=128034976720321">
https://marc.info/?l=pdns-users&m=128034976720321
</a>

<a href="https://doc.powerdns.com/authoritative/modes-of-operation.html">
https://doc.powerdns.com/authoritative/modes-of-operation.html
</a>