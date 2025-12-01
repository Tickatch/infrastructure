# Tickatch Infrastructure

Tickatch MSA 프로젝트의 인프라 구성 파일입니다.

Docker Compose 기반으로 서비스 디스커버리, 설정 관리, 메시징, 모니터링, 로깅 등 MSA 운영에 필요한 인프라를 제공합니다.

---

## 아키텍처 개요

```
                                    ┌─────────────────────────────────────────────────────────────┐
                                    │                         NGINX                               │
                                    │                    (Reverse Proxy)                          │
                                    │                      :80, :443                              │
                                    └─────────────────────────────────────────────────────────────┘
                                                              │
                    ┌─────────────────────────────────────────┼─────────────────────────────────────────┐
                    │                                         │                                         │
                    ▼                                         ▼                                         ▼
    ┌───────────────────────────────┐     ┌───────────────────────────────┐     ┌───────────────────────────────┐
    │        Infrastructure         │     │        Microservices          │     │         Monitoring            │
    │                               │     │                               │     │                               │
    │  ┌─────────┐ ┌─────────┐     │     │  ┌─────────┐ ┌─────────┐     │     │  ┌─────────┐ ┌─────────┐     │
    │  │ Eureka1 │ │ Eureka2 │     │     │  │  Auth   │ │  User   │     │     │  │Prometheus│ │ Grafana │     │
    │  │  :3150  │ │  :3151  │     │     │  │ Service │ │ Service │     │     │  │  :9090  │ │  :3000  │     │
    │  └─────────┘ └─────────┘     │     │  └─────────┘ └─────────┘     │     │  └─────────┘ └─────────┘     │
    │                               │     │                               │     │                               │
    │  ┌─────────────────────┐     │     │  ┌─────────┐ ┌─────────┐     │     │  ┌─────────┐ ┌─────────┐     │
    │  │    Config Server    │     │     │  │ Product │ │ Reserv  │     │     │  │  Alert  │ │Pushgate │     │
    │  │       :3100         │     │     │  │ Service │ │ Service │     │     │  │ Manager │ │  way    │     │
    │  └─────────────────────┘     │     │  └─────────┘ └─────────┘     │     │  │  :9093  │ │  :9091  │     │
    │                               │     │                               │     │  └─────────┘ └─────────┘     │
    └───────────────────────────────┘     │  ┌─────────┐ ┌─────────┐     │     └───────────────────────────────┘
                                          │  │ Notifi- │ │ Notifi- │     │
                    ┌─────────────────────│  │ cation  │ │ Sender  │     │─────────────────────┐
                    │                     │  │ Service │ │ Service │     │                     │
                    ▼                     │  └─────────┘ └─────────┘     │                     ▼
    ┌───────────────────────────────┐     └───────────────────────────────┘     ┌───────────────────────────────┐
    │          Messaging            │                                           │           Logging             │
    │                               │                                           │                               │
    │  ┌─────────┐ ┌─────────┐     │                                           │  ┌─────────┐ ┌─────────┐     │
    │  │RabbitMQ │ │  Kafka  │     │                                           │  │  Elast  │ │ Logstash│     │
    │  │  :5672  │ │  :9094  │     │                                           │  │ icsearch│ │  :5000  │     │
    │  │ :15672  │ │         │     │                                           │  │  :9200  │ │         │     │
    │  └─────────┘ └─────────┘     │                                           │  └─────────┘ └─────────┘     │
    │                               │                                           │                               │
    └───────────────────────────────┘                                           │  ┌─────────┐ ┌─────────┐     │
                                                                                │  │ Kibana  │ │ Zipkin  │     │
                                                                                │  │  :5601  │ │  :9411  │     │
                                                                                │  └─────────┘ └─────────┘     │
                                                                                └───────────────────────────────┘
```

---

## 디렉토리 구조

