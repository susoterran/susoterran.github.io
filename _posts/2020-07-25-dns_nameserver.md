---
title:  "[PowerDNS] DNS 이론 2 - 네임 서버"
excerpt: "네트워크에서 도메인이나 호스트 이름을 숫자로 된 IP 주소로 해석해주는 TCP/IP 네트워크 서비스"
header:
  overlay_color: "#333"
  actions:
--label: "PowerDNS 공식사이트"
--url: "https://www.powerdns.com/"
categories:
  - other
tags:
  - DNS
  - PowerDNS
last_modified_at: 2020-07-25
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---
## 네임 서버의 종류

네임 서버는 클라이언트의 요청에 대한 DNS 해석 결과를 전달하는 서버이다. 예를 들어 클라이언트가 www.naver.com의 IP 주소를 요청하면 네임 서버는 125.209.222.142를 응답해 줄 것이다.

앞에서도 이야기했듯이 네임 서버는 무수히 많은데, 이들 네임 서버들도 역할에 따라 종류가 나뉜다. 여기서는 도메인 정보를 어떤 방식으로 가지고 있는지에 따른 네임 서버 종류에 대해 알아본다.

### 도메인 정보 데이터베이스의 권한 유무

여기서 도메인 Zone 데이터베이스는 네임 서버가 가지고 있는 도메인 정보의 묶음이라 보면 된다. naver 도메인 네임 스페이스에 위치한 네임 서버는 blog.naver.com, www.naver.com 등 naver.com 이 뒤에 붙는 도메인 주소들에 대한 IP 주소를 알고 있다. 이 정보들은 naver.com 라는 Zone 으로 묶이게 된다.

도메인 정보 데이터베이스는 네임 서버들에 따라 Zone 파일 (bind)이나 데이터베이스의 레코드 (PowerDNS)에 저장하여 관리한다.

1) 마스터 네임 서버 
- Master Name Server, Primary Name Server
- Zone 데이터베이스를 직접 구성하는 서버
- Zone 데이터베이스에 대한 권한을 가지고 있는 서버


2) 보조 네임 서버
- Slave Name Server, Secondary Name Server)
- 마스터 네임 서버의 Zone 정보를 동기화 (복사)하여 Zone 데이터베이스를 구성하는 서버
- 마스터의 백업 서버가 아니다 
  - 마스터 서버와 동시에 서비스를 하지만 Zone에 대한 정보만 동기화 받는다.
  - 주기적인 동기화 과정을 통해 마스터 서버와 동일한 정보를 유지

※ 여기서의 백업은 특정 시점의 상태를 저장하고 있는 것을 말한다. 보조 네임 서버는 마스터 네임 서버의 정보를 '실시간'으로 동기화하므로 특정 시점의 상태를 저장하는 백업의 의미와는 거리가 멀다 ('클러스터 (Cluster)'라는 개념을 미리 알아둔다면 앞으로도 많은 도움이 될 것이다)

3) 전달자
- Forwarder
- 실제 Zone에 대한 정보를 가지고 있지 않지만 클라이언트의 요청을 특정 네임 서버로 전달하는 역할을 수행하는 서버


## DNS Zone

DNS Zone (DNS 영역, 여기서는 그냥 영역이라고 설명하겠다) 도메인 네임 스페이스의 하위 단위이며, 도메인 주소를 관리하는 실질적인 단위이다.

예를 들어, 앞선 포스트에 있는 트리 구조를 생각해보자. naver.com의 정보 (naver.com 영역의 정보)는 해당 사진 중 어디에 존재할까? naver 노드에 있다. naver 노드에 있는 naver.com 영역에는 www.naver.com과 blog.naver.com에 대한 정보가 있다. 마찬가지로 google.com 영역에 대한 정보는 google 노드에 있으며, google.com 영역에는 www.google.com에 대한 정보가 있다. 여기서 노드라고 표현했지만 이는 개념적인 표현이고, 실제로는 네임 서버를 의미한다.

다시 말하면, naver 네임 서버에는 naver.com 영역 정보가 존재하며, 해당 영역에는 www.naver.com과 blog.naver.com의 정보가 들어 있다.

영역은 크게 2가지로 나뉜다.
- 주 영역 : 읽고 쓰기가 가능한 영역
- 보조 영역 : 읽기 전용 영역. 주 영역의 데이터를 전부 복사하여 가지고 오며, 주 영역이 다운되면 주 영역 대신 사용할 수 있다. 주 영역 없이 사용자 요청에 응답이 가능하다.

IT에서 가장 중요한 것 중 하나가 가용성이다. 가용성이란 '시스템이 정상적으로 사용 가능한 정도'를 말하며, 가동률이라고 생각하면 된다 (참고 : 위키백과). naver.com 영역 정보를 가진 네임 서버를 1대만 운영한다고 해보자. 만약 이 네임 서버의 전원이 꺼지거나 고장이 난다면 전세계 모든 사람들은 naver.com 이하에 있는 모든 사이트에 대해 접속을 할 수 없게 된다. www.naver.com 도메인 주소에 대한 IP 주소를 응답할 수 있는 네임 서버는 전세계에 오직 1대 밖에 없었기 때문이다.

만약 네임 서버가 2대 있고, 1대를 마스터 네임 서버 (주 영역을 가지고 있는 네임 서버), 다른 1대를 보조 네임 서버 (보조 영역을 가지고 있는 네임 서버) 로서 관리하고 있다고 해보자. 마스터 네임 서버가 작동하지 않는다고 하더라도, 보조 네임 서버가 있기 때문에 naver.com 영역에 대한 DNS 조회에는 문제가 없다. 앞선 예와는 달리 가용성을 높일 수 있다.

