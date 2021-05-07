---
title:  "[MySQL/MariaDB] Replication with TLS"
excerpt: "MySQL/MariaDB 서버에서 TLS를 적용한 Replication 설정"
header:
  overlay_color: "#333"
  actions:
    - label: "MySQL공식사이트"
      url: "https://www.mysql.com/"
categories:
  - mysql
tags:
  - OpenSSL
  - GnuTLS
  - 서버보안
last_modified_at : 2021-03-26
last_modified_at_2 : 2021-03-26
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## 개요

<a href="https://susoterran.github.io/mysql/mysql_ssl/">[MySQL/MariaDB] MySQL/MariaDB SSL 적용하기
</a>

위의 글에서도 설명했듯이 기본적으로 MySQL/MariaDB는 서버-클라이언트 사이에서 평문 통신을 한다.

Replication에서도 마찬가지로 Master-Slave 서버 사이에서 평문 통신을 한다.

보통 Master-Slave 서버 사이의 통신은 Private 네트워크를 사용하기 때문에 평문 통신을 한다고 해서 크게 문제가 되지 않는다.

하지만 Public 네트워크를 이용하여 Master-Slave 서버 간 데이터 전송을 하는 환경에 경우 데이터 노출 문제가 생긴다.

AWS와 같은 클라우드 환경에서는 VPC라는 가상의 Private 네트워크를 부여받지만, 실질적으론 다른 사용자들과 자원을 공유하는 형태이기 때문에 보안 취약점에 의한 데이터 노출에 대한 우려가 있어 TLS를 이용하여 DB 서버 간 데이터 전송 시 암호화하도록 설정할 수 있다.


## Replication에서 사용할 사설 인증서 생성

아래의 링크 참고

<a href="https://susoterran.github.io/mysql/mysql_privatecertificate/">[MySQL/MariaDB] MySQL/MariaDB에서 사용할 사설 인증서 생성하기</a>

GnuTLS를 이용하여 설치한 MariaDB 10.4.18 이후 버전은 GnuTLS로 사설 인증서를 생성해야 한다.
  - 해당 버전부터 wolfSSL 버전이 4.4.0에서 4.6.0으로 변경되었으며 위 링크에서 생성한 OpenSSL로 생성한 사설 인증서를 정상적으로 인식하질 않는다.

사설 인증서는 MySQL 서버나 클라이언트 어디에서 생성하든 상관없다.

다만 사설 인증서 생성할 때 필요한 OpenSSL 또는 GunTLS가 설치되어 있어야 한다.

## 테스트 환경

- CentOS 7.9
- OpenSSL 1.1.1i
- MariaDB 10.3.23
- Master 서버 : 192.168.10.11
- Slave 서버 : 192.168.10.12

## Master 서버 설정

### 사설 인증서 확인

```
[root@localhost work]# cd /etc/ssl/mysql
[root@localhost mysql]# chown mysql. *
[root@localhost mysql]# ls -al
drwxr-xr-x  2 root  root   172  7월 18 11:35 .
drwxr-xr-x. 3 root  root    32  7월 18 11:17 ..
-rw-r--r--  1 mysql mysql 1675  7월 18 11:26 ca-key.pem
-rw-r--r--  1 mysql mysql 1314  7월 18 11:27 ca.pem
-rw-r--r--  1 mysql mysql 1176  7월 18 11:35 client-cert.pem
-rw-------  1 mysql mysql 1679  7월 18 11:35 client-key.pem
-rw-r--r--  1 mysql mysql  997  7월 18 11:31 client-req.pem
-rw-r--r--  1 mysql mysql 1168  7월 18 11:27 server-cert.pem
-rw-------  1 mysql mysql 1679  7월 18 11:27 server-key.pem
-rw-r--r--  1 mysql mysql  993  7월 18 11:27 server-req.pem
```

### /etc/my.cnf 설정 및 MySQL 재실행

여기에선 전체 DB에 대해서 Replication 설정하는 것을 가정하여 진행하겠다.

```
[root@localhost mysql]# vi /etc/my.cnf
[mysqld]
~~~
### Replication configure
log_bin=mysql-bin
server-id=1

### SSL/TLS
ssl_cert = /etc/ssl/mysql/server-cert.pem
ssl_key = /etc/ssl/mysql/server-key.pem
ssl_ca = /etc/ssl/mysql/ca.pem
~~~

[root@localhost mysql]# systemctl restart mysqld.service 

```

### Replication 용도의 계정 생성

