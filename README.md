# Redis Cluster Replica Failover Test Project

> Ubuntu + Docker 환경에서 Redis Cluster와 Replica 구조를 구성하고,
> 장애 상황(Failover)을 직접 재현하면서 복구 과정을 관찰하기 위한 실습 프로젝트입니다.

## 📌 프로젝트 목표

이 프로젝트는 단순히 Redis를 실행해보는 수준이 아니라 다음 내용을 직접 실험하고 검증하는 데 목적이 있습니다.

* Redis Cluster + Replica 구조 이해
* Master 노드 장애 발생 시 Replica 승격(Failover) 과정 관찰
* 장애 복구 실패 케이스와 성공 케이스 비교 실험
* Docker 기반 분산 환경 구성 경험
* Spring Boot 애플리케이션과 Redis 연동 복습
* 운영 환경에서 발생할 수 있는 Redis 장애 대응 흐름 학습

---

# 🧱 기술 스택

## Infra

* Ubuntu
* Docker
* Docker Compose
* Redis Cluster

## Backend

* Java 17
* Spring Boot
* Spring Data Redis
* Lettuce Client

## Script

* Bash Shell Script

---

# 📂 프로젝트 구조 예시

```text
redis-cluster-failover-lab/
├── docker/
│   ├── docker-compose.yml
│   ├── redis-node-1/
│   ├── redis-node-2/
│   ├── redis-node-3/
│   ├── redis-node-4/
│   ├── redis-node-5/
│   └── redis-node-6/
│
├── scripts/
│   ├── start-cluster.sh
│   ├── create-cluster.sh
│   ├── stop-master.sh
│   ├── recover-node.sh
│   ├── flush-all.sh
│   └── monitor-cluster.sh
│
├── spring-app/
│   ├── src/
│   └── Dockerfile
│
└── README.md
```

---

# 🧠 실습 시나리오

## 1. Redis Cluster 구성

Redis 6개 노드를 Docker로 실행합니다.

예시 구성:

| Node    | Role    |
| ------- | ------- |
| redis-1 | Master  |
| redis-2 | Master  |
| redis-3 | Master  |
| redis-4 | Replica |
| redis-5 | Replica |
| redis-6 | Replica |

각 Replica는 Master를 복제하도록 설정합니다.

---

# 🚀 실행 방법

## 1. Redis 노드 실행

```bash
./scripts/start-cluster.sh
```

---

## 2. Redis Cluster 생성

```bash
./scripts/create-cluster.sh
```

---

## 3. Cluster 상태 확인

```bash
./scripts/monitor-cluster.sh
```

또는:

```bash
docker exec -it redis-1 redis-cli cluster nodes
```

---

# 🔥 장애 상황 실험

## Master 강제 종료

예시:

```bash
./scripts/stop-master.sh
```

또는:

```bash
docker stop redis-1
```

---

# 🔍 관찰 포인트

장애 발생 후 다음 내용을 관찰합니다.

* Replica 노드가 새로운 Master로 승격되는가?
* Cluster 상태가 `fail` 에서 `ok` 로 회복되는가?
* Spring Boot 애플리케이션이 자동으로 새로운 Master를 인식하는가?
* 클라이언트 요청이 정상 처리되는가?
* 데이터 유실 여부는 어떻게 되는가?

---

# ❌ 실패 케이스 실험

대조군 실험도 함께 진행합니다.

예시:

* Replica 개수를 부족하게 설정
* 네트워크 분리
* quorum 부족 상황 유도
* 일부 노드만 실행
* Cluster 생성 없이 단일 Redis만 사용

이를 통해 다음을 비교합니다.

| 상황         | 결과             |
| ---------- | -------------- |
| Replica 존재 | 자동 Failover 성공 |
| Replica 없음 | 서비스 장애 발생      |
| Quorum 부족  | Failover 실패    |
| 네트워크 분리    | Split Brain 위험 |

---

# 🌱 Spring Boot 연동

Redis 장애 상황에서 실제 애플리케이션이 어떻게 동작하는지 확인하기 위해 간단한 Spring Boot 서버를 함께 구성합니다.

예시 기능:

* 캐시 저장 API
* 조회 API
* Redis 연결 상태 확인 API

---

## application.yml 예시

```yaml
spring:
  data:
    redis:
      cluster:
        nodes:
          - redis-1:7001
          - redis-2:7002
          - redis-3:7003
```

---

# 🧪 테스트 예시

## 데이터 저장

```bash
curl -X POST localhost:8080/cache/test
```

## 데이터 조회

```bash
curl localhost:8080/cache/test
```

장애 상황에서도 정상 동작 여부를 확인합니다.

---

# 📖 학습 포인트

이 프로젝트를 통해 다음 내용을 실습 중심으로 학습할 수 있습니다.

* Redis Cluster 내부 구조
* Gossip Protocol 개념
* Hash Slot 기반 분산 저장
* Replica Sync 동작
* Sentinel vs Cluster 차이
* 장애 복구(Failover) 메커니즘
* CAP 이론과 분산 시스템 특성
* Spring Data Redis Cluster 설정
* Docker 기반 멀티 노드 실습 환경 구성

---

# 🛠 주요 Redis 명령어

## Cluster 상태 확인

```bash
redis-cli cluster info
```

## 노드 목록 조회

```bash
redis-cli cluster nodes
```

## Replication 상태 확인

```bash
redis-cli info replication
```

---

# 📷 추천 추가 실험

추가적으로 다음 실험도 진행해볼 수 있습니다.

* Redis Sentinel 기반 HA 비교
* Redis Persistence(RDB/AOF) 실험
* Docker Network 장애 시뮬레이션
* Spring Retry 적용
* Redis Pub/Sub 테스트
* Kafka와 조합한 이벤트 캐싱 구조 실험

---

# 🎯 프로젝트 의의

실무에서는 Redis를 단순 캐시 서버가 아니라 고가용성(HA)이 중요한 핵심 인프라로 사용하는 경우가 많습니다.

이 프로젝트는 단순 CRUD 수준을 넘어:

* "장애가 발생했을 때 시스템이 어떻게 복구되는가?"
* "분산 시스템은 왜 복잡한가?"
* "애플리케이션은 장애 상황을 어떻게 처리해야 하는가?"

를 직접 체험하고 이해하는 데 초점을 맞춘 실습 프로젝트입니다.
