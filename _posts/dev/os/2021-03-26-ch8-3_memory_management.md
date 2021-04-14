---
title: "[Operating System] Ch8-2. Memory Management - Noncontiguous allocation"
categories: "OS"
tags:
  - Operating System
  - Memory Management
---

# Memory Management
## Noncontiguous allocation
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

### Paging
![](/assets/images/study/dev/2021/os/ch8_paging.png)
- Process의 virtual memory를 동일한 사이즈의 page단위로 나눔
- virtual memory의 내용이 page단위로 *noncontiguous*하게 저장됨
- 일부는 backing storage에, 일부는 physical memory에 저장
- **Basic Method**
  - physical memory를 동일한 크기의 frame으로 나눔
  - logical memory를 동일한 크기의 page로 나눔(frame과 같은 크기)
  - 모든 가용 frame들을 관리
  - page table을 사용하여 logical address를 physical address로 변환
  - external fragmentation 발생 안함
  - internal fragmentation 발생 가능
페이징처리는 다음과 같은 아키텍처의 흐름으로 논리적 주소공간<sub>logical address</sub>이 `page table`을 통해서 물리적 주소공간<sub>physical address</sub>으로 변환된다.  

![](/assets/images/study/dev/2021/os/ch8_address-translation-architecture.jpeg)

#### Implementation of Page table
- Page table은 main memory에 상주
- Page-table base register(PTBR)가 *page table*을 가리킴
- Page-table length register(PTLR)가 테이블의 크기를 보관
- 모든 메모리 접근 연산에는 2번의 memory access가 필요
  - page table접근 
  - 실제 data/instruction 접근 
- 속도 향상을 위해 associative regsiter 혹은 translation look-aside buffer(TLB)라 불리는 고속의 *loopup hardware cache* 사용
  - 주소 변환을 위한 캐시메모리
  - page table에서 빈번히 참조되는 일부 엔트리를 캐싱

![](/assets/images/study/dev/2021/os/ch8_paging-hardware-tlb.jpeg)
<span style="color:#c2c9d4; font-size: 10px;"> Look-aside buffer</span>
{: .text-center }

#### Associative Register
- *Associative registers*(TLB) : parallel search 가능
  - TLB에는 page table 중 일부만 존재
- Address translation
  - page table 중 일부가 associative register에 보관되어 있음
  - 만약 해당 page #가 associative register에 있는 경우 곧바로 frame #를 얻음
    - 그렇지 않은 경우 main memory에 있는 page table로 부터 frame #를 얻음
  - TLB는 context switch 때 flush (remove old entries)

### Two-Level Page Table
![](/assets/images/study/dev/2021/os/ch8_two-level-page-table.png)
- 현대의 컴퓨터는 address space가 매우 큰 프로그램을 지원
  - 32 bit address 사용시: 2<sup>32</sup>Byte (==4GB)의 주소 공간
    - page size가 4kb시 1M(1,000,000)개의 page table entry 필요
    - 각 page entry가 4B시 프로세스당 4M의 page table필요
    - 그러나, 대부분 프로그램은 4GB의 주소 공간 중 지극히 일부분만 사용하므로 page table 공간이 심하게 낭비됨
  - page table 자체를 page로 구성
  
> 결과적으로 2단계의 페이지 테이블을 사용하는 목적은 **공간을 효율적으로 활용(공간을 줄임)**하기 위함이다.

![](/assets/images/study/dev/2021/os/ch8_address-translation-scheme.png)
- 처음 논리적 주소에서, P<sub>1</sub>에 해당하는 주소 공간으로 접근 후 P<sub>1</sub>의 엔트리를 확인하여 `2 depth`주소 공간을 확인한다.
- P<sub>2</sub>의 엔트리를 확인하여 물리적인 `Page Frame` 정보를 확인할 수 있다.
- 확인한 Page frame 정보를 논리적 주소에 대입하여 인덱스 *d*번째 물리적 메모리 공간을 접근한다.

그런데 한가지 의아한 점이 있d다.  
2단계 페이지 테이블을 통해서 공간을 줄이는 효과를 취한다고 설명했는데, 아키텍처상의 흐름을 보면  
1단계 테이블이 존재하고, 그에 따른 2단계 테이블(inner table)이 존재한다.  
즉, 1차원적으로 해결할 수 있는 것을 2단계로 쪼개놨기 떄문에 메모리 공간도 더 활용하게 되고, 두단계에 거쳐가기 떄문에 그에 따른 속도에도 영향을 받을 것이다.  
과연 이것이 공간을 줄이는 효과를 본다고 할 수 있을까?