```
[root@localhost ~]# /usr/local/mysql/bin/mysql -u root -p
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.10.11' IDENTIFIED BY 'asdf1234' REQUIRE SSL;
```

### mysqldump를 이용하여 Master 서버 백업

```
[root@localhost ~]# /usr/local/mysql/bin/mysqldump -u root -p --all-databases --master-data=2 > /home/jslim/master.sql
```

- &#45;&#45;master-data=2 : 백업 결과 파일에 백업이 수행된 시점의 바이너리 로그 파일과 포지션 정보를 CHANGE MASTER TO 구문 형태로 기록한다. 값이 1이라면 CHANGE MASTER TO 구문 앞에 주석 없이 실행 가능한 형태로 기록된다. 값이 2라면 주석을 포함해서 기록되므로 백업 파일로 복원을 수행할 때 자동으로 실행되지 않는다.

Master 서버에서 백업 시점 정보를 확인한다.

```
MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      1048 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```

### Master 설정 확인

```
[root@localhost mysql]# /usr/local/mysql/bin/mysql -u root -p

MariaDB [(none)]> SHOW VARIABLES LIKE 'have_ssl';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| have_ssl      | YES   |
+---------------+-------+

```

## Slave 서버 설정

### 사설 인증서 확인

```
[root@localhost work]# cd /etc/ssl/mysql
[root@localhost mysql]# ls -al
drwxr-xr-x  2 root  root  4096 2020-07-14 12:51 .
drwxr-xr-x. 3 root  root  4096 2020-07-14 12:42 ..
-rw-r--r--  1 mysql mysql 1675 2020-07-14 12:50 ca.pem
-rw-r--r--  1 mysql mysql 1176 2020-07-14 12:50 client-cert.pem
-rw-------  1 mysql mysql 1679 2020-07-14 12:50 client-key.pem
-rw-r--r--  1 mysql mysql  997 2020-07-14 12:50 client-req.pem
```

### hosts 설정
```
[root@localhost jslim]# vi /etc/hosts
192.168.10.11 mysql-server
```

### /etc/my.cnf 설정 및 MySQL 재실행

```
[root@localhost mysql]# vi /etc/my.cnf
[mysqld]
~~~
### Replication configure
[mysqld]
server-id=2
relay-log=mysqld-relay-bin
read_only

### SSL/TLS
ssl_cert = /etc/ssl/mysql/client-cert.pem
ssl_key = /etc/ssl/mysql/client-key.pem
ssl_ca = /etc/ssl/mysql/ca.pem
~~~

[root@localhost mysql]# systemctl restart mysqld.service 

```

### Master와의 연결 설정

```
MariaDB [(none)]> CHANGE MASTER TO
  -> MASTER_HOST='192.168.10.10',
	-> MASTER_USER='repl',
	-> MASTER_PASSWORD='asdf1234',
	-> MASTER_PORT=3306,
	-> MASTER_LOG_FILE='mysql-bin.000001',
	-> MASTER_LOG_POS=1048,
  -> MASTER_SSL_VERIFY_SERVER_CERT=1,
  -> MASTER_SSL=1;

MariaDB [(none)]> start slave;

MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
	                Slave_IO_State: Waiting for master to send event
	                   Master_Host: mysql-server
	                   Master_User: pdns_repl
	                   Master_Port: 3306
	                 Connect_Retry: 60
	               Master_Log_File: mysql-bin.000003
	           Read_Master_Log_Pos: 2802
	                Relay_Log_File: localhost-relay-bin.000002
	                 Relay_Log_Pos: 555
	         Relay_Master_Log_File: mysql-bin.000003
	              Slave_IO_Running: Yes
	             Slave_SQL_Running: Yes
	               Replicate_Do_DB: powergslb
	           Replicate_Ignore_DB: 
	            Replicate_Do_Table: 
	        Replicate_Ignore_Table: 
	       Replicate_Wild_Do_Table: 
	   Replicate_Wild_Ignore_Table: 
	                    Last_Errno: 0
	                    Last_Error: 
	                  Skip_Counter: 0
	           Exec_Master_Log_Pos: 2802
	               Relay_Log_Space: 868
	               Until_Condition: None
	                Until_Log_File: 
	                 Until_Log_Pos: 0
	            Master_SSL_Allowed: Yes
	            Master_SSL_CA_File: /etc/ssl/mysql/ca.pem
	            Master_SSL_CA_Path: 
	               Master_SSL_Cert: /etc/ssl/mysql/client-cert.pem
	             Master_SSL_Cipher: 
	                Master_SSL_Key: /etc/ssl/mysql/client-key.pem
	         Seconds_Behind_Master: 0
	 Master_SSL_Verify_Server_Cert: Yes
	                 Last_IO_Errno: 0
	                 Last_IO_Error: 
	                Last_SQL_Errno: 0
	                Last_SQL_Error: 
	   Replicate_Ignore_Server_Ids: 
	              Master_Server_Id: 1
	                Master_SSL_Crl: /etc/ssl/mysql/ca.pem
	            Master_SSL_Crlpath: 
	                    Using_Gtid: No
	                   Gtid_IO_Pos: 
	       Replicate_Do_Domain_Ids: 
	   Replicate_Ignore_Domain_Ids: 
	                 Parallel_Mode: conservative
	                     SQL_Delay: 0
	           SQL_Remaining_Delay: NULL
	       Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
	              Slave_DDL_Groups: 2
	Slave_Non_Transactional_Groups: 0
	    Slave_Transactional_Groups: 3
```

