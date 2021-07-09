---
layout: post
category: etc
title: 시스템 디자인 인터뷰
tags: [System Design]
---

시스템 디자인 인터뷰 준비를 위한 요뷰

# 기본 컨셉들

- Performance vs Scalability
  - 성능: 단일 워크로드에 대한 처리 능력 - Scale Up
  - 확장성: 여러개의 워크로드에 대한 처리 능력 - Scale Out
- Scalability의 난점들
  - Resource의 수에 비례해 빨라지면서, 동일한 결과
  - 일부 Hardware의 성능에 비례해 빨라지면서, 동일한 결과
  - Redundancy가 추가되더라도 같은 결과
- Latency vs Throughput
  - Latency: 하나의 action 혹은 결과가 나오는데 걸리는 시간
  - Throughput: 한번의 처리할 수 있는 action의 수
  - 허용할만한 Latency에 대해서 Throughput을 최대화하자.
- Availability vs consistency
  - CAP 이론
    - Consistency: 모든 읽기가, 가장 최근에 쓴 값을 반환
    - Availability: 모든 요청이 처리된다.
    - Partition Tolerance: 네트워크 실패로 임의 메시지 소실 혹은 실패에도 시스템이 작동
    - 분산 시스템이 적합한 이론
    - P는 항상 달성 불가능하므로, CP 혹은 AP만 존재한다.
  - PACELC 이론
    - 참고: [슬라이드](https://speakerdeck.com/sids/cap-theorem-you-dont-need-cp-you-dont-want-ap-and-you-cant-have-ca), [Blog](http://happinessoncode.com/2017/07/29/cap-theorem-and-pacelc-theorem/)
    - Partition Tolerance를 완벽하게 만족하는 시스템은 없다.
    - 그러므로 Partition 상황에 따라 우선순위 설덩 선택
    - Partition 상황 발생시: Availability vs Consistency
    - 정상 상황에서는: Latency vs Consistency
    - MongoDB: PA/EC, 설정에 따라 PA/EL
      - <https://dzone.com/articles/mongodb-consistency-levels-cappaclec-theorem>
    - Master-Slave Async MySQL: PA/EL
    - Master-Slave Semi-sync MySQL: PA/EC
    - Zookeeper: PC/EC
    - Kafka: PA/EL(acks=0), PA/EC(acks=-1, min.isr=1), PC/EC(acks=-1, min.isr=#replicas)

# Consistency/Availability Pattern 

- Consistency patterns
  - Weak consistency
    - 쓴 뒤에, 값이 읽히는것을 보장하지 않음, 최선은 다함
    - 전화, 방송 등이 그러함, 중간에 끊긴다고 해서, 끊어졌던 내용을 들을 수 없음
  - Eventual consistency
    - 약간 지연되지만, 결과적으로 쓴 값을 읽을 수 있음
  - Strong consistency
    - RDBMS나 파일시스템 처럼 Lock을 걸고 읽고 씀
- Availability patterns
  - Failover
    - Active-passive: 장애시, passive가 active의 IP를 인수하여 active 역할을 수행
    - Active-active: 여러개의 active를 두어서 부하를 분산시킴
    - 문제점
      - 하드웨어 복잡성
      - active에서 passive로 동기화하기전에 active가 죽으면 데이터 소실
  - Replication (DB)
    - 따로 DB에서 다룸

# DNS/CDN

- DNS
  - 도메인에 메핑되는 IP 주소를 얻어온다.
  - 자세한 과정
    - OS가 ISP의 DNS에 요청
    - ISP는 Root-DNS에 TLD(.com, .net 등)에 매핑되는 TLD-DNS의 IP를 요청
    - TLD-DNS에서는 NameServer의 IP를 알려줌
    - NameServer는 마침내 도메인의 IP 주소를 알려줌
    - 각 과정은 캐싱되어 단축될 수 있고, TTL로 관리됨
  - 하나의 도메인에 여러개의 IP개 매핑되어 로드를 분산시킬 수 있음
    - 가중치 라운드 로빈 분산
    - 지연시간 기반 분산
    - 지리 기반 분산
  - 단점: 지연시간(캐싱으로 경감), DNS관리 복잡성, DDoS 공격 받을 수 있음
- CDN
  - 문서, 그림등 콘텐츠를 캐시해놓는 서버, 지역마다 있어서 지연시간 단축, 트래픽 분산에 도움이 됨
  - Push CDN: 콘텐츠를 명시적으로 푸쉬해서 관리
  - Pull CDN: 첫 사용자의 요청시 캐시하고, TTL로 관리
  - 단점: Cost, TTL로 인한 배포지연, CDN으로 URL을 교체해주어야 함


# Load Balancer / Reverse Proxy

- Load Balancer
  - 여러 장점 있음
    - 비 정상적인 요청 막음
    - 리소스 과부하 방지
    - Single-Point-Falilure 방지
    - SSL 종료
    - Sticky Session
  - 분산방법: 랜덤, Min-Load, Session/Cookie, Round-Robin, L4, L7
  - L4: IP / Port를 기반으로 부하 분산
  - L7: 헤더, 쿠키, 메시지 등 Application 단의 정보까지 읽어서 부하 분산
  - Horizontal Scaling, 수평 확장
    - 장점: 싸다, 확장성이 쉽다, 할 수 있는 사람 찾기도 쉽다.
    - 단점: 복잡하다, Sticky Session, 캐시, DB의 동시 연결 처리 문제
  - LB의 단점: 병목이 될 수 있음, 복잡성 증가, SinglePointFailure가 될 수 있음
-  Reverse Proxy
   - 프록시의 반대, 하나의 인바운드, 여러개의 아웃바운드
   - 로드밸런서는 리버스 프록시의 일종
   - 장점: 보안강화, 확장성, 유연성, SSL종료, 압축, 캐싱, 로깅, 정적컨텐츠 직접제공
   - 단점: 복잡성 증가, Single Point Failure
  
# Application Layer

- Monolithic
  - 하나로 짜여져 있는 큰 어플리케이션
  - 장점: 통합 시나리오 테스트가 간편, 초기 개발 용이, 배포 용이
  - 단점: 업데이트, 자원관리, 새로운 기술적용, 확장이 어려움
- Microservice Architecture
  - 각 기능을 쪼개서 개발
  - 장점: 확장성, 새로운 서비스 개발이 상대적으로 쉬움
  - 단점: 성능, 디버깅 어려움, 개발 복잡도 증가
- Service Discovery
  - MSA로 구성할 때, LB가 각 서버를 알 수 있도록 IP정보를 배포
  - 복잡성이 추가됨
- 대체로 요새는 Docker 같은 Container를 이용해 MSA를 배포

# Database - RDBMS

- 스키마 필요함
- ACID(Atomicity, Consistency, Isolation, Durability) 트랜젝션
  - Atomicity: 각 트랜젝션의 원자성
  - Consistency: 트랜젝션은 항상 같은 전이를 보장
  - Isolation: 트랜젝션의 동시 실행은 직렬 실행과 같은 결과를 보장
  - Durability: 트랜젝션이 Commit된 이후에 철회되지 않음
- Replication
  - Master-slave: master는 읽기쓰기 가능, Slave에는 읽기만 가능
    - 장점 
      - 읽기 Throughput 증가
      - 마스터가 죽었을때, Slave를 Master로 Promote 가능
    - 단점
      - 자동 Promote안됨, 로직 구현 필요
      - 복제 지연 증가
      - Slave에서 읽기 시작하면, ACID 위반
      - Master의 실패시, Slave로 미처 복제되지 못한 데이터 생길 수 있음
  - Master-Master: 여러개의 Master 사용
    - 장점
      - 읽기, 쓰기 Throughput 증가
      - Master 사망시에도, 자동으로 다른 Master 사용
    - 단점
      - 로드밸런서나, 쓰기 위치를 지정하는 로직 필요
      - 쓰기 지연 증가 
      - ACID를 위반하거나 vs 쓰기 지연 증가
      - Master 개수가 늘면, 대기시간 증가, 충돌 해결 비용도 증가
- Federation
  - Vertical Partitioning to multiple database
  - 데이터베이스를 기능 단위로 분할(예 - 유저, 블로그, 제품)
  - 단점: 두 데이터베이스를 Join 해야할 때 성능 하락이 심함, 복잡해짐
- Sharding
  - Horizontal Partitioning to multiple database
  - 한 테이블 내 행들을 여러개의 DB로 분할
  - 장점: 캐시 적중 높이고, Throughput 높아짐, 인덱스 크기 감소, DB 사이즈 감소
  - 단점: 매우 복잡해짐, 샤드 재조정이 어려움, 여러 Shard Join 매우 어려움
- Denormalization
  - Foreign Key 등을 써서 쪼개놓았던 두 테이블을 Join 시킨 체로 저장함
  - 이러면 여러개의 중복 데이터가 생김, Join을 안해도 되므로 빠르지만, 일관성 보장이 어려움
  - 읽기 집중적일때 좋음, 쓰기 집중적일때 안좋음

# Concepts for Distributed System

### Paxos Alogirhtm

- 참고: [위키백과](https://ko.wikipedia.org/wiki/%ED%8C%A9%EC%86%8C%EC%8A%A4_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))
- Consensus(합의)를 거쳐서 결론에 도달하는 과정
- Paxos Basics
  - 3가지 역할: proposers, acceptors, learner
    - 각 노드는 여러개의 역할을 동시에 할 수 있음
    - 각 노드는 과반수가 되는 acceptor의 수를 알아야 한다.
    - 각 노드는 persistent해야한다. 각 노드는 한번 accept한걸 잊지 못함.
  - 한번 합의에 도달하면 바꾸지 못함.
- Progress
  - prepare: proposer가 acceptor들에게 N번 제안을 준비하라고 보냄
  - promise: acceptor는 proposer에게 N번 미만의 제안을 더이상 수락하지 않겠다고 약속함
    - 만약, 이전에 수락한 적이 있었다면, 수락했었던 결과를 함께 돌려보냄
    - 만약, N보다 큰 제안을 이미 받았었다면, Ignore를 보냄
  - accept!: proposer가 acceptor에게 제안하는 데이터를 보냄
  - accepted: acceptor는 proposer의 제안을 수락할지 말지 결정
    - 만약, 이전에 확정한 제안이 없다면, 이를 모든 learner에게 알림
    - 만약, 이전에 합의에 이른 제안이 있다면, 들어온 제안을 무시하고, 거절(Nack)을 proposer에게 보냄

### Consistent Hashing

- Hash 공간 내의 Hash들의 재조정을 최소한으로 하면서 

# Database - NoSQL

### Memcached

- Key-Value DB
- 메모리만 이용, 복구 안됨, 스트링만 저장, LRU
- 연결 리스트 기반 Hash Table
- Consistent Hashing 기반으로 여러개의 Memcached를 이용 가능 [(참고)](https://cloud.google.com/architecture/deploying-memcached-on-kubernetes-engine)
- 여러개의 Memcached를 이용하는건 가용성 및 대역폭을 위함
  - Replication은 아니고 Sharding 임
  - https://github.com/twitter/twemproxy

### Redis

- 참고: [우아한 Redis](https://ict-nroo.tistory.com/133), [Redis 기본개념](https://sjh836.tistory.com/178)
- Key-Value DB
- 메모리+디스크 기반, 디스크 저장되어 복구 가능
- Persistent Storage
  - RDB(Redist Database): 일정 간격으로 데이터 세트의 스냅샷 저장
    - (+) 포크 이후 저장하므로 성능 하락 없음
    - (+) AOF보림 복원 빠름
    - (-) 장애시 일정 데이터의 손실 있을 수 있음 (Durability 하락)
    - (-) 데이터 세트가 큰 경우 CPU 중단 시간이 길어질 수 있음
  - AOF(Append Only File): 서버가 수신한 모든 작업을 저장하여, 추후에 Replay해 복원
    - (+) 훨씬 안정적이고 내구성이 높아짐
    - (-) RDB보다 저장 파일 사이즈가 크고, 복구시에 느림
  - RDB+AOF: RDB로 저장이 완료된 이후의 값들을 AOF로 가지고 있음
    - 공식 사이트에 따르면, 둘다 쓰는게 권장됨
- 자료구조 지원(String, Set, Sorted Set, Hash, List)
  - 금새 느려지므로 사용에 주의, 1만개 이상은 넣지말기
- 싱글스레드
- 주된 활용처는 채팅, 메시지, 대기열, 세션, 미디어, 지리공간 등


# Distributed Cordinator

### Zookeeper

- ACID는 아니더라도, PC/EC 제공
- Sequential Consistency: 순서대로 적용됨을 보장