페이징 처리 후 프로그램이 구동될 때, 실제로 사용되지 않는 페이지 공간이 대부분이다.  
실제 1단계 페이지테이블을 구성시, 각 페이지와 1:1로 맵핑되는 페이지 테이블의 엔트리를 보유하고 있어야 필요시 각 페이지에 해당하는 영역을 참조할 수 있기 때문에 사용하지 않는 동안에도 **maximum logical memory**의 크기 만큼 페이지 테이블이 그에 대응되어야 한다.

하지만 2단계 페이지 테이블을 통해서 이 문제를 해결할 수 있다.  
바깥쪽(1 depth) 페이지 테이블에선 전체 논리적 메모리 주소 만큼 생성되지만, 안쪽(2 depth)페이지 테이블에서는 사용하지 않는 테이블은 만들어지지 않고 포인터가 NULL 상태로 되어 있는 것이다. 실제로 사용되는 부분만 안쪽 테이블이 생성되어 물리적 메모리 공간을 가리키기 때문에 효과적으로 메모리를 사용할 수 있게 되는 것이다.


### Miltilevel Paging and Performance
- Address space가 더 커지면 다단계 페이지 테이블 필요
- 각 단계의 페이지 테이블이 메모리에 존재하므로 logical address의 physical address변환에 더 많은 메모리 접근 필요
- TLB를 통해 메모리 접근 시간을 줄일 수 있음
- 4단계 페이지 테이블을 사용하는 경우
  - 메모리 접근 시간이 100ns, TLB 접근 시간이 20ns이고
  - TLB<sub>translation look-aside buffer</sub> hit ratio가 98%인 경우
    - effective memory access time = 0.98 * 120 + 0.02 * 520
    = 128 nanoseconds.

    결과적으로 주소 변환을 위해 28ns만 소요됨


![](/assets/images/study/dev/2021/os/ch8_valid-invalid-table.png)
앞서 살펴봤던 페이지 테이블의 그림은, logical memory의 페이지 갯수 만큼 엔트리가 존재하고, 각 엔트리는 어떤 물리적 메모리에 연결되었는지를 보여주었다.

하지만 여기서 빠진 부분이 있는데, 엔트리에는 주소 정보만 존재하는 것이 아니고 각 엔트리마다 `bit`가 존재한다.  
bit는 **valid/invalid**로 표시되는 엔트리가 존재하는데, bit가 의미하는 것은 이런 것이다.  

그림에서 보면 논리적 페이지는 0~5까지 존재하지만, 엔트리는 0~7까지 *(논리적 페이지에 존재하지 않는 6,7이 존재)* 존재하는 것을 볼 수 있다.  
이렇게 존재하지 않는 페이지에도 엔트리가 존재하는 이유는 프로그램에 주소 공간을 갖을 수 있는 최대 크기만큼 페이지 테이블의 엔트리가 필요한데, 상위 부분의 페이지에는 `code`, `data`를 구성하는 페이지가 있고, 아래쪽에 `stack`을 구성하는 페이지와 그 사이에는 사용되지 않는 주소 영역이 존재할 수 있다.  
테이블 엔트리는 사용되지 않는 모든 영역까지 준비가 되어 있어야하기 때문에 엔트리가 만들어진다.  
(테이블이라는 구조 특성상, index로 접근하기 때문)

결과적으로 엔트리에는 페이지에 없는 6,7까지 생성이 되었으며, 실제로 사용되지 않기 때문에 valid/invalid bit에 의해서 i<sub>invalid</sub>로 표시된 것이다.

page 0의 엔트리 정보를 따라보면 (**frame number = 2** , v<sub>valid</sub>) 물리적 메모리 2번 공간에 page 0이 위차한 것을 확인할 수 있다.

#### Memory Protection
Page Table의 각 엔트리마다 아래의 bit을 둔다.
- Valid-invalid bit
  - `Valid` : 해당 주소의 farme에 그 프로세스를 구성하는 유효한 내용이 있음을 뜻함(접근 허용)
  - `Invalid` : 해당 주소의 frame에 유효한 내용이 없음(접근 불허)
- Proection bit
  - page에 대한 접근권한 (read/write/read-only)
    - 페이지마다 code, data, stack 각 영역을 담고 있을 수 있는데  
    특히 `code`영역은 내용의 변경이 있으면 안되기 때문에 *read-only* 권한을 주는 것이 적절하다.
    - `data`, `stack` 영역은 중간에 쓰고, 변경이 가능하기 때문에 *read*, *write* 권한을 모두 부여한다.

