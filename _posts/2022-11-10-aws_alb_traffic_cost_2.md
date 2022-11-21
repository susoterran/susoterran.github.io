---
title:  "[AWS] 아리송한 VPC 내부 트래픽 비용 산정 방식 2"
excerpt: "인터넷 경계 ALB를 통한 내부 트래픽 비용은 어떻게 산출할까?"
header:
  overlay_color: "#333"
categories:
  - other
tags:
  - AWS
last_modified_at : 2022-11-20
last_modified_at_2 : 2022-11-20
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# 인터넷 경계 ALB를 통한 내부 트래픽 비용은 어떻게 산출할까?

## 테스트 방법

- service-1, service-2 인스턴스를 생성한다. 이들 인스턴스는 같은 Private subnet에 위치한다.
- service-2에 nginx를 설치하고, CentOS 7.9 ISO 파일을 업로드 한다.
- 인터넷 경계 ALB를 생성하고 타겟그룹에 service-2를 추가한다.
- service-1에서 ALB의 주소로 접속하여 ISO 파일을 다운로드 받는다.
- 이후 AWS 청구서에 트래픽 사용량이 어떻게 나오는지 확인한다.

## 테스트 환경

- service-1 : 10.0.1.225
- service-2 : 10.0.1.43
- ALB
    - jslim-test-1773445617.ap-northeast-2.elb.amazonaws.com
    - 3.36.135.137
- 테스트 전 비용 및 트래픽 사용량

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_2.jpg?raw=true">

## 테스트 진행

- service-1 에서 service-2에 있는 ISO 파일을 wget을 이용하여 다운로드한다. 이때 접속 URL은 ALB의 URL이다. 
- 4.4GB 파일을 3번 다운로드 받음

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_3.jpg?raw=true">

## 테스트 후 지표 확인

1. NAT Gateway

- BytesInFromDestination
    - NAT 게이트웨이가 대상으로부터 수신한 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시
    - 두번째 봉우리는 service-1의 요청으로 service-2의 ISO 파일을 전송하는 트래픽을 측정한 것이다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_4.jpg?raw=true">

- BytesInFromSource
    - NAT 게이트웨이가 VPC 내 클라이언트로부터 수신한 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시
    - 두번째 봉우리는 service-1의 요청으로 인한 트래픽을 측정한 것이다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_5.jpg?raw=true">

- BytesOutToDestination
    - NAT 게이트웨이를 통해 대상으로 전송된 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시
    - 두번째 봉우리는 service-1의 요청으로 인한 트래픽을 측정한 것이다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_6.jpg?raw=true">

- BytesOutToSource
    - NAT 게이트웨이를 통해 VPC 내 클라이언트로 전송된 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시
    - 두번째 봉우리는 service-1의 요청으로 service-2의 ISO 파일을 전송하는 트래픽을 측정한 것이다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_7.jpg?raw=true">

2. service-1

- NetworkIn
    - service-1으로 들어온 트래픽을 측정

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_8.jpg?raw=true">

- NetworkOut
    - service-1에서 나간 트래픽을 측정

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_9.jpg?raw=true">

3. service-2

- NetworkIn
    - service-2로 들어온 트래픽을 측정
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_10.jpg?raw=true">

- NetworkOut
    - service-2에서 나간 트래픽을 측정
    service-1의 요청으로 service-2의 ISO 파일을 전송하는 트래픽을 나타났다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_11.jpg?raw=true">


## 테스트 후 비용 확인

1.  트래픽 비용
<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_12.jpg?raw=true">

	- 첫번째 항목 : 인터넷 인바운드 트래픽
    	- ISO 파일 다운로드를 위해 사용
	- 두번째 항목 : 인터넷 아웃바운드 트래픽
    	- 실질적인 사용량이 없는 상태
	- 3~4번째 항목 : 같은 리전 내의 다른 AZ 간의 트래픽
    	- 4.4GB ISO 파일을 3번 다운로드 받았고, service-2 아웃바운드 트래픽과 service-1 인바운드 트래픽이 3,4번 항목에 합쳐진 것을 알 수 있다 (4.4GB * 6)

2.  NAT Gateway 비용
<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_13.jpg?raw=true">

- service-2의 아웃바운드 트래픽이 NAT 게이트웨이를 통과했음을 알 수 있다.
- NAT 게이트웨이를 통한 트래픽만 계산하므로 service-2 → service-1 방향으로의 4.4GB 3번과 ISO 파일을 인터넷으로부터 다운받기 위한 4.4GB 1번이 포함되었다.

## 결론

- 분명 service-1과 service-2는 같은 서브넷 (같은 AZ)에 위치해 있었음에도 다른 AZ 간 트래픽으로 인식되었다. 이는 ALB의 Public IP가 EIP와 동일한 효과를 받았기 때문이다. 
- 이는 다음의 문서에서 확인할 수 있다.
    - 동일 AWS 리전 내 데이터 전송 (https://aws.amazon.com/ko/ec2/pricing/on-demand/)
```
IPv4: 퍼블릭 또는 탄력적 IPv4 주소에서 ‘송신’ 및 ‘수신’된 
데이터는 각 방향에 대해 GB당 0.01 USD가 부과됩니다.
```
```
동일 AWS VPC에서 EC2 인스턴스와 로드 밸런서 간에 프라이빗 IP 주소를 사용하여
Amazon Classic 및 Application Elastic Load Balancer에서 
‘송신’ 및 ‘수신’되는 데이터는 무료입니다.
```


- 앞서 이야기한 ALB의 Public IP는 또다른 문제를 불러오는데 바로 NAT Gateway 비용을 발생시키는 것이다. Private 서브넷에서 아웃바운드로의 인터넷 접속을 허용하기 위해 라우팅 테이블에 Destination 주소가 VPC 내부 IP가 아닌 경우 무조건 NAT Gateway를 거치게 되어 있기 때문이다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_29.jpg?raw=true">


# 참고자료
- Amazon VPC 네트워크 원리와 보안 (차정도 / 에이콘)