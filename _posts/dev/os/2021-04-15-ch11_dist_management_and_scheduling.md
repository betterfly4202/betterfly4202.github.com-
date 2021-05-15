---
title: "[Operating System] Ch11. Disk Management and Scheduling"
categories: "OS"
tags:
  - Operating System
  - Disk
---
# Disk
## Disk Structure
### Logical block
- 디스크의 외부에서 보는 디스크 단위 정보 저장 공간들
- 주소를 가진 1차원 배열처럼 취급
- 정보를 전송하는 최소 단위

### Sector
- Disk를 관리하는 가장 작은 단위
- Logical block이 물리적인 디스크에 매핑된 위치
- `Sector 0`의 위치는 가장 바깥쪽 실린더의 첫 번째 트랙에 있는 첫 번째 영역(**부팅**과 관련된 영역)

> 디스크 외부에서는 `Logical Block`단위로 디스크를 바라보고 있으며, 실제로 디스크 내부에는 `Sector`에 매핑이 되어 관리되는 구조

## Disk Management
### Physical formatting(Low-level formatting)
- 디스크를 컨트롤러가 읽고 쓸 수 있도록 sector들로 나누는 과정
- 각 섹터는 `header` + `실제 data` (512 bytes) + `trailer` 로 구성
  ![](/assets/images/study/dev/2021/os/ch11_disk-sector.jpeg)
- **header**와 **trailer**는 sector number, ECC<sub>Error-Correcting code</sub>등의 정보가 저장되며, controller가 직접 접근 및 운영

### Partitioning
- 디스크를 하나 이상의 실린더 그룹으로 나누는 과정
- OS는 이것을 독립적 disk로 취급(logical disk)
  - Windows에서 C드라이브, D드라이브 이렇게 나누는 작업

### Logical formatiing
- **파일 시스템**을 만드는 것
- FAT, inode, free space 등의 구조 포함

### Booting
- ROM에 있는 "*small bootstrap lodaer*"의 실행
- sector 0 (boot block)을 load하여 실행
- sector 0은 "*full Bootstrap loader program*"
- OS를 디스크에서 laod하여 실행

## Disk Scheduling
![](/assets/images/study/dev/2021/os/ch11_disk.png)
### Access time
- Seek time
  - 헤드를 해당 실린더(트랙)로 움직이는데 걸리는 시간
  - 기계 장치가 이동하므로 가장 큰 시간이 소요되는 시간
  - 따라서 **한번의 seek으로 최대한 많은 양의 데이터를 읽는 것이 성능 향상에 매우 큰 영향**을 줌(효율성 증대)
- Rotational latency
  - 헤드가 원하는 섹터에 도달하기까지 걸리는 회전지연시간
  - seek time/10 정도의 시간 소요
- Transfer time
  - 실제 데이터의 전송시간
### Disk bandwidth
- 단위 시간 당 전송된 바이트의 수
### Disk Scheduling
- seek time을 최소화하는 것이 목표
- Seek time = Seek distance

## Disk Scheduling Algorithm
### FCFS(First Come First Service)
![](assets/images/study/dev/2021/os/ch11_disk-scheduling-algorithms-fcfs.png)
- Queue에 들어온 순서대로 처리

### SSTF(Shortest Seek Time First)
![](assets/images/study/dev/2021/os/ch11_disk-scheduling-algorithms-sstf.png)
- Queue에 정의된 head의 위치를 정렬하여, 현재의 head위치에서 가장 가까운 순서대로 처리
- Starvation 문제 발생
  - 현재의 head위치와 가까운대로 처리를 하면서, Queue에 계속해서 인접한 head값만 들어올 경우 뒤에 위치한 head에 도달하지 못하는 문제

### SCAN(Elavator)
![](assets/images/study/dev/2021/os/ch11_disk-scheduling-algorithms-scan.png)
- Elavatmor의 움직임과 비슷하게 처리하는 알고리즘 방식
- disk arm이 디스크의 한쪽 끝에서 다른 반대편 끝까지 이동하면서 가는 길목에 있는 모든 요청을 처리
- 다른 한쪽 끝에 도달하면 역방향으로 방향을 전환하여, 이동하면서 오는 길목에 있는 모든 요청을 처리하며 반대쪽 끝으로 이동
- 문제점 : 실린더 위치에 따라 대기 시간이 다를 수 있음