```
infrastructure/
├── README.md
├── .env.example                    # 루트 환경변수 템플릿
├── .gitignore
├── docker-compose.yml              # 통합 Compose 파일
│
├── config/                         # 공통 설정 파일
│   └── logback-spring.xml
│
├── config-server/                  # Spring Cloud Config Server
│   ├── .env.example
│   └── docker-compose.yml
│
├── eureka-server/                  # Eureka Service Discovery (HA)
│   ├── .env.example
│   └── docker-compose.yml
│
├── gateway-server/                 # API Gateway (선택적)
│   ├── .env.example
│   └── docker-compose.yml
│
├── nginx/                          # Reverse Proxy
│   ├── nginx.conf
│   └── docker-compose.yml
│
├── rabbitmq/                       # Message Broker
│   └── docker-compose.yml
│
├── kafka/                          # Event Streaming
│   └── docker-compose.yml
│
├── elk/                            # ELK Stack (로깅)
│   ├── docker-compose.yml
│   └── logstash/
│       └── pipeline/
│           └── logstash.conf
│
├── zipkin/                         # Distributed Tracing
│   └── docker-compose.yml
│
├── monitoring/                     # Prometheus + Grafana
│   ├── .env.example
│   ├── docker-compose.yml
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── service-alerts.yml
│   ├── alertmanager/
│   │   └── alertmanager.yml
│   └── grafana/
│       └── provisioning/
│           └── datasources/
│               └── datasource.yml
│
├── notification-service/           # 알림 서비스
│   ├── .env.example
│   └── docker-compose.yml
│
├── notification-sender-service/    # 알림 발송 서비스
│   ├── .env.example
│   └── docker-compose.yml
│
└── data/                           # 데이터 볼륨 (gitignore)
    ├── elasticsearch/
    ├── grafana/
    ├── kafka/
    ├── prometheus/
    ├── rabbitmq/
    └── alertmanager/
```

---

## 빠른 시작

### 1. 사전 준비

```bash
# Docker 및 Docker Compose 설치 확인
docker --version
docker compose version

# 저장소 클론
git clone https://github.com/Tickatch/infrastructure.git
cd infrastructure
```

### 2. 환경변수 설정

```bash
# 각 서비스별 .env.example을 .env로 복사
cp .env.example .env
cp config-server/.env.example config-server/.env
cp eureka-server/.env.example eureka-server/.env
cp monitoring/.env.example monitoring/.env
cp notification-service/.env.example notification-service/.env
cp notification-sender-service/.env.example notification-sender-service/.env

# 각 .env 파일을 열어 실제 값으로 수정
```

### 3. 네트워크 및 서비스 시작

```bash
# Docker 네트워크 생성 (최초 1회)
docker network create tickatch-network

# 전체 서비스 시작
docker compose up -d

# 또는 개별 서비스 시작
docker compose up -d eureka-server-1 eureka-server-2
docker compose up -d config-server
docker compose up -d rabbitmq kafka
docker compose up -d elasticsearch logstash kibana
docker compose up -d prometheus grafana alertmanager
docker compose up -d nginx
```

### 4. 서비스 상태 확인

```bash
# 모든 컨테이너 상태 확인
docker compose ps

# 로그 확인
docker compose logs -f [서비스명]
```

---

## 서비스 상세

### 포트 매핑

| 서비스 | 내부 포트 | 외부 포트 | 설명 |
|--------|----------|----------|------|
| **Nginx** | 80, 443 | 80, 443 | Reverse Proxy |
| **Eureka 1** | 3150 | 3150 | Service Discovery (HA) |
| **Eureka 2** | 3151 | 3151 | Service Discovery (HA) |
| **Config Server** | 3100 | 3100 | 설정 서버 |
| **RabbitMQ** | 5672 | 9092 | Message Broker |
| **RabbitMQ Management** | 15672 | 15672 | 관리 콘솔 |
| **Kafka** | 9092 | 9094 | Event Streaming |
| **Elasticsearch** | 9200 | 9200 | 검색/로그 저장 |
| **Logstash** | 5000 | 5000 | 로그 수집 |
| **Kibana** | 5601 | 5601 | 로그 시각화 |
| **Zipkin** | 9411 | 9411 | 분산 추적 |
| **Prometheus** | 9090 | 9090 | 메트릭 수집 |
| **Pushgateway** | 9091 | 9091 | 메트릭 푸시 |
| **Grafana** | 3000 | 3000 | 모니터링 대시보드 |
| **Alertmanager** | 9093 | 9093 | 알림 관리 |

### 서비스 의존성

```
nginx
  └── eureka-server-1, eureka-server-2, config-server

config-server
  └── rabbitmq, eureka-server-1, eureka-server-2

logstash
  └── elasticsearch, kafka

kibana
  └── elasticsearch

grafana
  └── prometheus

notification-service
  └── kafka, rabbitmq, config-server, eureka

notification-sender-service
  └── kafka, rabbitmq, config-server, eureka
```

---

## 접속 URL

개발 환경 (localhost):

| 서비스 | URL |
|--------|-----|
| Eureka 1 | http://localhost:3150/eureka1 |
| Eureka 2 | http://localhost:3151/eureka2 |
| Config Server | http://localhost:3100 |
| RabbitMQ | http://localhost:15672 |
| Kibana | http://localhost:5601 |
| Zipkin | http://localhost:9411 |
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3000 |
| Alertmanager | http://localhost:9093 |

프로덕션 환경 (Nginx Proxy):