### Inverted Page Table
- Page table이 매우 큰 이유
  - 모든 process 별로 그 logical address에 대응하는 모든 page에 대해 page table entry가 존재함
  - page의 존재 유무에 상관없이 그에 대응하는 page table에는 entry로 존재
- Inverted page table
  - Page frame하나 당 page table에 하나의 entry를 두는 것(system-wide)
  - 각 page table entry는 각각의 물리적 메모리의 page frame이 담고 있는 내용 표시
  (process-id, procee의 logical address)
  - 단점
    - 테이블 전체를 탐색해야 함
  - 조치
    - associative register 사용 (expensive)

![](/assets/images/study/dev/2021/os/ch8_inverted-pagetable.png)
기존에는 **페이지 테이블-프로세스**은 1:1로 존재했다.  
하지만 inverted page table은 page table이 1개만 존재한다. 이 페이지테이블의 엔트리는 프로세스의 페이지 갯수만큼 존재하는 것이 아니라, 역으로 **물리적 메모리 공간의 페이지프레임 갯수만큼 존재**하는 것이다.

이전에 공부한 정방향 페이지 테이블의 구조는 논리적 메모리의 페이지 번호(p)를 따라서 그에 상응하는 p번째 엔트리를 조회하여 물리적 메모리의 주소를 접근했는데, *역방향 페이지 테이블*에서는 페이지 프레임(물리적 메모리에 속한) 갯수 만큼 엔트리가 존재하고 엔트리에 저장되는 내용이 반대로 되어 있다.  

즉 정방향에서는 **key = page inex, value = page frame address** 구조 였다면,  
역방향에서는 **key = page frame address, value = page index** 구조로 되어 있는 것이다.

하지만 기존의 주소 변환이라고 하면, 논리적인 주소 공간을 물리적인 공간으로 바꾸는 것이기 때문에 이러한 구조는 본래의 주소 변환 목적과 맞지 않는 구조 인데, 물리적인 주소 공간을 알고 싶다면 논리적 페이지 번호를 모든 엔트리를 조회해서 물리적인 주소 공간을 찾아낼 수 있다.(비용이 비쌈)

그럼에도 불구하고 역방향 테이블을 사용하는 이유는 무엇보다 **공간을 줄이고자 함**에 있다.  
(공간을 확보하는 대신 시간의 비용이 많이 발생함 => `associative register`라는 별도의 하드웨어를 통해서 엔트리들을 병렬적으로 동시에 검색해서 속도 향상)

### Shared Pages
- Shared code
  - Re-entrant Code (=Pure code)
  - *read-only*로 하여 프로세스간 하나의 code만 메모리에 올림
  (eg, text editors, compilers, window systems)
  - shared code는 모든 프로세스의 logical address space에서 동일한 위치에 있어야함
- Private code and data
  - 각 프로세스들은 독자적으로 메모리에 올림
  - Private date 는 logical address space의 아무 곳에 와도 무방함

![](/assets/images/study/dev/2021/os/ch8_shared-pages.jpeg)
서로 다른 프로세스 P<sub>1</sub>, P<sub>2</sub>, P<sub>3</sub>가 있는데, 이 프로그램들이 같은 코드를 실행시킨다고 생각해보자.  
예륻를어 3개의 `텍스트 에디터` 프로그램을 실행 시키면, 프로그램은 모두 동일한 **code 영역**으로 실행해도 무방하고, 중간에 입력되는 텍스트(=data)만 변경이 일어날 것이다.  
이처럼 프로그램에서 같은 코드를 접근하는 공유되는 코드 부분을 `shared code`라고 한다.

### Segmentation
> **Segment란** 의미 단위로 쪼개어 놓은 것을 말하는데, 예를들어 앞서 메모리 상에 code, data, stack으로 구분해놓은 각각의 의미있는 단위를 *segment*라고 한다.

- 프로그램은 의미 단위인 여러개의 `segment`로 구성
  - 작게는 프로그램을 구성하는 함수 하나하나를 세그먼트로 정의
  - 크게는 프로그램 전체를 하나의 세그먼트로 정의 가능
  - 일반적으로는 code, data, stack 부분이 하나씩의 세그먼트로 정의됨

#### Segmentation Architecture
- Logical address는 다음의 두가지로 구성
  - segment-number
  - offset