### C-SCAN(Circular SCAN)
![](assets/images/study/dev/2021/os/ch11_disk-scheduling-algorithms-c-scan.png)
- 헤드가 한쪽 끝에서 다른쪽 끝으로 이동하며 가는 길목에 있는 모든 요청을 처리
- 다른쪽 끝에 도달했으면 요청을 처리하지 않고 곧바로 출발점으로 다시 이동
- `SCAN`보다 균일한 대기 시간을 제공

### C-Look
![](assets/images/study/dev/2021/os/ch11_disk-scheduling-algorithms-c-look.png)
- C-SCAN을 개선한 알고리즘
- C-SCAN의 경우 요청사항이 없더라도, Queue에 정의된 한쪽 방향의 끝까지 이동하면서 처리하는 알고리즘인데 반해, 
C-LOOK은 진행 방향에 더이상 요청사항이 없으면 즉시 방향을 전환하여 반대 방향으로 전환함

## Disk Scheduling Algorithm의 결정
- SCAN, C-SCAN 및 그 응용 알고리즘은 LOOK, C-LOOK 등이 일반적으로 디스크 입출력이 많은 시스템에서 효율적인 것으로 알려져 있음
- File의 할당 방법에 따라 디스크 요청이 영향을 받음
- 디스크 스케줄링 알고리즘은 필요할 경우 다른 알고리즘으로 쉽게 교체할 수 있도록 OS와 별도의 모듈로 작성되는 것이 바람직함

## Swap-Space Managment
- Disk를 사용하는 두가지 이유
  - memory의 volatile한 특성(휘발성) -> `File System`
  - 프로그램 실행을 위한 memory 공간 부족 -> Swap space (swap area)
- Swap Space
  - Virtual memory system에서는 디스크를 memory의 연장 공간으로 사용
  - 파일 시스템 내부에 둘 수도 있으나, 별도 partition 사용이 일반적
    - 공간효율성보다는 속도 효율성이 우선
      - 공간 효율성이 중요하지 않는 이유는, 임시로 점유되는 공간이기 때문에 프로세스 종료시 제거되기 떄문
    - 일반 파일보다는 훨씬 짧은 시간만 존재하고 자주 참조됨
    - 따라서, block의 크기 및 저장 방식이 일반 파일시스템과 다름

![](/assets/images/study/dev/2021/os/ch11_swap-space-management.png)
- 하나의 물리적인 디스크를 논리적 공간으로 파티셔닝하여 구분할 수 있다.
  - File System은 각 섹터당 512bytes의 크기로 데이터를 저장
    - 파일 저장 방식은 FAT, inode, free space 등 사용
  - Swap Area는 프로그램이 실행되는 중, 각 프로세스가 주소 공간을 사용하여 프로그램 종료시 사라지게 된다.
    - 물리적 메모리의 연장 공간으로 활용되기 때문에, 제거되거나, page-fault 발생시 가능한 빠르게 디스크에 써줘야한다 

## RAID(Redundant Array of Independent Disks)
![](/assets/images/study/dev/2021/os/ch11_raid.png)
- 여러개의 디스크를 묶어서 사용하는 방식(각 디스크에 중복 저장)

### RAID의 사용 목적
- 디스크 처리 속도 향상
  - 여러 디스크에 block의 내용을 분산 저장
  - 병렬적으로 읽어옴(*Interleaving, Striping*)
- 신뢰성(reliability) 향상
  - 동일 정보를 여러 디스크에 중복 저장
  - 하나의 디스크가 고장(failure)시 다른 디스크에서 읽어옴(*Mirroring, Shadowing*)
  - 단순한 중복 저장이 아니라, 일부 디스크에 parity를 저장하여 공간의 효율성을 높일 수 있음 
    - *parity* : 오류가 발생했는지의 여부와 복구할 수 있을 정도의 중복 저장만 하는 방식

---

*Reference*
- Operating System Concepts (Paperback / 9th Ed.) Books
- https://www.slideshare.net/PareshParmar6/disk-scheduling-algorithms-71247712
- https://jhi93.github.io/category/os/2019-12-29-operatingsystem-11/