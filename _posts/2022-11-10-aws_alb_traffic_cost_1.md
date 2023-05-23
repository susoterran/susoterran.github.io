---
title:  "[AWS] 아리송한 VPC 내부 트래픽 비용 산정 방식 1"
excerpt: "ALB의 위치에 따라 달라지는 트래픽 비용 산정 방식"
header:
  overlay_color: "#333"
categories:
  - other
tags:
  - AWS
last_modified_at : 2022-11-19
last_modified_at_2 : 2022-11-19
toc: true
toc_label: "목차"
toc_sticky: true
classes: wide
share: false
layout: single
comments: true
---

# AWS에서의 트래픽 비용 산정

AWS에서의 트래픽 비용 산정은 ELB, CloudFront와 같이 자체적인 방식이 있지 않는 한 대부분 'EC2 온디맨드 요금'의 '데이터 전송'과 '동일 AWS 리전 내 테이터 전송'의 항목을 따른다.

클라우드 서비스에서의 트래픽 비용은 IDC 환경과 비교했을 때 많이 비싼 편인데, 그 중 AWS는 트래픽 비용이 비싼 편에 속한다.

그러다보니 서비스 확장에 따른 트래픽 비용 상승은 클라우트 엔지니어들을 힘들게 하는 원인 중에 하나가 된다.

트래픽 비용을 부과하는 방식은 대부분의 클라우드 서비스들이 비슷하다.

- 인터넷에서 VPC 내부로 들어오는 인바운드 트래픽에 대해서 과금하지 않는다.
- VPC 내부에서 인터넷으로 나가는 아웃바운드 트래픽에 대해서 과금한다.
- 같은 리전 내의 같은 가용 영역에서 전송되는 트래픽에 대해서는 과금하지 않는다.
- 같은 리전 내의 다른 가용 영역에서 전송되는 트래픽에 대해서 과금한다 (이때 송신, 수신에 대해 모두 과금함)
- 다른 리전 간 전송되는 트래픽에 대해서 과금한다

여기서 가장 머리 아프게 하는 부분은 4번째 항목이다. VPC 내부에서 돌아다니는 트래픽임에도 다른 가용 영역에서 송수신된 트래픽에 대해서도 과금을 한다는 것이다.

의아하게 생각할 수도 있지만 온프레미스 환경과 클라우드 환경에서의 구조적인 차이를 생각하면 이해하기 쉬워지는데, 바로 가용성이다.

온프레미스 환경이라면 보통 한 곳의 IDC와 랙/장비 대여 및 대역폭 계약 (Co-location)을 하게 된다. 

만약 이 IDC에 물리적인 사고로 랙에 있는 장비들을 운영할 수 없는 상태가 된다면 어떻게 될까? DR 구성이 되어 있지 않다면 서비스는 중단될 것이다. 

만약 이러한 DR 환경이 갖추어져 있다면 각 IDC 사이에서 발생하는 트래픽 비용은 어떻게 산정이 될까? 같은 회사가 운영하는 IDC라면 어느 정도 감면 혜택을 받을 수 있을테지만, 그렇지 않다면 내부 트래픽이 아닌 외부 트래픽으로 간주될 것이다.

다시 클라우드 환경, 특히 AWS 환경을 생각한다면 VPC라는 가상의 네트워크로 묶여있긴 하지만 각각의 가용 영역은 물리적으로 분리된 IDC를 의미하게 된다. 따라서 위에서 설명한 바와 같이 외부 네트워크가 아닌 내부 네트워크 간 트래픽으로 보이지만 엄연히 인터넷으로 전송되는 트래픽이기 때문에 어느 정도 감면을 받고 비용을 산정하게 된다.

<br>
<br>

# ALB의 위치에 따라 트래픽 흐름은 어떻게 바뀌게 될까?

결국 VPC 내부 트래픽이 어디로 어떻게 흘러 가는지 모른다면, 괜한 트래픽 비용만 나가게 된다.

이번에 알아보고자 하는 것은 'ALB의 위치에 따라 트래픽 흐름이 어떻게 바뀌는가' 이다.

ALB(NLB도 마찬가지)의 생성 위치는 '인터넷 경계'와 '내부'로 나뉘는데, 로드 밸런싱을 VPC 외부에서 들어오는 트래픽에 대해 적용할지, VPC 내부의 트래픽에 대해 적용할지를 구분하기 위함이다.

이때 몇 가지 차이점이 있다.

- '인터넷 경계'에 생성하고자 한다면 Internet Gateway와 연결되어 있는 Public 서브넷에 위치해야 한다.
- '인터넷 경계'에 생성하게 되면 ALB에 접속하기 위한 IP는 Public IP로 할당 된다.
- '내부'에 생성하게 되면 ALB에 접속하기 위한 IP는 매핑된 서브넷의 Private IP로 할당된다.

결국 ALB의 IP 주소가 Public이냐 Private이냐의 차이점이 크게 작용하는데 여기서 의문이 든다.

만약 내부 인스턴스끼리 통신을 할 때 ALB를 사용하게 된다고 가정할 때

- '내부' ALB를 통해 통신을 한다면 같은 리전 내의 트래픽으로 인지하여 비용이 산출될 것이다.
- 그렇다면 '인터넷 경계' ALB를 통해 통신을 한다면, 같은 리전 내 트래픽으로 인지할까? 아니면 인터넷 아웃바운드 트래픽으로 인지할까?

ALB가 가지고 있는 Public IP는 실제 VPC에서 사용하는 IP 대역이 아니다. 따라서 VPC 외부로 나갔다가 라우팅을 거쳐 다시 VPC로 되돌아오게 되면서 아웃바운드 트래픽으로 계산되지 않을까?

<img src="https://github.com/susoterran/susoterran.github.io/blob/master/assets/img/2022-11-10-aws_alb_traffic_cost/alb_traffic_1.jpg?raw=true">