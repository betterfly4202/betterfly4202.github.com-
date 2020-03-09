---
title: "Google Clound Summit Next - Kubernetes"
categories: "Conference"
tags:
  - Kuberenetes
---

인프라는 컴퓨터의 전체적인 프로세스를 이해해야하는 가장 기본적이고 중요한 개념이다.

하지만 그런 개념이 부족하여 공부해야겠다는 생각만할 뿐. 현실은 당장에 사용될 기술 스펙들에 더욱 집중하고 있었다.

그러던 중 아주 좋은 기회에 쿠버네티스에 대하여 상세하고 실습까지 할 수 있는 워크숍이 마련되어 운좋게 참석할 수 있었다.

스스로 개념도 완벽하게 정리되지 않았고, 방대한 내용을 다루었기 때문에 알아 들을 수 있었던. 쉽게 이해된 내용만 급하게 정리하였다.

---

## 컨테이너 기술(Docker)의 특징
- 컨테이너 기술의 기본 개념 : VM(Virtual Machine)과 같이 OS 이미지를 뜬다는 개념.
- VM은 인프라부터 전부 가상화 하여 OS까지 묶지만, 컨테이너 기술은 Host OS를 기반으로 패키징할때 필요한 부분만 가져다 쓸 수 있다.(경량화)
- VM은 vmware, virtual box, azure vm 등과 같은 다양한 종류가 있어 각각의 사용법이 조금씩 다를 수 있지만, 도커는 어떤 환경(azure, aws, gcp)에 관계없이 패키징한 시스템으로 동일하게 배포할수있다. (portable)

> 컨테이너를 적정한 하드웨어에 분산배치해주는 기술

> Tutorials : https://google.qwiklabs.com

## Concept

- Pods : 쿠버네티스 최소의 배포 단위 (쿠버네티스 노드에는 최소 하나의 pod로 구성된다.)<br/>
  하나의 pod에서는 디스크와 네트워크를 공유(application - infra 영역을 공유하여 사용함)
  > TIP : (Pod config 관리를 위해 .yaml 확장자를 사용해야하는데, 까다로운 인덴트를 관리하기 위해 Atom, VS Code를 이용하면 편리하다.)

- Kuberenetes Namespace : GCP의 Project와 비슷한 개념

## Label
Kubernetes 의 리소스에 각각의 태그를 달 수 있다(태그에 따른 각각의 명령어 사용 가능) -> 태그 설계(pod별로 select할 수 있음)
(Set based selector -> 집합의 개념)

## Replication Controller
pod를 관리해주는 것들 -> Controller 그 중 대표적을 사용하는 것 Replication Controller

pod의 사용/배포/관리 의 컨트롤 타워

## Replicaset
Replication Controller의 개념과 같지만 Replication Controller는 등호방식, set은 집합 방식

 ## Deployment
 Kubernetes -> [Deployment > ReplicaSet > Pod > Service 의 구조]

 ---

 ## Job
 Replication은 자동으로 pod을 실행해주면서 계속 리스타트 해줌
 
 Job은 배치의 개념 -> 설정한 Job에 따라 진행되며, 종료시 실행되지 않음
 (Job 오픈소스 : Argo)

 ## StatefulSet
pod는 stateless 때문에 네이밍을 해도 네이밍-code와 같이 부팅시마다 새로운 이름이 부여된다.(RDB와 같이 정확한 값을 요구할 때 문제됨)

StatefulSet 설정을 통해 네이밍-index 와 같이 일정한 네이밍을 부여하여 관리할 수 있다.


### kube-dns
> 내부 dns(외부에 노출되지 않음). 이름이 계속 바뀌기 때문에 내부에서 관리되기 위해서 이름이 붙여짐

# Kuberenetes Dashboard는 절대 사용하지 말 것!(보안 이슈)


### AMbassador Pattern
 다이렉트로 거치는게 아니라 Ambassador를 거쳐서 서비스에 접근함(proxy의 개념)

 ### Side Car
 메인 컨테이너 옆에 side car를 두어 서브 관리

 ### Adapter
 format이 틀린 것을 같은 형태로 바꾸어 주는 패턴

 [참고]
 - https://microservices.io/
 - https://docs.microsoft.com/ko-kr/azure/architecture/patterns/sidecar
 - https://www.katacoda.com/
 - https://www.katacoda.com/courses/kubernetes/launch-single-node-cluster

--- 
https://kubernetes.io/ko/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/

---

