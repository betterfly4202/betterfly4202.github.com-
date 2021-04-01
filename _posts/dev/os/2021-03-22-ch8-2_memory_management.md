---
title: "[Operating System] Ch8-2. Memory Management"
categories: "OS"
tags:
  - Operating System
  - Memory Management
---

# Memory Management
## Allocation of Physical Memory
- 메모리는 일반적으로 두 영역으로 나뉘어 사용
  - OS 상주 영역
    - interrupt vector와 함께 낮은 주소 영역 사용
  - 사용자 프로세스 영역
    - 높은 주소 영역 사용

- 사용자 프로세스 영역의 할당 방법
  - Contiguous allocation
    - 각각의 프로세스가 메모리의 연속적인 공간에 적재되도록 하는 것  
    앞서 다루었던 `limit register` & `relocation register`를 이용하여 메모리를 할당받는 방식으로, 1차원적으로 앞서 사용되던 메모리 공간을 이어서 연속적으로 공간을 할당받음
      - Fixed partition allocation
      - Variable partition allocation
    - Noncontiguous allocation
      - 하나의 프로세스가 메모리의 여러 영역에 분산되어 올라갈 수 있음  
      근래의 프로그램에서 사용하는 방법으로, 메모리의 효율을 위하여 아래의 방법을 통해서 메모리를 분산하여 저장 후 참조하는 방법
        - Paging
        - Segmentation
        - Paged Segmentation
  
### Contiguous allocation
#### 고정 분할 방식
![](/assets/images/study/dev/2021/os/ch8_fixed-partition.png)
그림처럼 고정된 메모리 공간에 `프로그램A`, `프로그램B`가 개입되는 상황이다.
- <분할 1> 공간에 `프로그램A`가 할당된다.
- <분할 2> 공간의 크기보다 `프로그램B`의 필요한 메모리 공간이 크기 때문에 <분할 2> 공간은 사용할 수 없다.
  - 할당된 메모리보다 **프로그램의 사이즈가 커서** 메모리를 가용할 수 없는 경우 => **외부 조각**
- <분할 3> 공간에 `프로그램B`가 할당된다.
  - 할당된 메모리의 공간이 **프로그램의 사이즈보다 커서** 해당 영역안에 메모리 공간이 남는 경우 => **내부 조각**

#### 고정 분할 방식 특징
- 물리적 메모리를 몇 개의 영구적 분할(partition)로 나눔
- 분할의 크기가 모두 동일한 방식과 서로 다른 방식이 존재
- 분할당 하나의 프로그램 적재
- 융통성이 없음
  - 동시에 메모리에 load되는 프로그램의 수가 고정됨
  - 최대 수행 가능 프로그램 크기 제한
- 내부 조각<sub>Internal fragmentation</sub>발생 (외부 조각<sub>external fragmentation</sub>도 발생)

#### 가변 분할 방식
![](/assets/images/study/dev/2021/os/ch8_variable-partition.png)
`프로그램A`, `프로그램B`, `프로그램C`가 수행중인 메모리가 있다.
- `프로그램B`가 종료 후 `프로그램D`가 수행된다. 이 때 `프로그램B`가 사용한 공간은 메모리가 작아서 여유있는 공간을 찾아서 메모리를 할당한다. 

#### 가변 분할 방식 특징
- 프로그램의 크기를 고려해서 할당
- 분할의 크기, 개수가 동적으로 변함
- 기술적 관리 기법이 필요함
- 외부 조각<sub>external fragmentation</sub> 발생


#### Hole
- 가용 메모리 공간
- 다양한 크기의 hole들이 메모리 여러 곳에 흩어져 있음
- 프로세스가 도착하면 수용가능한 hole을 할당
- 운영체제는 다음의 정보를 유지
  - 할당 공간
  - 가용 공간(hole)

![](/assets/images/study/dev/2021/os/ch8_hole.png)


#### Dynamic Storage-Allocation Problem
가변 분할 방식에서 size **n**인 요청을 만족하는 가장 적잘한 hole을 찾는 문제

- First-fit
  - Size가 n 이상인 것 중 최초로 찾아지는 hole에 할당
- Best-fit
  - Size가 n 이상인 가장 작은 hole을 찾아서 할당
  - Hole들의 리스트가 크기순으로 정렬되지 않은 경우 모든 hole의 리스트를 탐색해야 함
  - 많은 수의 아주 작은 hole들이 생성됨
- Worst-fit
  - 가장 큰 hole에 할당
  - 모든 리스트를 탐색
  - 상대적으로 아주 큰 hole들이 생성됨

> First-fit과 best-fit이 worst-fit보다 속도와 공간 이용률 측면에서 효과적인것으로 알려짐


#### Compaction
외부 조각<sub>External fragmentation</sub>문제를 해결하는 한 가지 방법
- 사용 중인 메모리 영역을 한군데로 몰고 hole들을 다른 한 곳으로 몰아, 큰 block을 만드는 것
- 매우 큰 비용이 발생함
- 최소한의 메모리 이동으로 compaction하는 방법은 매우 복잡한 문제
- Compaction은 프로세스의 주소가 실행 시간에 동적으로 재배치 가능한 경우에만 수행될 수 있음

### Noncontiguous allocation
하나의 프로세스가 메모리의 여러 영역에 분산되어 올라갈 수 있음

- Paging
  - 프로세스의 주소 공간을 **같은 크기로 분할**하여(페이징), 각 페이징별로 메모리 상에 올려놓음
  - 물리적인 메모리 공간 역시 페이징 크기로 분할하여 놓음 => Page Frame
  - 분할하여 작은단위로 메모리를 사용하기 때문에 앞서 `Contiguous allocation`에서 다루었던 **Hole**또는 **Dynamic Storage Allocation** 문제 등을 회피할 수 있다.
- 하지만 MMU를 통한 주소 변환시, 각 페이지별로 흩어진 메모리 공간의 address binding이 복잡해지는 문제가 있다.

- Segmentation
  - 프로그램의 주소 공간을 **의미 있는 단위**로 분할하는 것
    - 예를들어 `code`, `data`, `stack` 각 segment 별로 쪼개어서 각각 물리적으로 다른 위치에 할당한다.
  - 의미별로 분할하기 때문에 각 segment의 크기가 균일하지 않다.
    - 때문에 **Dynamic Storage-Allocation** 문제가 발생할 수 있다.

---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books