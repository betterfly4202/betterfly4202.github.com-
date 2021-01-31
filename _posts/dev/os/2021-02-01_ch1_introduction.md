---
title: "[Operating System] Ch1. 운영체제의 기본"
categories: "OS"
tags:
  - Operating System
  - System Structure
  - Program Execution
---

# 운영체제란 무엇인가?

## 운영체제의 정의
![](/assets/images/study/dev/2021/os/ch1_os_layer.png)

그림처럼 하드웨어와 소프트웨어 중간에 위치하여, 모든 소프투웨어와 하드웨어으를 연결해주는 **소프트웨어 계층**을 말한다.  
좁은의미로 운영체제는 운영체제의 핵심으로 메모리에 상주하고 있는 커널<sup>kernel</sup>을 말하며,  
넓은의미의 운영체제는 커널뿐만 아니라, 메모리에 포함되지 않는 각종 주변 시스템 유틸리티를 포함한다.

다시 정리하면, 운영체제는 **하드웨어 자원을 효율적으로 관리하는 목적**으로 설계된 소프트웨어를 말한다.

## 운영체제의 분류

운영체제는 다음의 3가지로 분류될 수 있다.

- 동시 작업 가능 여부
- 사용자의 수
- 처리 방식

### 동시작업 가능 여부
- Single tasking : MS-DOS와 같이 한 명령의 수행을 종료 후 다른 명령을 실행하는 것을 말한다.  
엘레베이터, TV와 같은 단순한 단일 명령만 수행하는 경우를 말한다.  
- Multi tasking : UNIX, Windows와 같은 소프트웨어하에서 동시에 다양한 명령을 실행하는 것을 말한다. (실제로 **동시**보다는 아주 빠른속도로 `context switching`이 일어난다)

### 사용자의 수
컴퓨터 한 대를 동시에 접근 가능한 경우
- Single user: MS-DOS, Windows 등
- Multi user : UNIX, NT Server(하나의 서버 장비에 여러 계정을 생성하여 동시에 접근이 가능)

### 처리 방식
- 일괄처리(batch processing) : 작업 요청의 일정량을 모아서 처리하는 방식을 말한다. (작업이 완료 종료시점까지 대기해야 함)
- 시분할(time sharing) : 여러 작업을 수행시 컴퓨터 처리 능력을 일정 시간 단위로 분할<sup>context switching</sup>하여 사용 -> 위에 설명한 `multi tasking` 해당, 상호간의 interactive
- 실시간 (realtime OS) : 정해진 시간안에 Job이 반드시 종료되는 것이 보장하는 시스템


---

*Reference*

- Operating System Concept
- http://www.kocw.net/home/search/kemView.do?kemId=1046323