## Ingress
Service와 같은 로드밸랜서의 역할

URL Path based routing -> url base로 routing해줌 (ex: /user/*, /product/*)
L7 Level, SSL 터미널이 가능함. 기본 GCP는 L4레벨로 TCP접근


# Kubernetes Deployment
Servier > Pods >> Replication Controller
rolling update(하나씩 바꾸는 방식), re-create(기존 다지우고 새로 연결)

- Blue/Green : A > B 통쨰로 버전 변경 (롤백 불가)
- Canary : 같은 버전으로 복제본을 놓고 하나씩 테스트해서 정상적이면 적용

---

## Configmap and secret
- 환경변수를 넘기고 싶을때..(String 을 넘길수도, File자체를 넘길 수도)

---

## Volume type
Type : temp, local, //  network

>PersistentVolume

---

>kubectl scale --replicas=3 statefulset mongo

mongo라는 서비스에 statefulset 의 볼륨 조건으로 pod을 scale 한다는 것.

Scale in : 확장 > 증대
Sacle out : 축소 > 제거

> kubectl get statefulset 

statefulset 의 상태를 확인할 수 있음

---

Cloud 환경이 있는데 kubernates와 같은 docker를 사용해야 하는 이유.
단순히 portable하고 자동화의 초점 뿐만 아니라,
GAE의 경우, 우리는 GCP에서 제공하는 was 이미지 자체를 사용해야 한다.
하지만 vm, kubernetes와 같은 서비스를 통해 구처젝언 config 설정 등을 컨트롤 하기 위해 쓰인다.

---

## Advanced Scheduling

### Taint & Toleration
- Taint : node 에 붙이는 label >> 여기에는 pod 배포하지마
- Toleration : 특정 node에 접근하기 위한 구분자 같은 것

>master node는 기본적으로  taint로 관리해서 일반적인 node의 접근을 막는다.

- **node affinity : 우리 node로 와줘**
- hard node affinity  : key 에 설정한 값이 매칭되면 무조건 감 >>> 1순위

- soft node affinity  : key 에 설정한 값에 움직이지만, '되도록이면' 접근 >>> 차순위 ,우선순위 할당 (a,b,c 중 조건에 가장 가까운 곳에)

hard가 매칭된 상태에서 soft 조건이 맞으면 soft상태를 바라봄..

--- 

### pod affinity : pod를 배포할때 node를 바라보는 것이 아니라, 그 노드 안에 있는 pod를 바라봄
> ex : 톰캣 pod를 배포하는데, db가 있는 node를 찾는 것이 좋기 떄문에 db의 pod가 있는 node를 찾아 접금한


### pod anti-affinity : 해당 pod를 피해서 배포됨

## topologyKey
예를들어, node에 존재하는 topologyKey를 찾아서 해당 노드?(존?) candidate가 된다.

---

## Monitoring
Host > Container > App > Kubernetes 구조. 각 계층에서 monitoring이 필요함 (보통은 app 정도만 생각함)

결론 : Stackdriver 쓰세요! 두 번 쓰세요!

---

## Security
Namespace 단위로 Resource > Role 권한을 관리함(예를들어 pods : watch, get, list) 권한 을 설정함
Role Assign해서 특정 유저에게 권하을 부여하여 관리할 수 있음

Cluster Role을 관리하면 > 

보안 원칙 : 불편할수록 안전하다(쉬울수록 잘 털림)

### Network Policy
Ingress control : 방화벽의 개념(IP차단) + Label 접근까지 차단

### Security Context
컨테이너에 security 기능 정의 하는것

> Dockerfile 컨피그에 설정을 하지 않으면 root권한으로 실행되기 때문에 반드시 처리할 것

---

## Resource Management
- cpu와 memory 넣는 것

현재 쿠버네티스에 넣을 수 있는 리소스는 cpu와 memory밖에 없음.

---

# Beyond KUBERNETES

[MSA대표 디자인 패턴]
Circuit breaker pattern
**Netflix** Spring Hystrix Sample이 잘 구현한 예
공부해 볼 것.

> 서버가 죽으면 디비풀링할것이 아니라, static하게 박아 놓아서 장애 차단

application에서 처리할 수 있는 부분을 infra에서만 처리하려고 하면 위험하다 -> Over Engineering

---

# Spinnaker
> CI/CD

Open Source
- Ambassador : proxy pattern의 디자인패턴
- Argo : Workflow engine for Kub.

---

- www.coursera.org
GCP 강의(유료)