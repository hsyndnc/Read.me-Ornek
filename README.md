<h1 align="center">
<picture>
<img height="125" alt="Paywall Logo" src="./docs/images/logo.png">
</picture>




Paywall Backend Case Project


<a href="#">
<img src="https://img.shields.io/badge/.NET%208-512bd4?style=flat-square&logo=dotnet&logoColor=white">
</a>
<a href="#">
<img src="https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white">
</a>
<a href="#">
<img src="https://img.shields.io/badge/Redis-dc382d?style=flat-square&logo=redis&logoColor=white">
</a>
<a href="#">
<img src="https://img.shields.io/badge/Hangfire-522d82?style=flat-square">
</a>
<a href="#">
<img src="https://img.shields.io/badge/ElasticSearch-005571?style=flat-square&logo=elasticsearch&logoColor=white">
</a>
<a href="#">
<img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white">
</a>
</h1>

<p align="center">
<em><b>Paywall</b>, farklı merchant'ların tek bir merkezi altyapı üzerinden güvenli ve izlenebilir biçimde ödeme kabul etmesini sağlayan bir <b>Payment Orchestration</b> sistemidir. Clean Architecture ve CQRS pattern kullanılarak geliştirilmiş bu proje; <b>AuthApi</b> ve <b>PaymentApi</b> olmak üzere iki ana servisten oluşur.</em>
</p>


## 📑 Table of Contents

- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [API Endpoints](#-api-endpoints)
- [Architecture](#-architecture)
- [Authentication Flow](#-authentication-flow)
- [Payment Flows](#-payment-flows)
- [Technologies](#-technologies)
- [Docker Compose](#-docker-compose)
- [Test Scenarios](#-test-scenarios)

---

## ⚙️ Quick Start

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Git](https://git-scm.com/)

### 1. Clone Repository

```bash
git clone https://github.com/yelizozkan/paywall-payment-system.git
cd paywall-payment-system
```

### 2. Start Infrastructure (Docker)

```bash
docker-compose up -d postgres redis elasticsearch
```

### 3. Apply Database Migrations

```bash
cd src/Paywall.Payment/Paywall.Payment.Infrastructure
dotnet ef database update --startup-project ../Paywall.PaymentApi
```

### 4. Run Services

**Option A: Visual Studio**
- Open `Paywall.sln`
- Set Multiple Startup Projects (AuthApi + PaymentApi)
- Press F5

**Option B: Terminal**

```bash
# Terminal 1 - AuthApi
cd src/Paywall.AuthApi
dotnet run

# Terminal 2 - PaymentApi
cd src/Paywall.Payment/Paywall.PaymentApi
dotnet run
```

### 5. Access Swagger UI

| Service | URL |
|---------|-----|
| AuthApi | https://localhost:7218/swagger |
| PaymentApi | https://localhost:7027/swagger |
| Hangfire Dashboard | https://localhost:7027/hangfire |

---


## 🔌 API Endpoints

### AuthApi

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/auth/validate` | Validate API Key | Paywall-Api-Key |

### PaymentApi

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/payments` | Create Payment | Paywall-Api-Key |
| GET | `/api/payments/{id}` | Get Payment by ID | Paywall-Api-Key |
| GET | `/api/payments/by-tracking/{trackingCode}` | Get by TrackingCode (LINQ) | Paywall-Api-Key |
| GET | `/api/payments/by-external/{externalPaymentId}` | Get by ExternalPaymentId (Raw SQL) | Paywall-Api-Key |
| GET | `/api/payments` | List Payments (Paginated) | Paywall-Api-Key |
| PUT | `/api/payments/{id}/complete` | Complete Payment | Paywall-Api-Key |
| PUT | `/api/payments/{id}/cancel` | Cancel Payment | Paywall-Api-Key |




<br>



## 🏗 Architecture

### High-Level Architecture

```mermaid
flowchart TB
    Client[CLIENT<br/>Mobile App / Web App / Third-Party]

    PaymentAPI["Payment API<br/>.NET 8 - Clean Architecture"]
    AuthAPI["Auth API<br/>Stateless - API Key Validation"]

    Redis[(Redis<br/>Cache + Rate Limiting)]
    PostgreSQL[(PostgreSQL<br/>Payments Data)]
    Hangfire["Hangfire<br/>Background Jobs"]
    Elastic["ElasticSearch<br/>Request/Response Logging"]
    Callback[[External Callback<br/>HTTP POST]]

    Client -->|HTTPS + Paywall-Api-Key| PaymentAPI
    PaymentAPI -->|Validate API Key| AuthAPI
    PaymentAPI -->|Cache / Rate Limit| Redis
    PaymentAPI -->|CRUD Operations| PostgreSQL
    PaymentAPI -->|Enqueue Job| Hangfire
    Hangfire -.->|Async POST| Callback
    Hangfire -.->|Cancel Expired| PostgreSQL
    PaymentAPI -->|Request/Response Log| Elastic
```



### Clean Architecture Layers

```
┌─────────────────────────────────────────────────────────┐
│                    PaymentApi                           │
│              (Controllers, Middlewares)                 │
├─────────────────────────────────────────────────────────┤
│                    Application                          │
│         (CQRS Commands/Queries, DTOs, Validators)       │
├─────────────────────────────────────────────────────────┤
│                      Domain                             │
│            (Entities, Enums, Interfaces)                │
├─────────────────────────────────────────────────────────┤
│                   Infrastructure                        │
│    (EF Core, Redis, Hangfire, ElasticSearch, AuthApi)   │
└─────────────────────────────────────────────────────────┘
```


### Component Summary

| Component | Responsibility | Scaling Strategy |
|-----------|----------------|------------------|
| AuthApi | API Key validation | Stateless – Horizontal |
| PaymentApi | Payment operations | Horizontal |
| PostgreSQL | Transactional data | Read Replica |
| Redis | Cache + Rate Limit | Distributed |
| Hangfire | Background Jobs | Worker Scaling |
| ElasticSearch | Logging | Cluster |

---


<br>
<br>



## 🔐 Authentication Flow

```mermaid
sequenceDiagram
    participant Client
    participant PaymentApi
    participant AuthApi

    Client->>PaymentApi: Request + Paywall-Api-Key Header
    PaymentApi->>AuthApi: GET /api/auth/validate
    AuthApi-->>PaymentApi: 200 OK (MerchantId, MerchantName)
    PaymentApi-->>Client: Continue Request / 401 Unauthorized
```


<div align="center">
 
### 🔎 Authentication Steps

| Step | Action | Success | Failure |
|------|--------|---------|---------|
| 1 | Extract `Paywall-Api-Key` from Header | Continue | 401 (5002) |
| 2 | Validate via AuthApi | MerchantId loaded | 401 (5001) |
| 3 | Inject MerchantId into Context | Request proceeds | - |

---


<br>
<br>



## 💳 Payment Flows

### Payment Creation

```mermaid
sequenceDiagram
    participant Client
    participant PaymentApi
    participant Redis
    participant AuthApi
    participant PostgreSQL
    participant ElasticSearch

    Client->>PaymentApi: POST /api/payments
    PaymentApi->>Redis: Check Rate Limit
    Redis-->>PaymentApi: OK / 429 Too Many Requests

    PaymentApi->>AuthApi: Validate API Key
    AuthApi-->>PaymentApi: MerchantId / 401

    PaymentApi->>PostgreSQL: Check ExternalPaymentId
    PostgreSQL-->>PaymentApi: Not Found / 400 Conflict

    PaymentApi->>PostgreSQL: Insert Payment (Pending)
    PostgreSQL-->>PaymentApi: Created

    PaymentApi->>ElasticSearch: Log Request/Response

    PaymentApi-->>Client: 201 Created
```



### Payment State Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Completed : PUT /complete
    Pending --> Cancelled : PUT /cancel or Timeout (30min)
    Completed --> [*]
    Cancelled --> [*]
```



| From | To | Trigger |
|------|----|---------|
| Pending | Completed | Manual completion |
| Pending | Cancelled | Manual cancel or 30min timeout (Hangfire) |

---


<br>

 ### 2.Failure Scenarios

<div align="center">

### 🚨 Failure Cases

| Scenario | HTTP Status |
|----------|------------|
| Rate limit exceeded | 429 |
| Invalid API key | 401 |
| Duplicate ExternalPaymentId | 409 |

</div>

<br>
<br>



## 🐳 Docker Compose

### Services

| Service | Port | Description |
|---------|------|-------------|
| postgres | 5432 | PostgreSQL Database |
| redis | 6379 | Redis Cache |
| elasticsearch | 9200 | ElasticSearch Logging |
| auth-api | 5002 | Authentication Service |
| payment-api | 5001 | Payment Service |

### Commands

```bash
# Start all infrastructure
docker-compose up -d postgres redis elasticsearch

# Start everything (including APIs)
docker-compose up --build -d

# View logs
docker-compose logs -f payment-api

# Stop all
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

### Redis CLI

```bash
docker exec -it paywall-redis redis-cli
KEYS *
```

### ElasticSearch

```bash
# Check cluster health
curl http://localhost:9200

# View indices
curl http://localhost:9200/_cat/indices?v

# Search logs
curl http://localhost:9200/paywall-logs-*/_search?pretty
```

---

<br>


## 🧪 Test Scenarios

### ✅ Success Cases

| # | Test | Expected |
|---|------|----------|
| 1 | Valid API Key (AuthApi) | 200 + MerchantId |
| 2 | Create Payment | 201 Created |
| 3 | Get Payment by ID | 200 + Payment |
| 4 | Get by TrackingCode | 200 + Payment |
| 5 | Get by ExternalPaymentId | 200 + Payment |
| 6 | List Payments | 200 + Paginated List |
| 7 | Complete Payment | 200 + Status=Completed |
| 8 | Cancel Payment | 200 + Status=Cancelled |

### ❌ Error Cases

| # | Test | Expected |
|---|------|----------|
| 1 | Missing API Key | 401 (5002) |
| 2 | Invalid API Key | 401 (5001) |
| 3 | Duplicate ExternalPaymentId | 400 (1000) |
| 4 | Complete non-Pending | 400 |
| 5 | Cancel non-Pending | 400 |
| 6 | Payment Not Found | 404 |
| 7 | Rate Limit Exceeded | 429 |

### Test API Keys (Development)

| API Key | Merchant |
|---------|----------|
| `pk_test_merchant1_abc123xyz` | Test Merchant 1 |
| `pk_test_merchant2_def456uvw` | Test Merchant 2 |
| `pk_test_merchant3_inactive` | Test Merchant 3 |


---
<br>


## 📊 Middleware Pipeline

```
Request
    │
    ▼
┌─────────────────────────────┐
│ ExceptionHandlingMiddleware │  ← Global error handling
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│ RequestResponseLoggingMiddleware │  ← ElasticSearch logging
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│ ApiKeyAuthenticationMiddleware │  ← AuthApi validation
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│   RateLimitingMiddleware    │  ← Redis rate limiting
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│        Controller           │
└─────────────────────────────┘
```

---
<br>

## 🎯 Engineering Decisions

| Concern | Approach | Reason |
|---------|----------|--------|
| Idempotency | Unique constraint on ExternalPaymentId | Prevent duplicate payments |
| Scalability | Stateless AuthApi | Horizontal scaling |
| Performance | Redis cache for queries | Reduce DB load |
| Observability | ElasticSearch structured logging | Production debugging |
| Reliability | Hangfire with retry | Job resilience |
| Security | Middleware-level auth | Consistent authentication |

---

# 🏭 11. Production Enhancements
<div align="center">
    
| Area        | Improvement          |
| ----------- | -------------------- |
| Security    | API Gateway, mTLS    |
| Scalability | Kubernetes           |
| Reliability | Circuit Breaker      |
| Consistency | Outbox Pattern       |
| Monitoring  | Prometheus + Grafana |
</div>

---

## 🏭 Production Ortamı Değerlendirmeleri

<br>
<br>

### 🔐 Güvenlik (Security)

Production ortamında güvenlik, servisler arası iletişimden credential yönetimine kadar çok katmanlı olarak ele alınmalıdır.

<div align="center">

| Önlem | Açıklama | Amaç |
|-------|----------|------|
| API Gateway | Rate limiting, IP filtering, WAF | Dış saldırıları engellemek |
| mTLS | Servisler arası şifreli iletişim | Internal güvenliği artırmak |
| Secret Management | Vault / Secret Manager kullanımı | Credential güvenliği |
| API Key Rotation | Anahtarların periyodik yenilenmesi | Anahtar sızıntısı riskini azaltmak |
| Request Signature | Callback doğrulama | Sahte callback’i engellemek |


</div>

---

### 📈 Ölçeklenebilirlik (Scalability)

Sistem, artan trafik altında performans kaybı yaşamadan yatay olarak ölçeklenebilir şekilde tasarlanmalıdır.

<div align="center">


| Yaklaşım | Açıklama | Amaç |
|----------|----------|------|
| Horizontal Scaling | PaymentApi & AuthApi çoğaltılabilir | Trafik artışına dayanıklılık |
| Kubernetes | Container orchestration | Otomatik ölçekleme |
| Redis Cluster | Dağıtık cache | Yük altında performans |
| PostgreSQL Read Replica | Okuma yükünü dağıtmak | DB performansını artırmak |
| Worker Scaling | Hangfire worker sayısını artırmak | Arka plan işlerini hızlandırmak |

</div>

---

### 🧱 Dayanıklılık (Resilience)

Bağımlı servis hatalarında sistemin tamamen çökmesini engellemek için hata tolerans mekanizmaları uygulanmalıdır.

<div align="center">

| Mekanizma | Açıklama | Amaç |
|------------|----------|------|
| Circuit Breaker | Bağımlı servis arızasında devre kesme | Zincirleme hatayı önlemek |
| Retry Policy | Exponential backoff | Geçici hataları tolere etmek |
| Timeout Policy | Maksimum bekleme süresi | Sistem bloklanmasını önlemek |
| Health Checks | Servis sağlık kontrolleri | Otomatik restart / failover |

</div>

---

### 📊 Gözlemlenebilirlik (Observability)

Production ortamında hataların hızlı tespiti ve performans analizi için ölçülebilir ve izlenebilir bir yapı kurulmalıdır.

<div align="center">

| Bileşen | Açıklama | Amaç |
|----------|----------|------|
| Structured Logging | JSON format log | Kolay analiz |
| Centralized Logging | ElasticSearch cluster | Tek noktadan log takibi |
| Distributed Tracing | OpenTelemetry | Request izleme |
| Metrics | Prometheus | Performans ölçümü |
| Alerting | Grafana | Anlık hata bildirimi |

</div>

---

### 🧾 Veri Tutarlılığı (Data Consistency)

Ödeme gibi kritik domain’lerde veri tutarlılığı deterministik ve kontrollü state geçişleri ile sağlanmalıdır.

<div align="center">

| Strateji | Açıklama | Amaç |
|----------|----------|------|
| Outbox Pattern | Event güvenli publish | Event kaybını önlemek |
| Idempotent Endpoint | Aynı isteğin tekrarında güvenli işlem | Çift ödeme önleme |
| Optimistic Concurrency | Version kontrolü | Çakışma önleme |
| Transaction Boundary | Net transaction scope | Tutarlı veri yönetimi |

</div>


---

### ⚙️ Performans

Düşük gecikme süresi ve yüksek throughput için cache, indeks ve bağlantı optimizasyonları uygulanmalıdır.

<div align="center">

| Optimizasyon | Açıklama | Amaç |
|--------------|----------|------|
| Redis TTL | Cache süresi yönetimi | Gereksiz DB yükünü azaltmak |
| Index Optimization | ExternalPaymentId & TrackingCode index | Hızlı sorgu |
| Connection Pooling | DB bağlantı yönetimi | Resource verimliliği |
| Async I/O | Asenkron işlem | Yüksek throughput |

</div>


---

### 🚦 Rate Limiting & Abuse Prevention

Kötüye kullanım ve ani trafik artışlarına karşı sistem korunmalı ve adil kullanım sağlanmalıdır.

<div align="center">

| Tür | Açıklama | Amaç |
|-----|----------|------|
| Merchant Bazlı | Merchant başına limit | Adil kullanım |
| IP Bazlı | IP başına limit | Bot saldırılarını önlemek |
| Sliding Window | Zamana dayalı limit | Burst kontrolü |
| Global Limit | Sistem genel limiti | Stabilite |

</div>

---

### 🔄 CI/CD & Deployment

Deployment süreçleri otomatikleştirilerek kesintisiz ve güvenli sürüm geçişi sağlanmalıdır.

<div align="center">

| Uygulama | Açıklama | Amaç |
|----------|----------|------|
| Docker | Containerization | Taşınabilirlik |
| Blue-Green Deployment | Paralel release | Zero downtime |
| Rolling Updates | Kademeli geçiş | Servis kesintisini önlemek |
| Automated Migration | Migration kontrolü | Veri tutarlılığı |

</div>

---

### 📦 Disaster Recovery

Olası veri kaybı veya sistem arızalarında hızlı kurtarma için yedekleme ve failover stratejileri uygulanmalıdır.

<div align="center">

| Strateji | Açıklama | Amaç |
|----------|----------|------|
| Günlük Backup | Otomatik yedekleme | Veri kaybını azaltmak |
| Point-in-Time Recovery | Belirli zamana dönme | Hızlı kurtarma |
| Multi-Zone Deployment | Farklı availability zone | Yüksek erişilebilirlik |
| Failover | Otomatik yedek sisteme geçiş | Süreklilik |

</div>

---

# 🎯 13. Engineering Decisions Summary



Bu bölüm, mimari seçimlerin performans, ölçeklenebilirlik, güvenlik ve veri tutarlılığı açısından neden tercih edildiğini özetler. 
Alınan kararlar, minimal gereksinimlerin ötesinde production-ready bir sistem hedefiyle şekillendirilmiştir.

<div align="center">
    
| Concern       | Approach          |
| ------------- | ----------------- |
| Idempotency   | Unique constraint |
| Scalability   | Stateless auth    |
| Performance   | Redis cache       |
| Observability | ElasticSearch     |
| Reliability   | Hangfire retry    |

</div>

---

## 📝 Sonuç

Bu sistem, ödeme işlemlerinin güvenli, tutarlı ve ölçeklenebilir şekilde yönetilebilmesi amacıyla tasarlanmıştır. 
Minimal gereksinimlerin ötesinde production ortamı senaryoları düşünülmüş, 
observability ve resilience prensipleri uygulanmıştır.

Mimari tercihler, deterministik state yönetimi ve idempotent işlem garantisi üzerine kuruludur.
