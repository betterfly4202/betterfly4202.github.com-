# 11번가 Spring Cloud 기반 MSA로의 전환_지난 1년간의 이야기...

## 기존의 11번가는..
- 초대형 거대 monolithic System
- 8년 넘게 사용해 온 낙후도니 S/W Stack
- 너무나 커진 200만 라인의 공통 모듈

--- 
이로 인해 많은 개발팀의 코드를 한번에 배포
    - 매주 목요일 밤샘 정기 배포, 잦은 장애
- 개발자들이 새로운 기술을 접해 볼 기회 원천 봉쇄
    - Java 1.6, Spring 2.x or 외주 개발 Framework
- IDE 띄우는 것 조차 버거운 개발 환경
    - 200만 라인의 공통 코드인해 항상 메모리 부족

개발팀 별로 자유로운 개발 환경 구출 필요함

어떻게 분리할 것인가?

Project Vine
> 11번가 점진적 MSA 전환 Project

- Strangler Application Pattern
- Legacy 교살 전략
    - 새로운 Application 은 독립된 API 작성
    

Hybrid MSA in 11st
- 업무 도메인 별로 서버 분리
- Legact 코드에서는 새로운 api 서버들을 호출
- 기존 코드와 새로운 API호출은 DB flag통해 Switchable 하게..

### MSA 플랫폼 Solutions 선정
- MSA 위한 Best Practice 를 빠르게 적용하는 것이 필요
    - Netflix oss open source
    - Spring boot
    - Spring cloud

Deep Dive into Hystrix, Ribbon, Eureka
Hystrix > Fault Tolerance Library
    - 장애 전파 방지 & Resilience
    
기능정 관점에서 본 4가지 기능
    - Circuit breaker
    - fallbkack
    - thread isolation
    - timeout

써킷 브레이커
 1. 일정 시간도안 2. 일정 개수 이상의 호출이 발생한 경우 3. 일정 비율이상의 에러가 발생하면 > 써킷 오픈(호출 차단)
 4. 일정 시간 경과 후 단 한개으 ㅣ요청에 대해 호출을 허용하고(하프 오픈) 이 호출이 성공하면 > 써킷 클로즈(호출 허용)
 
 >> 10초간 20개 이상의 호출이 발생한 경우 50%이상의 에러가 발생하면 5초간 써킷 오픈


폴백
- ㅅ펄백으로 지정된 메소드는 다음의 경우에 원본 메소드 대신 실행된다


리본을 사용하는 이유
기존에 로드밸런스 장비가 있는데 왜 써야하는가
개발친화적임

IRule - 주어진 서버 목록에서 어떤 서버를 선택할 것인가
IPing - 각 서버가 살아있는가를 검사하여 서버를 사용할지를 검증함
등 7가지 이유..많음



