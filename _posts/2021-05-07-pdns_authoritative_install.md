---
title:  "[PowerDNS] Authoritative Server (4.3) 설치하기"
excerpt: "PowerDNS Authoritative Server 설치하기"
header:
  overlay_color: "#333"
  actions:
    - label: "PowerDNS 공식사이트"
      url: "https://www.powerdns.com/"
categories:
  - other
tags:
  - PowerDNS
last_modified_at : 2021-05-07
last_modified_at_2 : 2021-05-07
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

## PowerDNS

- C++로 제작된 DNS 서버 시스템
- GPL 라이선스를 사용
- MySQL, Postgre SQL, SQLite3 등 데이터베이스를 백엔드로 사용할 수 있다.
- 백엔드의 기능 (ex : MySQL Replication)을 통해 1차, 2차 네임 서버 구축이 가능하다.
- 주로 용도에 따라 Authoritative Server, Recursor, dnsdist 를 사용하게 된다.
	- Authoritative Server : 도메인 존 정보를 가지고 있는 네임 서버
	- Recursor : Cache 서버
	- dnsdist : PowerDNS 전용 로드 밸런서

## 테스트 환경

- CentOS 7.9.2009
- OpenSSL 1.1.1f
- httpd 2.4.46, pcre 8.43, apr 1.7.0, apr-util 1.6.1
- MySQL 5.7.30
- PHP 7.2.33
- PowerDNS 4.3

해당 포스트에서는 PowerDNS를 네임 서버로 사용하고, MySQL을 백엔드로 사용한다.

MySQL Replication을 이용하여 1차, 2차 네임 서버 구축도 가능한데, 기본적으로 1차, 2차에 PowerDNS 및 MySQL 설치를 진행하고 두 서버간에 Replication 설정을 하면 끝이다.

여기서는 Replication 설정은 제외하고 PowerDNS 설치와 MySQL 백엔드 구성까지만 진행하겠다.

## PowerDNS 설치

```
[root@localhost work]# yum install epel-release yum-plugin-priorities && curl -o /etc/yum.repos.d/powerdns-auth-43.repo https://repo.powerdns.com/repo-files/centos-auth-43.repo && yum install pdns
```

## MySQL 백엔드 설치

PowerDNS와 MySQL 연동하기 위해 사용하는 패키지를 설치한다.

```
[root@localhost ~]# yum -y install pdns-backend-mysql
```

## MySQL 설정

### PowerDNS용 계정 생성
```
[root@localhost ~]# /usr/local/mysql/bin/mysql -u root -p 
mysql> create database pdns;
mysql> grant all privileges on pdns.* to 'pdns'@'localhost' identified by 'asd123';
mysql> grant all privileges on pdns.* to 'pdns'@'127.0.0.1' identified by 'asd123';
mysql> flush privileges;
mysql> exit
```

### 데이터베이스 설치
```
[root@localhost ~]# vi schema.mysql.sql

CREATE TABLE domains (
  id                    INT AUTO_INCREMENT,
  name                  VARCHAR(255) NOT NULL,
  master                VARCHAR(128) DEFAULT NULL,
  last_check            INT DEFAULT NULL,
  type                  VARCHAR(6) NOT NULL,
  notified_serial       INT UNSIGNED DEFAULT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'utf8';
	
CREATE UNIQUE INDEX name_index ON domains(name);
	
CREATE TABLE records (
  id                    BIGINT AUTO_INCREMENT,
  domain_id             INT DEFAULT NULL,
  name                  VARCHAR(255) DEFAULT NULL,
  type                  VARCHAR(10) DEFAULT NULL,
  content               VARCHAR(12000) DEFAULT NULL,
  ttl                   INT DEFAULT NULL,
  prio                  INT DEFAULT NULL,
  disabled              TINYINT(1) DEFAULT 0,
  ordername             VARCHAR(255) BINARY DEFAULT NULL,
  auth                  TINYINT(1) DEFAULT 1,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'utf8';

CREATE INDEX nametype_index ON records(name,type);
CREATE INDEX domain_id ON records(domain_id);
CREATE INDEX ordername ON records (ordername);
	
CREATE TABLE supermasters (
  ip                    VARCHAR(64) NOT NULL,
  nameserver            VARCHAR(255) NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (ip, nameserver)
) Engine=InnoDB CHARACTER SET 'utf8';

CREATE TABLE comments (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  name                  VARCHAR(255) NOT NULL,
  type                  VARCHAR(10) NOT NULL,
  modified_at           INT NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  comment               TEXT CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'utf8';
	
CREATE INDEX comments_name_type_idx ON comments (name, type);
CREATE INDEX comments_order_idx ON comments (domain_id, modified_at);
	
CREATE TABLE domainmetadata (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  kind                  VARCHAR(32),
  content               TEXT,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'utf8';
	
CREATE INDEX domainmetadata_idx ON domainmetadata (domain_id, kind);
	
CREATE TABLE cryptokeys (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  flags                 INT NOT NULL,
  active                BOOL,
  published             BOOL DEFAULT 1,
  content               TEXT,
  PRIMARY KEY(id)
) Engine=InnoDB CHARACTER SET 'utf8';

CREATE INDEX domainidindex ON cryptokeys(domain_id);
	
CREATE TABLE tsigkeys (
  id                    INT AUTO_INCREMENT,
  name                  VARCHAR(255),
  algorithm             VARCHAR(50),
  secret                VARCHAR(255),
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'utf8';
	
CREATE UNIQUE INDEX namealgoindex ON tsigkeys(name, algorithm);
```

```
[root@localhost ~]# /usr/local/mysql/bin/mysql -u pdns -p pdns < schema.mysql.sql
```

```
[root@localhost ~]# /usr/local/mysql/bin/mysql -u pdns -p     
mysql> use pdns;
mysql> ALTER TABLE records ADD CONSTRAINT `records_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
mysql> ALTER TABLE comments ADD CONSTRAINT `comments_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
mysql> ALTER TABLE domainmetadata ADD CONSTRAINT `domainmetadata_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
mysql> ALTER TABLE cryptokeys ADD CONSTRAINT `cryptokeys_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
```

## PowerDNS 설정

여기서는 PowerDNS와 MySQL 간 연동하는 설정만 추가하였다.

추후 로그와 보안 설정에 대해 포스트를 업로드하겠다.

```
[root@localhost ~]# vi /etc/pdns/pdns.conf
(아래 내용 추가)
# launch=
launch=gmysql
gmysql-host=127.0.0.1
gmysql-user=pdns
gmysql-password=asd123
gmysql-dbname=pdns

[root@localhost ~]# chown pdns. /etc/pdns/pdns.conf
[root@localhost ~]# chmod 400 /etc/pdns/pdns.conf
```

## PowerDNS 실행
```
[root@localhost ~]# systemctl enable pdns
[root@localhost ~]# systemctl start pdns
```

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-07-pdns_authoritative_install/powerdns_port.jpg?raw=true"></center>
<br>

<br>
<center><img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2021-05-07-pdns_authoritative_install/powerdns_process.jpg?raw=true"></center>
<br>


## 참고 자료
- PowerDNS 공식 문서

    <a href="https://doc.powerdns.com/authoritative/installation.html">https://doc.powerdns.com/authoritative/installation.html
    </a>

- MySQL용 DB SQL

    <a href="https://github.com/PowerDNS/pdns/blob/rel/auth-4.3.x/modules/gmysqlbackend/schema.mysql.sql">https://github.com/PowerDNS/pdns/blob/rel/auth-4.3.x/modules/gmysqlbackend/schema.mysql.sql
    </a>