## 패킷 확인

SSL 통신 설정 전에는 아래와 같이 SQL 쿼리가 그대로 패킷에 나타난다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-03-26-mysql_replication_with_tls/tls_off.jpg?raw=true"></center>
<br>

설정 이후에는 아래와 같이 패킷으로 SQL 쿼리를 확인할 수가 없다.

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-03-26-mysql_replication_with_tls/tls_on.jpg?raw=true"></center>
<br>

## 트러블 슈팅

### SSL certificate validation failure


MySQL 콘솔에서 "show slave status\G; " 명령어 입력 후 아래의 에러메시지가 출력되면서 Replication 연결이 실패한다.
	
Last_IO_Error: error connecting to master 'pdns_repl@192.168.10.11:3306' - retry-time: 60  maximum-retries: 86400  message: SSL connection error: SSL certificate validation failure

#### 원인

해당 문제는 Master 서버의 호스트 네임과 Master 서버에서 사용하는 서버 인증서의 CN (Common Name) 간의 정보 불일치로 발생한다.

이는 'MASTER_SSL_VERIFY_SERVER_CERT' 옵션이 활성화 되면서 Slave에서 Master와 통신을 할 때 Slave에 설정된 Master 서버의 호스트 네임과 Master 서버 인증서에 포함된 CN을 비교하게 된다.


#### 해결 방법

Master 서버 인증서 생성 시 입력했던 CN을 호스트 네임에 맞게 설정한다.
	
Slave 서버에서 아래와 같이 설정을 변경해준다.
	
```
[root@localhost jslim]# vi /etc/hosts
192.168.10.11 mariadb-master
	
[root@localhost jslim]# /usr/local/mysql/bin/mysql -u root -p
MariaDB [(none)]> stop slave;
	
MariaDB [(none)]> CHANGE MASTER TO
-> master_host='mariadb-master';
	
MariaDB [(none)]> start slave;
	
MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
Slave_IO_State: Waiting for master to send event
Master_Host: mariadb-master
```

#### ASN: before date in the future

MySQL 콘솔에서 "show slave status\G; " 명령어 입력 후 아래의 에러메시지가 출력되면서 Replication 연결이 실패한다.
	
Last_IO_Error: error connecting to master 'pdns_repl@192.168.10.11:3306' - retry-time: 60  maximum-retries: 86400  message: SSL connection error: ASN: before date in the future

#### 원인

Slave 서버 또는 Master 서버 간의 시간 동기화 문제에 의해 발생한다.


#### 해결 방법

Slave 서버와 Master 서버의 시간을 현재 시간에 맞추어 동기화한다.

```	
[root@localhost ~]# rdate -s time.bora.net
```

## 참고 자료
- MariaDB Replication에 SSL/TLS 통신 설정

    <a href="https://mariadb.com/kb/en/replication-with-secure-connections/">https://mariadb.com/kb/en/replication-with-secure-connections/
    </a>

- change master to 옵션

    <a href="https://mariadb.com/kb/en/replication-with-secure-connections/">https://mariadb.com/kb/en/change-master-to/
    </a>

- MariaDB에서 SSL/TLS 설정

    <a href="https://mariadb.com/kb/en/replication-with-secure-connections/">https://mariadb.com/kb/en/secure-connections-overview/
    </a>

- Server Certificate Verification

    <a href="https://mariadb.com/kb/en/replication-with-secure-connections/">https://mariadb.com/kb/en/secure-connections-overview/#server-certificate-verification
    </a>