그럼 마스터 네임 서버와 보조 네임 서버는 어떻게 정보를 동기화 할까? 이는 뒤에서 살펴볼 SOA 레코드와 관련이 있지만, 여기서는 짧게만 정리하고 넘어가겠다.

1) AXFR
- Authoritative Transfer
- 기본적인 영역 전송 방법
- 보조 네임서버가 지정된 주기(refresh)에 따라 serial 정보 요청을 전달
  - serial 값 : 영역 정보의 갱신, 수정, 삭제 등이 일어 났을 때 증가
- 마스터 네임 서버가 영역의 serial 정보를 전달한다.
- 보조 네임 서버에서 설정된 serial 보다 증가되었음을 확인하면 영역에 대한 전체 동기화 (AXFR)을 요청 한다.
- 마스터 네임 서버는 요청 받은 영역의 모든 정보를 전달한다.
- 문제점 : refresh의 주기 때문에 동기화 지연 시간이 발생 할 수 있다.

2) Notify 동기화
- Zone Transfer
- 기본적인 AXFR 동기화의 지연 시간 문제점을 해결
	- 영역의 변화가 발생 (serial 값 증가)하면 마스터 네임 서버가 즉시 변화를 알림 (Notify 메시지)
	- 영역의 변화를 신속하게 동기화 할 수 있다.
- 변화를 감지한 마스터 네임 서버가 Notify 메시지로 영역의 serial 정보를 전달
- 보조 네임 서버에서 설정된 serial보다 증가되었음을 확인하면 영역 동기화 (AXFR/IXFR)을 요청
- 마스터 네임 서버는 요청 받은 영역의 정보를 전달 (AXFR/IXFR)

3) IXFR
- Incremental Zone transfer
- 영역에 대한 전체 정보가 아닌 변경된 내용만 전송 받는 동기화 방식
- 동기화 과정에서 발생하는 네트워크 트래픽의 부하를 줄임
- Notify 메시지와 함께 사용
- 마스터와 보조 네임 서버 둘 다 IXFR 방식을 지원해야 한다.
- UDP/TCP 포트 53을 선택적으로 이용할 수 있다.

DNS 영역 동기화에 대해서 알고 있으면 좋지만 몰라도 상관없다. PowerDNS에서 제대로 구현이 안되어 있기 때문이다. 엥? 그럼 영역 정보를 무슨 수로 동기화한다는 걸까? 동기화할 수 있는 방법은 무궁무진하다. PowerDNS에서 제공하는 기능들을 사용할 수 있고, 아예 새로 커스텀마이징으로 만드는 방법도 있다. 힌트를 준다면 여기서는 DBMS를 이용할 것이다 (언제 PowerDNS의 영역 정보 동기화까지 쓸 수 있을런지...)


### DNS 레코드

DNS 레코드 (줄여서 레코드)는 DNS의 정보 단위이며, 실제 주소 해석에 사용되는 정보이다.

| 코드 | 이름 | 의미 |
|:---:|:---|:---|
|1|A (Adress) | 요청된 name의 IPv4 주소|
|28|AAAA | 요청된 name의 IPv6 주소|
|2|NS (Name Server) | 요청된 name의 이름 해석을 담당하는 네임 서버의 도메인 주소|
|5|CNAME (Canonical Name) | 시스템에 지정된 공식 도메인 주소에 대한 별칭 도메인 주소|
|6|SOA (Start of Authrity) | DNS 영역의 시작을 알림. Zone에 대한 동기화 설정|
|12|PTR (Pointer) | 요청된 IP 주소의 도메인 주소|
|15|MX (Mail eXchanger) | 메일서버 별칭에 대한 실제 도메인 주소|

앞서 영역 동기화는 SOA 레코드와 관련이 있다고 했는데, SOA 레코드의 구조를 살펴보자 (여기서의 필드 값은 bind에서 참조하였으며, PowerDNS에서도 거의 비슷하게 구현되었다)

```
$TTL 86400       ; 1 day
@         IN SOA    ns1.test.com. root.test.com.(
                    2020072501  ;serial
                    21600       ;refresh (6시간)
                    1800        ;retry (30분)
                    1209600     ;expire (2주)
                    86400       ;minimum (TTL, 1일)
                    )
```

| 필드  | 설명  |
|:---|:---|
|$TTL|캐시에 DNS 영역 정보를 저장하는 시간 (초단위)|
|@|영역 데이터에 대한 도메인 네임을 지칭|
|IN|클래스 이름. 일반적으로 IN (Internet)을 사용한다|
|SOA| 레코드 유형|
|ns1.test.com.|마스터 네임 서버의 도메인 주소|
|root.test.com.|관리자 이메일 주소|
|serial|영역 정보의 갱신여부를 표기. 갱신될 때마다 1씩 증가|
|refresh|보조 네임 서버에서 마스터 네임 서버의 영역 정보의 갱신여부 체크 간격|
|retry|갱신여부 체크가 실패했을 경우 재시도까지의 시간 간격|
|expire|갱신여부 체크가 연속으로 실패 시 보조 네임 서버의 영역 정보를 삭제시키기 까지의 간격|
|minimum|영역 정보의 TTL 값 (유효 기간)|