| 서비스 | URL |
|--------|-----|
| Eureka 1 | https://your-domain.com/eureka1 |
| Eureka 2 | https://your-domain.com/eureka2 |
| Config Server | https://your-domain.com/config |
| Zipkin | https://your-domain.com/zipkin |
| Kibana | https://your-domain.com/kibana |
| Prometheus | https://your-domain.com/prometheus |
| Grafana | https://your-domain.com/grafana |
| Alertmanager | https://your-domain.com/alertmanager |
| RabbitMQ | https://your-domain.com/rabbitmq |

---

## 환경변수 설정

### 민감 정보 관리

모든 `.env` 파일은 `.gitignore`에 포함되어 있습니다. 저장소에는 `.env.example` 파일만 포함됩니다.

```bash
# .gitignore
.env
**/.env
**/data/
```

### 주요 환경변수

**루트 (.env)**
```bash
# Slack Webhook URLs (Alertmanager용)
DEFAULT_SLACK=https://hooks.slack.com/services/xxx/xxx/xxx
CRITICAL_SLACK=https://hooks.slack.com/services/xxx/xxx/xxx
WARNING_SLACK=https://hooks.slack.com/services/xxx/xxx/xxx
```

**Config Server**
```bash
SPRING_CLOUD_CONFIG_SERVER_GIT_URI=https://github.com/your-org/config-repo
SPRING_CLOUD_CONFIG_SERVER_GIT_USERNAME=your-username
SPRING_CLOUD_CONFIG_SERVER_GIT_PASSWORD=your-token
```

**Notification Service**
```bash
# Database
SPRING_DATASOURCE_URL=jdbc:mysql://host:3306/notification
SPRING_DATASOURCE_USERNAME=username
SPRING_DATASOURCE_PASSWORD=password

# Messaging
SPRING_RABBITMQ_HOST=rabbitmq
SPRING_RABBITMQ_USERNAME=tickatch
SPRING_RABBITMQ_PASSWORD=tickatch123
```

---

## 운영 가이드

### 로그 확인

```bash
# 특정 서비스 로그
docker compose logs -f config-server

# 최근 100줄만
docker compose logs --tail=100 eureka-server-1

# 여러 서비스 동시에
docker compose logs -f eureka-server-1 eureka-server-2
```

### 서비스 재시작

```bash
# 특정 서비스만 재시작
docker compose restart config-server

# 설정 변경 후 재생성
docker compose up -d --force-recreate config-server
```

### 스케일 아웃

```bash
# 서비스 인스턴스 수 조정 (지원되는 서비스만)
docker compose up -d --scale notification-service=3
```

### 데이터 백업

```bash
# Elasticsearch 데이터 백업
docker run --rm -v tickatch_elasticsearch_data:/data -v $(pwd):/backup alpine tar cvf /backup/es-backup.tar /data

# Grafana 대시보드 백업
docker run --rm -v tickatch_grafana_data:/data -v $(pwd):/backup alpine tar cvf /backup/grafana-backup.tar /data
```

### 전체 초기화

```bash
# 모든 서비스 중지 및 볼륨 삭제
docker compose down -v

# 특정 데이터만 삭제
rm -rf data/elasticsearch/*
rm -rf data/kafka/*
```

---

## 문제 해결

### Eureka 서버 연결 실패

```bash
# Eureka 서버 상태 확인
curl http://localhost:3150/eureka1/actuator/health
curl http://localhost:3151/eureka2/actuator/health

# 네트워크 확인
docker network inspect tickatch-network
```

### Elasticsearch 시작 실패

```bash
# 메모리 설정 확인 (호스트)
sysctl vm.max_map_count

# 메모리 설정 변경 (Linux)
sudo sysctl -w vm.max_map_count=262144

# 영구 설정
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

### RabbitMQ 연결 실패

```bash
# RabbitMQ 상태 확인
docker exec tickatch-rabbitmq rabbitmq-diagnostics check_running

# 사용자 확인
docker exec tickatch-rabbitmq rabbitmqctl list_users
```

### 디스크 공간 부족

```bash
# Docker 리소스 정리
docker system prune -a

# 사용하지 않는 볼륨 삭제
docker volume prune

# 오래된 로그 삭제
find data/ -name "*.log" -mtime +7 -delete
```

---

## 보안 고려사항

1. **환경변수**: 모든 민감 정보는 `.env` 파일로 관리하고 절대 커밋하지 않음
2. **네트워크 격리**: `tickatch-network` 내부에서만 서비스 간 통신
3. **SSL/TLS**: Nginx에서 Let's Encrypt 인증서로 HTTPS 적용
4. **인증**: RabbitMQ, Grafana 등 관리 콘솔에 인증 적용
5. **방화벽**: 필요한 포트만 외부에 노출