- Segment table
  - 엔트리에는 다음의 2가지 정보를 저장
    - base - segment의 물리적 저장 주소
      - Segment-table base register(STBR)
      물리적 메모리에서의 segment table의 위치
    - limit - 세그먼트의 길이
      - Segment-table length register(STLR)
      프로그램이 사용하는 segment의 수

![](/assets/images/study/dev/2021/os/ch8_segmentation-hardware.jpeg)
- CPU가 논리 주소를 주게 되면, `segment-number`와 `offset`으로 나눈다. 
- segment의 시작 위치는 STBR이 보유하고 있을 것이고, segment-number 만큼 떨어진 위치에 가면 물리적 메모리 어떤 공간에 위치해 있는지를 알 수 있다.  
그런데 paging기법에서는 page크기가 동일했던 것과 달리, segmentation기법에서는 **의미단위**로 구분을 하기 때문에 메모리의 크기가 균일하지 않을 수 있어서 물리적 메모리의 시작 위치(`base`)와 segment의 길이(`limit`)정보를 갖고 있다.
- 주소 변환시 2가지를 염두해두어야 한다.
  - 첫번쨰는 CPU에 주어진 주소에서 segment-number가 STLR의 길이보다 작은지를 확인해봐야한다.(segment-number가 클 경우 잘못된 시도 -> trap 발생)
  - segment의 길이(limit)가 1,000bytes 인데, offset의 크기가 2,000bytes를 요청한다면 역시 문제가 발생하여 trap을 발생시킨다.

#### Segmentation Archtecture
- 장점
  - Proection
    - 각 세그먼트 별로 protection bit 존재
    - 각 엔트리별로 설정
      - read / write / execution 권한 bit (segment요소 별로 권한 부여)
  - Sharing
    - shared segment
    - same segment number  

> segment는 의미 단위이기 때문에 공유와 보완에 있어 paging보다 훨씬 효과적 

- 단점
  - Allocation
    - first fit / best fit
    - 외부조각<sub>external fragmentation</sub> 발생

> segment의 길이가 동일하지 않으므로 가변분할 방식에서와 동일한 문제점들이 발생함

![](/assets/images/study/dev/2021/os/ch8_segmentation_memory.jpeg)

그림과 같이 segment를 사용 후 물리적 메모리 공간에 빈공간이 생기게 되는데(그림, 우측 메모리 공간 segment3, segment4),  
이렇게 비어있는 공간보다 큰 segment가 유입시 그 공간을 활용할 수 없으므로, hole들이 생기게 된다.(**external fragmentation 발생**)

그에 반해 테이블(segment table)은 실제 메모리에 사용하는 정보만 담고 있기 때문에 테이블의 공간은 훨씬 효율적으로 관리된다.

#### Sharing of Segments
앞서 `Shared page`에서 다루었던 예제와 같이, **텍스트 에디터**라는 프로그램을 2개 실행한다고 가정하자.  

![](/assets/images/study/dev/2021/os/ch8_sharing-segment.png)

그림과 같이 프로세스 P<sub>1</sub>, P<sub>2</sub> 모두 에디터를 실행하고 있는데, 이 둘은 모두 **base = 43062**메모리 공간을 가르키고 있다. 즉 두 프로세스의 `segment 0`은 모두 동일한 공간에 위치하고 있다.

하지만 data를 다루는 *private segment*인  `segment 1`의 경우 서로 다른 공간을 점유하고 있다.

### Segmentation with Paging
- Pure segmentation과의 차이점
  - segment-table 엔트리가 segment의 *base address*를 가지고 있는 것이 아니라,  
  segment를 구성하는 **page table의 base address**를 가지고 있음

![](/assets/images/study/dev/2021/os/ch8_segmetation-with-paging.png)
이 기법은 **segment 한개가 여러개의 page**로 구성되는 기법이다.

- 전통적인 segmentation의 경우 's'번째 엔트리에 접근하면 주소 변환 정보(segment-table)를 확인하여, 곧바로 물리 메모리 어느 곳에 위치했는지 확인할 수 있었다.  
- 하지만 이 기법은 segment 한개에 여러개의 page로 구성되기 때문에 메모리에 올라갈 때 페이지 단위로 쪼개져서 올라가게 된다.
  - 즉 각각의 segment 마다 크기가 달라서 발생하던 **allocation문제(external fragmentation)**를 자연스럽게 해결할 수 있게 된다.
- 실제로 오리지날 segmentation 기법을 사용하는 경우는 거의 없고, segmentation과 paging을 조합하여 사용한다.

---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books