---
title:  "[PowerDNS] DNS 이론 3 - DNS 동작 방식"
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

## DNS 동작 방식

![도메인 주소 구조](https://raw.githubusercontent.com/susoterran/susoterran.github.io/master/assets/img/dns/domain-query.jpg){: .center}


<p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;① 클라이언트(리졸버)가 www.naver.com 에 대해 DNS 조회를 진행
</p><p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;② 캐시 네임 서버에 www.naver.com에 대한 값이 있다면 8번으로 진행. 없다면 루트 네임 서버에 www.naver.com 에 대한 응답을 요청
</p><p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;③ 루트 네임 서버는 com 네임 서버의 IP 주소를 캐시 네임 서버에 전달
</p><p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;④ 캐시 네임 서버는 com 네임 서버에 www.naver.com에 대한 응답을 요청
</p><p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;⑤ com 네임 서버는 naver 네임 서버의 IP 주소를 Cache 네임 서버에 전달
</p><p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;⑥ 캐시 네임 서버는 naver 네임 서버에 www.naver.com에 대한 응답을 요청
</p><p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;⑦ naver 네임 서버는 www.naver.com의 IP 주소를 캐시 네임 서버에 전달
</p><p style="font-size: 0.7em">
&nbsp;&nbsp;&nbsp;&nbsp;⑧ 캐시 네임 서버는 www.naver.com의 IP 주소를 클라이언트(리졸버)에게 전달
</p>

사용자가 크롬 브라우저로 www.naver.com 주소를 입력하고 Enter를 누르면 크롬 브라우저는 Windows OS 내부에 존재하는 리졸버 (리졸버는 장비 자체가 될 수 있고, DNS 질의를 실질적으로 요청하는 프로그램일 수 있다) 에게 해당 www.naver.com 주소에 대한 IP 주소를 요청한다.

리졸버는 Windows 상에 등록된 네임 서버에 DNS 질의를 요청하게 된다. 여기서의 네임 서버는 보통 ISP에 운영하는 캐시 네임 서버를 의미한다. 대표적으로는 KT에서 운영하는 168.126.63.1과 구글에서 운영하는 8.8.8.8 등이 있다.

캐시 네임 서버는 클라이언트들의 요청을 응답해주기도 하지만 응답된 값들을 캐시(cache)에 임시로 저장하기도 한다. 동일한 요청이 들어오면 위와 같이 다른 네임 서버들 사이를 왔다갔다 하는 과정 없이 캐시에 저장된 값으로 응답한다.

전자와 같이 다른 네임 서버들 사이를 왔다갔다하면서 최종적으로 응답받은 값을 전달하는 과정을 'Recursive Query'라고 하며 (1~8번 까지의 과정), Recursive Query 이후 전달되는 응답을 '권한 없는 응답'이라고 한다.

후자와 같이 캐시 네임 서버가 가지고 있는 값을 바로 전달하는 과정을 'Iterative Query' 라고 하며(1,8번 과정), Iterative Query 이후 전달되는 응답을 '권한 있는 응답'이라고 한다.

여기서 리졸버의 개념이 굉장히 모호하고 왔다갔다하는 것을 볼 수 있다. DNS 조회를 요청하는 장비나 프로그램은 모두 리졸버라고 말하기 때문인데, 어렵게 생각할 거 없이 네임 서버와 반대되는 클라이언트를 리졸버라고 생각하면 된다. DNS 상의 클라이언트 프로그램도 리졸버라고 부르기는 하지만, 서버-클라이언트 관계에서 서버를 네임 서버 클라이언트를 리졸버라고 편의상 생각하는게 정신 건강에 도움이 된다. 


참고 자료
- KISA '고급 DNS 관리 실무' 교육
- '성공과 실패를 결정하는 1%의 네트워크 원리' (성안당)