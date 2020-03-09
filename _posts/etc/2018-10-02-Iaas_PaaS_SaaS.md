---
title: "IaaS, PaaS, SaaS"
categories: "etc"
tags:
  - IaaS
  - PaaS
  - SaaS
  - Cloud Service
---

## IaaS & PaaS & SaaS

*"Google Compute Engine 과 Google App Engine의 차이가 무엇인가요?"*

질문으로 시작됐다.

현재 우리 서비스는 GCP위에서 구동되기 때문에 그 중 Compute Engine과 App Engine이 코어서비스이다.

그러자 사수분께서 이렇게 말씀하셨다.

"IaaS, PaaS, SaaS 에 대해서 들어보신 적 있으신가요?"

"아니요..."

처음 들어보는 개념이었다. 그리고 곧장 구글에 위 키워드를 검색하자 꽤 많은 결과를 확인할 수 있었고.

요즘 대세로 자리잡은 Cloud Platform 하에서 가장 기본적이고, 중요한 개념이었다.

우리는 우리가 사용한것들을 '왜'사용해야 하고, '무엇을' 얻을 수 있는지 명확하게 이해해야 한다.

### 클라우드 서비스는 왜 이용할까?

간단하다. 대규모 서비스를 지원할 경우 운영서버 또는 DB서버 등 물리적으로 많은 서버자원 또는 가변적인 자원이 요구된다. 하지만 많은 자원을 관리하는데는 그만한 비용과 비효율이 발생할 수 있기 때문에 이를 대신 관리해주는 컴퓨팅 서비스에 일정한 대가를 지불하고 온전히 본인들의 서비스 분야에만 집중하기 위해서이다.

그러면 명확해졌다. 클라우드 컴퓨팅은 우리가 비효율적인 시간에 낭비하는 것을 방지해주고, 보다 간편한 환경구성과 필요에 따라 정확히 필요한 서비스를 효율적으로 사용할 수 있다.

그러면 이 클라우드 환경이 우리에게 무엇을 어떻게 제공해주는지 이해해야한다.

### 필요한 만큼, 원하는 만큼 컴퓨팅 인프라를 쓰자 ; IaaS(Infrastructure as a Services)

서버사이드 인프라를 구축하기 위해 필요한 자원인 네트워크, 스토리지, 데이터베이스 등을 제공한다.

사용자는 이 모든 환경을 커스터마이징 함으로써, 자신의 스펙에 맞는 서버 환경을 설정하고 로우레벨부터 제어할 수 있다.

### 기호에 맞춰 SW 개발 돕는, 개발자를 위한 서비스 ; PaaS(Platform as a Services)

쉽게 말해, 웹 개발자가 배포(deploy)만하면 되도록 안정적인 환경을 제공해준다.  응용프로그램을 손쉽게 개발할 수 있도록 API까지 제공해준다.

### 필요한 소프트웨어, 설치 없이 웹에서 뚝딱 ; SaaS(Software as a Services)

클라우드 환경에서 운영되는 최종 애플리케이션 서비스이다. 다시 말해, 최종 생산물인셈이다.

위 내용을 그림으로 정리하면 다음과 같다.

![IaaS_PaaS_SaaS](/assets/images/study/dev/2018/10_iaas-paas-saas-comparison-1024x759.jpg)

사실 위 그림을 참고하는 것이 가장 쉽게 참고할 수 있다.

전혀 어려운 개념이아니다.

마지막으로 각각의 서비스 별로 이해하기 좋게 분류한 표가 있다.

![useCase](/assets/images/study/dev/2018/10_usecase.png)

출처 : https://www.bmc.com/blogs/saas-vs-paas-vs-iaas-whats-the-difference-and-how-to-choose/
