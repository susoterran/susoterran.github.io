---
title:  "[AWS] 아리송한 VPC 내부 트래픽 비용 산정 방식 3"
excerpt: "내부 ALB를 통한 내부 트래픽 비용은 어떻게 산출할까?"
header:
  overlay_color: "#333"
categories:
  - other
tags:
  - AWS
last_modified_at : 2022-11-21
last_modified_at_2 : 2022-11-21
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# 내부 ALB를 통한 내부 트래픽 비용은 어떻게 산출할까?

## 테스트 방법

- 첫번째 테스트와 다른 점은 ALB를 '인터넷 경계'가 아닌 '내부'에 생성한다는 것이다.
- service-1, service-2 인스턴스를 생성한다. 이들 인스턴스는 각각 Private subnet 1과 Private subnet 2에 위치한다.


## 테스트 환경

- service-1 : 10.0.1.97
- service-2 : 10.0.1.234
- ALB
	- internal-jslim-test-1979261898.ap-northeast-2.elb.amazonaws.com
	- 10.0.1.76
- 테스트 전 비용 및 트래픽 사용량

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_14.jpg?raw=true">

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_15.jpg?raw=true">


## 테스트 진행
- service-1 에서 service-2에 있는 ISO 파일을 wget을 이용하여 다운로드한다. 이때 접속 URL은 ALB의 URL이다.
- 4.4GB 파일을 3번 다운로드 받음

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_16.jpg?raw=true">


### 테스트 후 지표 확인

1. NAT Gateway

- BytesInFromDestination
    - NAT 게이트웨이가 대상으로부터 수신한 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_17.jpg?raw=true">

- BytesInFromSource
    - NAT 게이트웨이가 VPC 내 클라이언트로부터 수신한 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_18.jpg?raw=true">

- BytesOutToDestination
    - NAT 게이트웨이를 통해 대상으로 전송된 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_19.jpg?raw=true">

- BytesOutToSource
    - NAT 게이트웨이를 통해 VPC 내 클라이언트로 전송된 바이트 수
    - 첫번째 봉우리는 외부로부터 service-2에 ISO 파일을 받기 위한 것이므로 무시

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_20.jpg?raw=true">


2. service-1

- NetworkIn
    - service-1으로 들어온 트래픽을 측정

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_21.jpg?raw=true">

- NetworkOut
    - service-1에서 나간 트래픽을 측정

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_22.jpg?raw=true">

3. service-2

- NetworkIn
    - service-2으로 들어온 트래픽을 측정

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_23.jpg?raw=true">

- NetworkOut
    - service-2에서 나간 트래픽을 측정

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_24.jpg?raw=true">


## 테스트 후 비용 확인

1. 트래픽 비용

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_25.jpg?raw=true">


2. NAT Gateway 비용
<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_26.jpg?raw=true">

# 결론

- 인터넷 아웃바운드로의 트래픽 사용량의 변화는 없다.
- 리전 간 트래픽에도 변화가 없다.
- NAT Gateway를 통한 테스트는 인터넷으로부터 ISO 파일을 다운받기 위해 4.4GB 정도 트래픽을 사용한 것이 포함된 거 외에는 사용량의 변화가 없다.

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_27.jpg?raw=true">


# 참고자료
- Amazon VPC 네트워크 원리와 보안 (차정도 / 에이콘)