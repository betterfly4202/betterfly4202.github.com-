---
title: "RAID 1+0과 RAID 0+1"
categories: "OS"
tags:
  - RAID
  - Disk
---

최근 사내에 MongoDB 서버를 새로 구성하면서 서버를 어떻게 구성할건지에 대한 논의가 있었다.  
그 중 분산 시스템을 다루는 환경에서 중요한 부분인, **'디스크를 어떻게 구성할 것인지'** 에 대한 뜨거운 논의가 있었다.

> **RAID 1+0(Mirror + Stripe)로 구성하는게 어떨까요?**

이게 어떤 의미일까?

# RAID와 RAID 1+0

**RAID<sub>Redundant Array of Independent Disks</sub>**  
직역하면 "독립적인 디스크<sub>independent disks</sub>를 여분의 배열<sub>redundant array</sub>로 구성하는 것"을 말한다.  
다시 말해, '독립적인 디스크를 연결하여 중복으로 관리한다'는 의미일 것이다.

왜 그렇게할까?

바로 이전 포스팅에 이 내용을 기록해 두었는데, 그냥 이런게 있구나 라고만 적어놓고 까맣게 잊어 먹고 있어서 반성을 하며 다시 공부했다.

[[Operating System] Ch11. Disk Management and Scheduling](https://betterfly4202.github.io/os/2021/04/15/ch11_dist_management_and_scheduling/)

RAID를 사용하는 이유는 다름아닌 데이터를 분산/중복 저장 함으로써 

- 분산 처리를 통한 성능 향상
- 데이터의 복제를 통한 안정성 향상

위 두가지의 이유를 들 수 있다.  
RAID는 이해됐다. 그러면 `RAID 1+0` 이건 뭘까?

# RAID의 특징
## RAID 0

- Stripe/Striping
- 두 개 이상의 디스크를 병렬로 연결하고 각 디스크에 분산 저장하여 하나의 디스크처럼 이용하는 기술

![](/assets/images/study/dev/2021/os/0801_raid0.png)

이렇게 물리적으로 분리되어 있는 디스크에 "A","B","C","D"."E" 이렇게 5개의 데이터가 각각 저장된다고 가정해보자.

- A → A1 (Disk0)
- B → A2 (Disk1)
- C → A3 (Disk0)
- D → A4 (Disk1)
- E → A6 (Disk0)

이렇게 두개의 디스크에 분산되어 저장될 것이다. (실제로 이렇게 정확히 순서대로 구분되진 않을 것이다.)  
그러면 이렇게 데이터를 쪼개서 저장하는하는 이유는 뭘까?  
앞서 RAID의 특징 중 하나인, **분산 처리를 통한 성능 향상**이 그 이유이다.

`Disk I/O`는 하드웨어 기술 중 상당히 처리속도가 느리다. (memory, cpu에 비하여) 때문에 `Disk I/O`를 최대한 효율적으로 처리하는것은 꽤 의미있는 작업이 될 것이다.

## RAID 1

- Mirror/Mirroring
- 두 개 이상의 디스크를 병렬로 연결해서 데이터를 복제하는 기술

![](/assets/images/study/dev/2021/os/0801_raid1.png)

앞서 `RAID 0` 은  각각의 디스크에 서로 다른 데이터를 분산하여 저장했다면, `RAID 1` 기술은 같은 데이터를 각각의 디스크에 복제하는 기술을 말한다.  
이는 RAID의 특징 중 **데이터의 신뢰성(복제 기능)**을 택한 것이다.  

비록 디스크 공간을 효율적으로 활용하지 못하고, 성능 향상에는 한계가 존재하지만,  
당연히 하나의 디스크에 문제가 발생했을때 다른 디스크를 통해 언제든지 복원이 가능하므로, 데이터의 안정성이 요구되는 환경에 사용된다.

---

자 그럼 이제 이 두개의 RAID를 혼합하는 응용편을 알아보자.

## RAID 0+1(RAID 01)

먼저 `RAIA 0+1(RAID 01)` 방법인데, 앞에 나온 숫자가 앞서 살펴본 `RAID 0` 방식위에 `RAID 1`로 구성한 방식이다.

- 각 RAID를 Striping<sub>RAID 0</sub> 후 Mirroring<sub>RAID 1</sub>하는 방식

![](/assets/images/study/dev/2021/os/0801_raid01.png)

## RAID 1+0 (RAID 10)

- 각 RAID를 Mirroring<sub>RAID 1</sub> 후에 Striping<sub>RAID 0</sub>하는 방식
- 특정 디스크 장애가 발생한 경우, 데이터의 무결성에 영향을 주지 않으며, 미러링 된 다른 디스크를 사용하며 문제가 있는 디스크만 교체하면 된다.(뒤에 보강 설명)

![](/assets/images/study/dev/2021/os/0801_raid10.png)

# RAID 0+1 vs RAID 1+0

얼핏보면 조삼모사같은데 이 두가지 방식은 어떤 점이 다른걸까?

## 공통점
- 분산 저장(Striping)을 통한 속도 향상과 
- 복제(Mirroring)를 신뢰성을 만족

## 차이점

### 안정성

![](/assets/images/study/dev/2021/os/0801_raid_difference.png)

그림 좌측의 `RAID 0+1`의 경우 각 RAID에 첫번째 디스크(Disk 0, Disk 2)의 문제가 발생할 경우 디스크의 유실 또는 복원이 어려운 문제가 발생한다.  
하지만 우측의 `RAID 1+0`은 각 RAID 디스크의 한 쪽씩 문제가 발생해도 문제가 발생하지 않는다.

**하지만**, `RAID 1+0`의 경우 하나의 RAID전체가 망가지는 경우 복원이 불가능한 문제가 있지만,  
일반적으로 RAID 자체에 문제가 발생할 확률이 더 낮기 때문에 안정성이 높다고 한다.

내 생각에 아마도 RAID마다 **Master Disk**와 같이 *write*를 담당하는 디스크와 그 데이터를 복제하는 **Slave Disk**로 구분하여 사용한다면, 상대적으로 I/O작업이 많은 Master 디스크에 부담을 많이 주기 때문에 `RAID 1+0`의 경우엔 RAID 전체 디스크에 문제가 일어날 확률이 적은게 아닐까 싶다.

`RAID 0+1`은 각 디스크마다 write/read를 모두 해야하기 때문에 모든 디스크에 부담이 가는 것이 아닌가 싶다.

### 문제 복원 방법

`RAID 0+1` 에서 **Disk 0**에서 문제가 발생한 경우, 데이터를 복원할 때 미러링된 다른 RAID 전체를 복제 한 후 다시 Striping해야 한다.  
반면, `RAID 1+0`의 경우 **Disk 0**이 깨진 경우, 깨진 데이터를 복원 후 같은 RAID의 다른 디스크(Disk 1)에서 복제하면 되기 때문에 `Disk I/O`비용이 매우 저렴하다.

---

*Reference*  
- [https://hellowoori.tistory.com/53](https://hellowoori.tistory.com/53)
- [https://soft.plusblog.co.kr/125](https://soft.plusblog.co.kr/125)