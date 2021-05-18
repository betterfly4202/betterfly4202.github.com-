---
title: "[Webinar] MongoDB"
categories: "DB"
tags:
  - mongodb
---

# Mongo DB

무료 교육

[Free MongoDB Official Courses | MongoDB University](https://university.mongodb.com/)

**ACID Transaction**

[[DB이론] 트랜잭션(transaction)과 ACID 특성을 보장하는 방법](https://victorydntmd.tistory.com/129)

이전 단일 어플선택하여 클라우드 이동하고 유사한 관계형 디비로 이동함

레거시 워크로드를 클라우드 유사 아키로 옮겨서 얻는것은? 투자 수익률

---

몽고디비는 어떻게 저장하느냐

# 몽고 장점

## 첫째

rdb : 테이블 형태로

→ 서로 다른 성격이 있을수 있지만, rdb는 필드를 추가해야만 사용할 수 있음 (초개인화시대에 적합하지 않음)

mongodb : document 형태이기 때문에 데이터를 자유롭게...

## 둘째

복제셋과 샤드 구조

- 고가용성을 위한 레플리케이션 구조
- 스케일 아웃을 위한 샤딩 구조

![Mongo%20DB%20666d6ede29e74555ad3bca5825655e94/Untitled.png](Mongo%20DB%20666d6ede29e74555ad3bca5825655e94/Untitled.png)

---

왜 몽고DB 인가?

## Scale out

모바일 ↔ 웹 양 기종간 사용시 사용 기록을 그대로 유지해주기 위함(네이버 웹툰에 적용됨)

1. 개인화 영역의 방대한 데이터 처리 필요(data size, traffic)
2. 조회에 대한 다양한 조건 필요(secondary index)
3. 데이터 및 트래픽의 분산 처리 및 조회 성능 극대화 할 DB 필요

## Flexible schema

같은 기능을 각각의 서비스 개발팀에서 만들어야하는 이슈가 있음

(위에 말했던 장점과 동일함)

네이버 카페, V Live 등 각 서비스에서 같은 데이터플랫폼이 필요

고정적인 스키마가 아닌, 서비스에 따른 유연한 스키마를 필요로 함

---

구조

App server

Router(mongos)

|

Config Servers(Replica set)

|

Shard(replica set)

- removeShard()문제 → 삭제시 복구가 불가능함
removeShard시 특수한 경우 데이터 유실 발생 가능 → 유실 시 복구 불가능함
- primary-secondary-arbiter 구성문제 (아비터 데몬을 사용하는 경우)
아비터 데몬 사용시, 특수한 환경에서 서비스에 문제가 발생할 수 있음
프라이머리가 정상 살아있음에도 세컨더리 서버에 문제 발생시 프라이머리에 캐시 프레셔가 발생할 수 있음 ⇒ 극복 불가
때문에 빨리 세컨더리와 아비터를 정상화 시켜야만 함
- primary-secondary-arbiter 구성문제
과반 투표를 위해 아비터 데몬 사용 → 특수한 경우 문제
프라이머리 
이 구조는 클러스터에 문제가 발생하더라도 서비스에 지장이 없을때 사용해야함

- 아비터 서버 사용시 multi document transaction 되지 않는 문제
primary - secondary - secondary 구성으로 몽고디비 사용시
secondary 중 서버 하나 죽을 경우 HA 유지 위해 아비터 데몬을 임시로 붙인 후 해당 secondary  정상화 시킨 후 다시 교체 투입을 함
하지만 서비스의 HA 유지위해서 아비터서버를 사용하는 순간
서비스쪽에서 multi document transaction 기능을 사용하고 있었다면 해당 transcation은 모두 실패됨 따라서 멀티 도큐 트랜잭션 사용하는 서비스에서 아비터 데몬을 사용하지 못함
이 부분을 인지하고 있어야함

primary - se - ar 구성 사용 시 무엇보다 pri - secon 중 어느 한대 문제 발생시 혹은 해당 멤버가 복구 불가능한 경우가 문제인데 몽고디비에서는 백업을 restore하는 기능이 없음
따라서 한쪽 멤버가 문제가 발생했을떄 initial sync하거나 정상적인 멤버 데몬을 중단 후 해당 데이터를 그대로 카피하는 방법 말고는 복구할 수 있는 방법이 없음

initial sync는 데이터 사이즈, 구조에 따라서 매우 오랜시간이 걸릴 수 있고
캐시프레셔 문제를 똑같이 갖고 있음
따라서 새로운 멤버를 만들어야 하는 경우 기존 정상 멤버 중단 후 해당 멤버의 데이터를 그대로 복제하여 새로 구성함
프-세-아 구성에서는 한쪽 문제가있고 한쪽 멤버로 서비스를 하고 있는 상황인데
새로운 멤버 만들기 위해선 해당 멤버를 킬해야하는데 → 서비스 장애 발생
따라서 이런 경우 프-세-아 경우에 이런 저런 문제 발생할 수 있음을 반드시 인지해야함
특정 서비스에 문제가 발생하지 않는 경우에만 이렇게 사용해야함

- 단일 서버 multi daemon 사용의 문제
몽고디비 클러스터 구성시 하나의 서버에 여러 데몬을 구성하는 경우가 있는데,
좋지 않음
왜냐하면 OS 힙 영역의 충돌 발생, 메모리 최적화 문제 발생
따라서 단일 서버에 멀티 데몬은 권장하지 않음
이렇게 하고싶으면 리플리카셋을 구성하는 것으로 권장함

찾아볼 키워드

- 이니셜 싱크

    [Replica Set #2 (MongoDB Replication) - MongoDB DBA Exam mongodb](https://m.blog.naver.com/PostView.naver?blogId=ijoos&logNo=221325427941&proxyReferer=https:%2F%2Fwww.google.com%2F)

- 캐시 프레셔 이슈

---

4.4 는 안정화 되지 않았다고 판단

signing key 이슈가 있음  4.2.12 버전에서 해결되었음

---

## Task Executor Pool

sharding cluster 에서 mongos와 shard 사이의 connection

이슈 발생시 이 지표가 급등하고 DB 장애로 연결

샤딩 클러스터를 사용한다면, 어플 작업 처리하는 mongos가 있고 실제 데이터가 있는 실제 작업을 처리하는 샤드 멤버사이에 커넥션이 필요함

왜냐하면 각각 다른 데몬이기 때문

몽고스와 샤드멤버사이의 커넥션풀을 task executor pool 이라고함

따라서 샤딩 클러스에 이슈 발생시, 제일 먼저 이 풀이 급증하는 문제가 발생함

가장 기본 확인을 위해서 이 풀을 모니터링하고 있음

지금 서비스가 느린데 확인좀 부탁드립니다 ⇒ 이 풀이 정상인지부터 확인함

정상인 경우 ⇒ 몽고디비는 문제가 없습니다 라고 할 수 있음

몽고디비 운영에 가장 중요한 지표 중 하나

**반드시 모니터링 해야함**

<모니터링이나 알람 구축>

## Tcmalloc.spinlock

몽고 어그리게이션 쿼리 사용시 몽고 내부에서 tcmalloc 알고리즘 사용하는데,

어그리게이션 쿼리는 아무리 데이터 셋이 작아도 몽고 내부에서 16mb 할당 받아야하는데 

그런데 tcmalloc에서 16mb 할당받기 위해선 센트럴 힙을 할당받아야하는데 이 힙은 동시에 하나만 받아야함

따라서 몽고디비클러스터에서 동시에 어그리게이션 쿼리 혹은 많은 수의 카운터 쿼리를 호출시 

tcmalloc이 센트럴 힙을 할당받기 위해 대기를 탈 수 밖에 없음

이것을 `spinlock wait`이라고 함

따라서 tcmalloc의 spinlock wait을 모니터링 해야함

## Eviction

필요한 메모리 공간을 확보하기 위해서 메모리 버퍼에서 오래된 순으로 지워주는 작업

문제는 어플리케이션에서 갑자기 쿼리가 몰리게 될 경우 원활한 메모리 할당을 받지 못하게 될때

어플리케이션 스레드에서 직접 eviction 을하게 되는데 이 경우를 application eviction 이라고 함

application eviction 발생시 해당 스레드는 eviction 작업 이후 자기가 필요한 메모리를 할당하고 필요한 작업을 수행 후 클라이언트(어플리케이션)에 쿼리 수행결과를 응답해주기 때문에

그만큼 쿼리 응답 지연이 발생함

따라서 몽고 클러스터에서 application eviction 발생 여부 확인 및 모니터링이 필요함

## Read & Write ticket

몽고디비에서 서버스펙 상관없이 읽기/쓰기 처리하기 위한 백그라운드 스레드가 128개로 고정되어 있음

따라서 동시에 읽기 작업 128개가 수행되면, 그 다음부터 백그라운드 티켓을 할당받을떄까지 대기를 하게 된다. 따라서 티켓이 고갈되어있다면 티켓을 할당받지 못한 작업들이 대기를 해서 쿼리 응답이 지연될 수 있음

따라서 read write ticket 에 대한 모니터링도 필요함

**이 부분을 이해하고 있고 모니터링/알림을 적용해야 함**

---

# 저장 및 검색 최적화를 위한 Aggregation framework

## Aggregation

find, findOne으로 일반 조회 가능,

하지만 agg 를 통해 훨씬 다양한 operation 가능

예를들어, 

- 기존 필드가 아닌 연산된 필드
- 요약 및 그룹화된 값
- 다큐먼트 구조를 변경

예를들어 embedded document 를 root document로 올릴수도있고

array 안에 embedded document 를 여러 다큐먼트로 찢을 수 있음

이와 같이 find의 제한된 연산을 조금 더 활용하는것

Pipeline 이란?

한개 이상의 파이프로 구성됨

각각의 파이프는 하나의 변환단계를 거침

하나의 거대한 SQL 문장이 아닌, 단계별 pipe로 구성되어 있음

- 이해하기 쉬움
- 디버깅이 쉬움
- mongodb가 재작성하거나 최적화하기 쉬움

단순 조회시 find()가 훨씬 효율적임.

### aggregation을 쉽게 쓰는 방법

파이프라인을 한번에 작성하기 보다는, 변수에 어그리게이션 스테이지를 작성하고 파이프라인에 추가하여 테스트/디버깅을 쉽게할 수 있음

일반적으로 여러 파이프라인을 여러 어레이로 작성하면 수정/디버깅에 어려움

따라서 각 스테이지를 분리하고 의미를 부여하면 재사용하거나 디버깅에 쉬울 수 있음

GUI Aggregation Builder

Compass 툴 → Aggregation Builder 시각화 도우미가 있음

aggregation code를 export할 수 있고 (import도 가능) python, node, java 소스로 export 가능함

**"Join" Stages**

$lookup

left outer join 혹은 nested select와 비슷함

하지만 관계형 디비처럼 사용하면 안됨(기존 알디비 스키마를 몽고디비로 그대로 가져와서 lookup을 통해 join 연산을 사용하는건 비효율)

$lookup 연산을 구현하기전에 먼저 스키마 디자인을 검토해야함

몽고디비에서 굳이 컬렉션간 조인을 해야하는지 먼저 검토해야함

불가피하게 조인을 해야한다면 그때 $lookup을 사용

왜냐하면 데이터를 조인할때 항상 연산비용이 크게 발생하므로 스키마 디자인을 잘 구성해야함

또한 조인되는 컬렉션이 커질수록 선형적으로 증가하기 때문에 성능에 문제가 큼

$lookup은 단순히 두개 컬렉션 필드로 조인할수도있지만 

타겟 테이블에 어그리게이션 연산 수행 후 해당 결과로 조인을할수도 있음

또한 하나이상의 컬렉션을 연속해서 $lookup할수도 있음

이때 주의할 것은 $lookup을 수행하며 $lookup 필드에 대한 인덱스는 항상 확인해야함

**More $grouop-ing**

$bucket : $bucket 에 조금 더 특수화된 그룹 연산자

$bucket 은 추가 표현식 없이 특정 범위의 값을 조회 가능함

$bucketAuto : 몽고 디비가 자동으로 그룹핑하는 바운더리를 정해줌

$facet: 여러 하위 집계를 실행 후 단일 문서에 포함할 수 있는 방법

→ 연도별, 지역별, 성별 집계등의 데이터를 구성할 수 있음

**Grouping : $sortByCount**

$group + $sort 와 동일

특정 필드의 가장 공통된 값을 찾기 위한 방법

## Join

## Merge

`$out` : 이미 존재하는 컬렉션의 데이터를 넣을때 이미 존재하는 데이터를 삭제 후 데이터를 삽입

`$merge` : merge 컬렉션의 데이터가 있어도, 사용자가 선택적으로 업데이트할지 replace할지 선택 가능

$merge를 사용하려면 반드시 컬렉션에 유니크키가 있어야함 → 머지를 위한 기본 키 조건

사용자가 지정하지 않으면 디폴트 : `_id` (upsert와 동일)

```jsx
{
	$merge:{into:<target>,
		whenNotMatched: "insert", 
		whenMatched:"merge"
	}
}
```

만약 $merge를 통해 update할때 추가적으로 로직을 통해 타겟 데이터를 가공하고자 한다면 

⇒ aggregation pipeline 통해서 수행 가능

```jsx
{
	$merge:{into:<target>,
		let:{new :"$$ROOT"},
		whenMatched: [
			{
				$set: { 
					total: {$sum: ["$total" "$$new.total"]}
			}
		]
	}
}
```

- $total : 기존 타겟에 있던 컬렉션의 필드값
- $$new.total : merge되는 신규 소스코드의 도큐먼트

## UnionWith

기존 RDB 유니온의 한계

- UNION 하는 쿼리들이 동일 개수의 컬럼을 projection
데이터의 성격은 비슷하지만 스키마가 다를 경우 두 개의 테이블을 유니온할때
기존 테이블을 신규 스키마로 이관하거나, 공통되지 않은 컬럼을 포기하고 유니온해야함
- UNION 하는 쿼리들의 컬럼순서가 동일해야 함
- UNION 하는 컬럼의 데이터 타입이 호환 가능해야함

몽고디비는 스키마와 상관없이 유니온이 가능함 

⇒ RDB 제약에서 벗어남 : 스키마 유연성 및 수평 확장 가능

## UNION ALL 을 통해서 Single View 를 생성할 수 있음

서로 다른 소스, 서로 다른 스키마에 유사한 성격의 데이터를 가지고 하나의 view로 UNION이 가능함

---

Aggregation Optimization

- Pipeline은 자동으로 단계를 병합하고 필요에 따라 재정렬함
사용자가 $sort → $match 를 하더라도, optimizer가 내부적으로 순서를 바꿔서 $match → $sort를 하게끔 최적화를 함

일부 사용자들이 프로젝트를 통해 aggregation pipeline을 중간 단계에서 데이터를 줄이려고한다면
예를들어 평균이나 섬을 구하는 등의 새로운 필드를 산출하는 것이 아니라면,
이 부분은 몽고디비의 옵티마이제이션 작동에 도움이 되지 않음

따라서 사용자는 중간 단계에서 데이터 양을 줄이기 위해 프로젝션하는 방법은 도움이 되지 않음

또한 어레이 사용에서 어레이의 평균이나 합을 구할때,
$unwind를 수행후 그룹하는 경우가 있는데 → 이 경우 $unwind 보다는 $map, $reduce와 같은 accumulator를 사용해서 값을 산출한 후에 group 하는 것을 권장함

aggregation pipeline 각 단계는 streaming 이거나 blocking 형태로 작동함
예를들어 streaming 형태는 데이터를 프로젝션하거나 필드를 추가하거나 다큐먼트를 트랜스퍼 하는 경우
블록킹하는 경우는 sort, grouping, bucketAuto와 같은 그룹데이터를 중간 연산하는 것을 말함
⇒ $sort, $groupo, $bucket, $facet 이 단계들은 모든 결과를 처리할때까지 다음 단계를 block 함
따라서  데이터양이 많고 위와같은 연산을 하는 경우 하나의 aggregation을 하는 것보다는 데이터를 분산해서 parallel로 수행하는 것을 권장함

---

# On-premise에서 mongoDB 확장성 구성

## 개발 및 운영측면에서 샤딩을 구성하는 것이 어렵고 복잡하다

mongodb 는 기존의 데이터플랫폼에서 커버되지 않는 sharding 영역을 지원하는 것이 핵심

rdbms에서는 수평 partitioning 이라고 불림

기존 on-premise 구성하려면 mongos-config-shard 이 3개의 롤이 필요하고

각 롤의 특성을 살릴 수 있는 몽고 디비 아키텍처 구현위해서는

mongos * 2 - config * 3 - shard * 6

이렇게 최소 11대의 서버 구성 필요함

기본적으로 구성되는 서버가 다른 디비보다 많음

이렇게 아키텍처 복잡성, 한번 샤딩 구성후에는 해제 불가

샤딩을 구성하면 하드웨어 리소스와 부하는 줄어들 수 있지만, 샤딩으로 구성후에는 레플리카셋으로 변환이 불가함

**⇒ 클러스터에서 데이터가 자동으로 옮겨 다니기 때문**

MongoDB 샤딩 : 다중 replica Set에 데이터를 분산 저장하는 구성

특징 : 데이터/트래픽이 증가하여 더이상 하나의 replica set에 담을 수 없게 되면 데이터를 샤딩하여 부하는 분산시켜주는 역할

## DB 고가용성은 보장하지만, 서버장애시 완전 보장은 어렵다

가용성 제공을 위해 replica set으로 구성하면 알아서 primary 투표 후 선출하기 때문에 메뉴얼한 작업이 필요 없음

하지만 대규모 서비스에서 primary 서버 장애, 서버 복구 불가상태, secondary 이슈가 생길 시 데이터 유실 가능성을 배제할 수 없음

⇒ 대규모 서비스에서 primay 서버 장애시 복구하는데 장시간 소요 및 데이터 유실 위험

Replica Set : 데이터의 동일한 복사본을 여러 서버에 보관하는 방법임

*Primary(클라이언트 요청 처리) - Secondary 여러 대 (Primary 복사본)

특징 : 한 대 서버 장애에도 서비스 고가용성을 실현하여 데이터를 안전하게 보관함

## Secondary 이슈 발생시 백업본으로 NAS 없이는 Secondary 구성작업이 어려움

secondary 이슈가 생겨서 신규로 secondray를 만든다면, 백업본으로 secondray를 만들 수 없음

때문에 primary에서 initial sync 작업을 해야함

스냅샷을 떠놓지 않은 경우 initial sync 동안 서비스 다운 타임이 장시간 소요될 수 있음

⇒ 따라서 on-premise환경에서 몽고 운영하며 nas 사용하지 않으면 데이터 스토리지를 복사해서 처음부터 재구성해야하기 때문에 시간이 많이 소요되고 관리가 어려움

정리 

1. 몽고디비는 기본적으로 구성되는 서버가 다른 디비보다 많음
⇒ 서버 증설시 샤드별 별도 수동으로 구성하는 경우 설정 오류 위험이 높음

2. 백업본으로 Secondary 생성이 불가
⇒ 샤드 서버에 스펙을