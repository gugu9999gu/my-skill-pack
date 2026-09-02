---
name: dev-staging-production-environment-management
title: 개발·스테이징·라이브 서버 분리 및 운영 스킬
version: "1.0"
language: ko-KR
category:
  - DevOps
  - Infrastructure
  - Security
  - CI/CD
  - Database
  - Release Management
description: >
  웹/앱 서비스를 Local, Development/Preview, Staging, Production 환경으로 분리하고
  서버, 도메인, 데이터베이스, 스토리지, 환경변수, Secret, 접근권한, Git 브랜치,
  CI/CD, Migration, 배포, 롤백, 모니터링을 안전하게 관리하기 위한 운영 스킬.
---

# 개발·스테이징·라이브 서버 분리 및 운영 스킬

## 1. 이 스킬의 목적

이 스킬은 하나의 서비스를 개발할 때 **개발 환경과 실제 운영 환경이 서로 영향을 주지 않도록 격리**하고,
변경사항이 검증된 뒤에만 라이브 서버에 반영되도록 표준 운영 절차를 정의한다.

핵심 목표는 다음과 같다.

1. 개발 중 오류가 실제 사용자에게 영향을 주지 않게 한다.
2. 개발자가 실수로 운영 DB를 수정하거나 삭제하는 사고를 방지한다.
3. 운영용 API Key, DB Password, Secret을 개발자나 AI Agent가 직접 알 필요가 없게 한다.
4. 모든 변경사항을 Git과 CI/CD를 통해 추적 가능하게 만든다.
5. 운영 배포 전 실제 운영과 유사한 환경에서 검증한다.
6. 장애 발생 시 빠르게 이전 버전으로 롤백할 수 있게 한다.
7. 개발자, AI Agent, QA, 관리자마다 필요한 권한만 부여한다.
8. 서비스 규모가 커져도 동일한 구조를 확장할 수 있게 한다.

---

# 2. 기본 원칙

## 2.1 가장 중요한 원칙

> **Development와 Production은 서버 주소만 다른 것이 아니라, 데이터와 자격증명까지 분리되어야 한다.**

아래 구조는 잘못된 환경 분리다.

```text
dev.example.com
        │
        └──── Production DB

example.com
        │
        └──── Production DB
```

개발 서버에서 오류가 발생하면 실제 사용자 데이터가 영향을 받기 때문이다.

권장 구조:

```text
Development
├─ DEV App
├─ DEV API
├─ DEV DB
├─ DEV Storage
├─ DEV Redis
└─ DEV Secrets

Staging
├─ STAGING App
├─ STAGING API
├─ STAGING DB
├─ STAGING Storage
├─ STAGING Redis
└─ STAGING Secrets

Production
├─ PROD App
├─ PROD API
├─ PROD DB
├─ PROD Storage
├─ PROD Redis
└─ PROD Secrets
```

---

# 3. 권장 환경 구성

일반적인 웹 서비스는 다음 4개 환경을 기준으로 운영한다.

| 환경 | 주소 예시 | 목적 | 데이터 |
|---|---|---|---|
| Local | `localhost:3000` | 개인 개발 | 로컬/테스트 |
| DEV / Preview | `dev.example.com` | 기능 개발 및 통합 테스트 | DEV 전용 |
| STAGING | `staging.example.com` | 운영 직전 최종 검증 | STAGING 전용 |
| PROD | `example.com` | 실제 서비스 | 실제 운영 데이터 |

백엔드 API도 동일한 방식으로 분리한다.

```text
Local
http://localhost:8000

Development
https://api-dev.example.com

Staging
https://api-staging.example.com

Production
https://api.example.com
```

---

# 4. 각 환경의 역할

## 4.1 Local

개발자의 개인 PC에서 실행되는 환경.

사용 목적:

- 코드 작성
- 빠른 기능 테스트
- 단위 테스트
- UI/UX 수정
- API Mock 테스트
- DB Migration 초안 작성

예:

```text
Frontend
localhost:3000

Backend
localhost:8000

Database
localhost:5432

Redis
localhost:6379
```

### Local에서 금지할 것

```text
Production DB 직접 연결
Production Storage 접근
Production Secret 사용
Production 결제 API 실결제 호출
Production 이메일 대량 발송
Production Slack/Webhook 직접 사용
```

---

# 5. Development / Preview 환경

Development는 기능 개발과 통합 테스트를 위한 서버다.

예:

```text
dev.example.com
api-dev.example.com
```

또는 기능 브랜치별 Preview Server를 생성할 수도 있다.

```text
feature-login.preview.example.com
feature-dashboard.preview.example.com
pr-152.preview.example.com
```

## 주요 목적

- 개발 기능 통합
- 여러 개발자 간 테스트
- AI Agent가 개발한 결과 검증
- Frontend / Backend 연동 테스트
- 외부 API Sandbox 테스트
- 자동화 테스트
- QA 이전 테스트

## 접근 권한

일반 사용자에게 공개하지 않는다.

허용 대상 예:

```text
개발자
QA
관리자
CI/CD
승인된 AI Agent
Service Account
```

---

# 6. Staging 환경

Staging은 **Production과 최대한 비슷하게 구성하는 최종 검증 환경**이다.

예:

```text
staging.example.com
api-staging.example.com
```

Staging에서는 다음을 검증한다.

- Production Build
- DB Migration
- 인증
- 결제 Sandbox
- OAuth
- Storage
- CDN
- Cache
- Email
- Webhook
- Background Job
- Scheduler
- API Gateway
- 권한 정책
- 실제 브라우저 환경
- 모바일 환경
- 성능
- 배포 자동화

## Production과 차이

```text
Production
실제 사용자
실제 결제
실제 개인정보

Staging
테스트 사용자
Sandbox 결제
가상/마스킹 데이터
```

---

# 7. Production / Live 환경

실제 사용자가 사용하는 환경.

```text
https://example.com
https://api.example.com
```

Production은 가장 강한 권한 통제가 필요하다.

## 기본 원칙

- 사람의 직접 서버 수정 최소화
- SSH 직접 수정 최소화
- Production DB 직접 Query 최소화
- 모든 변경은 Git을 기준으로 수행
- 모든 배포 기록 보존
- Secret 접근 기록 보존
- DB Backup 활성화
- 장애 모니터링 활성화

---

# 8. 실 사용자가 DEV/STAGING에 접근하는 문제

도메인을 만들었다고 자동으로 보호되는 것은 아니다.

예:

```text
dev.example.com
staging.example.com
```

DNS가 인터넷에 공개되어 있고 별도 인증이 없다면 주소를 아는 사람은 접근할 수 있다.

따라서 **환경 분리와 접근 통제는 별개의 문제**로 다뤄야 한다.

---

# 9. 접근 통제 권장 구조

## Production

```text
example.com
        │
        ▼
Public Internet
        │
        ▼
Production
```

일반 사용자 접근 허용.

---

## Staging

```text
staging.example.com
        │
        ▼
Access Gateway
        │
        ▼
SSO Authentication
        │
        ▼
허가된 사용자
```

허가 대상:

```text
개발팀
기획팀
QA
관리자
외부 테스트 담당자
```

---

## Development

```text
dev.example.com
        │
        ▼
Private Access
        │
        ├─ VPN
        ├─ Zero Trust Access
        ├─ Tailscale
        └─ Service Token
```

---

# 10. Cloudflare 사용 시 권장 구조

Cloudflare를 사용하는 경우 다음 형태가 실용적이다.

```text
Internet
   │
   ▼
Cloudflare
   │
   ├─ example.com
   │     └─ Public
   │
   ├─ staging.example.com
   │     └─ Cloudflare Access
   │           └─ Google Workspace 로그인
   │
   └─ dev.example.com
         └─ Cloudflare Access
               ├─ 개발자
               ├─ CI/CD
               └─ Service Token
```

더 엄격하게 운영할 경우 DEV 자체를 VPN 내부로 제한한다.

```text
Developer
    │
    ▼
Tailscale / VPN
    │
    ▼
Private DEV Server
```

---

# 11. robots.txt와 noindex는 보안이 아니다

다음 설정은 검색엔진 노출 방지에는 사용할 수 있다.

```text
robots.txt
Disallow: /
```

```html
<meta name="robots" content="noindex,nofollow">
```

그러나 이것은 접근 제한 기능이 아니다.

사용자가 주소를 알면 접근 가능하다.

보안은 반드시 아래 중 하나로 구현한다.

```text
Authentication
Authorization
Zero Trust Access
VPN
Firewall
Private Network
IP Allowlist
mTLS
Service Token
```

---

# 12. Frontend만 막아서는 안 된다

다음 구조는 위험하다.

```text
staging.example.com
→ 인증 필요

api-staging.example.com
→ Public
```

사용자는 API 주소를 직접 호출할 수 있기 때문이다.

따라서 다음 모두 같은 접근 정책을 적용한다.

```text
staging.example.com
api-staging.example.com
admin-staging.example.com
files-staging.example.com
```

---

# 13. Database 환경 분리

가장 중요한 보안 영역 중 하나다.

권장 구조:

```text
DEV Database
STAGING Database
PRODUCTION Database
```

각각 별도의 DB Instance 또는 별도의 Project로 운영하는 것을 권장한다.

예:

```text
Supabase

myapp-dev
myapp-staging
myapp-production
```

또는:

```text
PostgreSQL

postgres-dev
postgres-staging
postgres-prod
```

---

# 14. 운영 데이터 복제 정책

Production 데이터 전체를 Development로 그대로 복사하지 않는다.

특히 다음 정보는 주의한다.

```text
이름
전화번호
주소
이메일
결제정보
주문정보
인증정보
개인 식별 정보
기업 비밀정보
```

필요하다면 마스킹한다.

예:

```text
Production

이효식
010-1234-5678
abc@example.com
```

개발용:

```text
테스트회원023
010-0000-0023
test023@example.test
```

---

# 15. Seed Data

Development와 Staging에서는 재현 가능한 Seed Data를 사용하는 것이 좋다.

예:

```text
/db
 ├─ migrations/
 └─ seed/
     ├─ users.sql
     ├─ products.sql
     └─ orders.sql
```

또는:

```bash
npm run db:seed
```

개발 환경을 언제든 초기화할 수 있어야 한다.

---

# 16. 환경변수 분리

코드에서는 같은 이름을 사용하고 값만 환경별로 변경한다.

예:

```text
DATABASE_URL
REDIS_URL
API_URL
SUPABASE_URL
SUPABASE_ANON_KEY
STORAGE_URL
JWT_SECRET
STRIPE_SECRET_KEY
```

Development:

```env
DATABASE_URL=postgres://dev...
API_URL=https://api-dev.example.com
```

Staging:

```env
DATABASE_URL=postgres://staging...
API_URL=https://api-staging.example.com
```

Production:

```env
DATABASE_URL=postgres://production...
API_URL=https://api.example.com
```

---

# 17. Secret 관리

Secret을 Git Repository에 저장하지 않는다.

금지:

```text
.env Production 파일 Git Commit
DB Password 코드 하드코딩
API Key 코드 하드코딩
SSH Private Key Repository 저장
```

사용 가능한 방식:

```text
GitHub Actions Secrets
GitHub Environments
Cloudflare Secrets
Supabase Secrets
AWS Secrets Manager
Google Secret Manager
HashiCorp Vault
Doppler
1Password Secrets Automation
```

---

# 18. Production Secret 접근 원칙

권장 구조:

```text
Developer
     │
     ├──── DEV Secret 접근 가능
     │
     └──── PROD Secret 접근 불가

CI/CD
     │
     └──── Production Deployment 과정에서만
           Production Secret 사용
```

이 구조에서는 개발자가 Production DB Password를 직접 알지 않아도 된다.

AI Agent 또한 마찬가지다.

---

# 19. AI Agent 권한 설계

AI Agent에게 다음 권한은 기본적으로 허용할 수 있다.

```text
DEV Repository 수정
Feature Branch 생성
DEV Database
Preview Server
Test 실행
Lint
Build
Migration 초안
Pull Request 생성
```

Production에서는 제한한다.

```text
Production Secret 열람
Production DB 직접 접근
Production 사용자 데이터 접근
Production Storage 삭제
Production 배포 직접 실행
```

권장 흐름:

```text
AI Worker
    │
    ▼
Feature Branch
    │
    ▼
Preview
    │
    ▼
AI Reviewer / Human Review
    │
    ▼
Staging
    │
    ▼
Approval
    │
    ▼
CI/CD
    │
    ▼
Production
```

---

# 20. Git 브랜치 전략

소규모/중규모 팀에서는 지나치게 복잡한 Git Flow가 필요하지 않다.

권장:

```text
main
│
├─ feature/login
├─ feature/dashboard
├─ feature/payment
├─ fix/order-error
└─ refactor/database
```

`main`은 항상 배포 가능한 상태를 유지한다.

---

# 21. Preview Deployment

각 Pull Request마다 별도 Preview 환경을 생성하면 좋다.

예:

```text
feature/login
        │
        ▼
pr-152.preview.example.com

feature/dashboard
        │
        ▼
pr-153.preview.example.com
```

장점:

- 기능 간 충돌 감소
- AI Agent 병렬 개발 가능
- QA가 특정 기능만 검증 가능
- Merge 전에 실제 브라우저 테스트 가능

---

# 22. 권장 배포 흐름

```text
Feature Branch
        │
        ▼
Commit
        │
        ▼
Pull Request
        │
        ▼
Preview Deployment
        │
        ▼
Automated Test
        │
        ▼
Code Review
        │
        ▼
Staging
        │
        ▼
QA
        │
        ▼
Approval
        │
        ▼
main Merge
        │
        ▼
Production Deployment
```

---

# 23. Production 배포 권한

Production은 `main` 브랜치에서만 배포하도록 제한하는 것이 좋다.

```text
feature/*
    ❌ Production

develop
    ❌ Production

staging
    ❌ Production

main
    ✅ Production
```

---

# 24. CI/CD 권장 단계

Production 배포 Pipeline 예:

```text
main Merge
    │
    ▼
Install Dependencies
    │
    ▼
Lint
    │
    ▼
Unit Test
    │
    ▼
Integration Test
    │
    ▼
Build
    │
    ▼
Security Check
    │
    ▼
Migration Validation
    │
    ▼
Production Approval
    │
    ▼
Backup
    │
    ▼
Database Migration
    │
    ▼
Deploy
    │
    ▼
Health Check
    │
    ▼
Smoke Test
    │
    ▼
Monitoring
```

---

# 25. Database Migration 관리

Production DB를 직접 수정하지 않는다.

잘못된 방식:

```sql
ALTER TABLE users ...
```

를 운영 DB Console에서 직접 수행.

권장 방식:

```text
/db/migrations/

001_initial.sql
002_add_users_phone.sql
003_create_orders.sql
004_add_order_index.sql
005_create_coupon_table.sql
```

---

# 26. Migration 흐름

```text
Migration 작성
      │
      ▼
Local 적용
      │
      ▼
Development 적용
      │
      ▼
Test
      │
      ▼
Staging 적용
      │
      ▼
QA
      │
      ▼
Production Backup
      │
      ▼
Production Migration
```

---

# 27. 위험한 Migration

다음 변경은 특별히 주의한다.

```sql
DROP TABLE
DROP COLUMN
ALTER COLUMN TYPE
DELETE
TRUNCATE
UPDATE 전체
UNIQUE Constraint 추가
NOT NULL Constraint 추가
대용량 Index 생성
```

가능하면 호환성을 유지하면서 단계적으로 변경한다.

예:

```text
1차 배포
새 컬럼 추가

2차 배포
새 컬럼 사용 시작

3차 배포
기존 컬럼 사용 중단

4차 배포
기존 컬럼 삭제
```

---

# 28. Zero-Downtime 배포 개념

가능하면 기존 버전을 즉시 종료하고 새 버전으로 교체하지 않는다.

권장:

```text
Old Version
     │
     ├──── Traffic
     │
New Version
```

Health Check 통과 후:

```text
Traffic
   │
   ▼
New Version
```

문제 발생 시:

```text
Traffic
   │
   ▼
Old Version
```

---

# 29. 롤백

배포 전 반드시 다음 질문에 답할 수 있어야 한다.

```text
문제가 발생하면 이전 코드로 돌아갈 수 있는가?
DB Migration도 되돌릴 수 있는가?
이전 Container/Image가 남아 있는가?
배포 전 DB Backup이 있는가?
```

최소한 최근 정상 Production Artifact를 보관한다.

예:

```text
app:v1.8.3
app:v1.8.4
app:v1.8.5-current
```

---

# 30. Health Check

배포 후 자동으로 확인해야 한다.

예:

```http
GET /health
```

응답:

```json
{
  "status": "ok"
}
```

추가 확인:

```text
Database Connection
Redis Connection
Storage Connection
Queue
External API
```

---

# 31. Smoke Test

배포 직후 최소 핵심 기능을 확인한다.

예:

```text
홈페이지 접속
로그인
회원정보 조회
상품 조회
주문 생성
API 요청
DB Write
Storage Upload
```

서비스 성격에 따라 핵심 경로를 정의한다.

---

# 32. Monitoring

Production에서는 최소 다음을 수집한다.

```text
CPU
Memory
Disk
Network
HTTP Error Rate
API Latency
Database Connection
Database CPU
Database Storage
Redis Memory
Queue
Background Job Failure
Frontend Error
```

---

# 33. 로그 환경 표시

모든 로그에 Environment를 기록하면 좋다.

예:

```json
{
  "environment": "production",
  "service": "order-api",
  "level": "error",
  "message": "payment failed"
}
```

DEV 오류와 Production 오류가 섞이지 않게 한다.

---

# 34. 환경별 외부 서비스 분리

가능하면 외부 서비스도 환경별로 분리한다.

예:

```text
Payment

DEV
Sandbox Key

STAGING
Sandbox Key

PROD
Live Key
```

메일:

```text
DEV
Mail Sandbox

STAGING
테스트 수신자 제한

PROD
실제 발송
```

---

# 35. Webhook 분리

Webhook URL도 환경별로 구분한다.

```text
https://api-dev.example.com/webhooks/payment

https://api-staging.example.com/webhooks/payment

https://api.example.com/webhooks/payment
```

Production Webhook이 DEV로 전달되지 않도록 한다.

---

# 36. Scheduler / Cron 분리

의외로 사고가 많이 발생하는 영역이다.

DEV Scheduler가 실제 고객에게 영향을 주지 않게 한다.

예:

```text
DEV
쿠폰 자동지급 → Test User만

STAGING
쿠폰 자동지급 → Test User만

PROD
쿠폰 자동지급 → 실제 사용자
```

---

# 37. Object Storage 분리

Storage Bucket도 분리한다.

예:

```text
myapp-dev
myapp-staging
myapp-prod
```

Production 이미지나 사용자 파일을 Development에서 삭제할 수 없어야 한다.

---

# 38. Redis / Cache 분리

Redis도 환경별로 분리하는 것이 가장 안전하다.

최소한 Namespace를 나눈다.

```text
dev:user:123
staging:user:123
prod:user:123
```

가능하면 Instance 자체를 분리한다.

Production에서 다음 명령은 강하게 제한한다.

```text
FLUSHALL
FLUSHDB
```

---

# 39. Queue 분리

Background Worker가 있다면 Queue도 분리한다.

```text
dev-orders
staging-orders
prod-orders
```

DEV Worker가 Production Job을 처리하면 안 된다.

---

# 40. DNS 권장 예

```text
example.com
www.example.com

api.example.com

dev.example.com
api-dev.example.com

staging.example.com
api-staging.example.com
```

Preview:

```text
*.preview.example.com
```

---

# 41. 관리자 페이지

관리자 페이지도 환경별로 분리한다.

```text
admin.example.com

admin-dev.example.com

admin-staging.example.com
```

Production Admin은 일반 Production보다 더 강하게 보호하는 것을 권장한다.

예:

```text
Cloudflare Access
+
Application Login
+
MFA
```

---

# 42. Production 변경 금지 항목

다음 행동은 특별한 승인 없이 수행하지 않는다.

```text
Production DB DROP
Production DB TRUNCATE
Production Storage 전체 삭제
Production Redis FLUSHALL
Production Secret 변경
Production DNS 변경
Production Firewall 변경
Production OAuth Redirect 변경
Production Scheduler 대량 작업
Production User Data Migration
```

---

# 43. 권한 계층 예시

## Developer

```text
Local       Full
DEV         Full
Staging     Read/Deploy 제한
Production  Read 제한
```

## QA

```text
Local       None
DEV         Optional
Staging     Full Test
Production  Normal User 수준
```

## AI Agent

```text
Local       Optional
DEV         제한된 Full
Staging     필요 시 Read/Test
Production  기본 접근 금지
```

## CI/CD Service Account

```text
DEV         Deploy
Staging     Deploy
Production  Deploy
```

단, Production Secret은 배포 과정에서만 사용할 수 있게 한다.

---

# 44. 추천 최종 아키텍처

```text
                    GitHub
                       │
              ┌────────┴────────┐
              │                 │
        Feature Branch         main
              │                 │
              ▼                 │
       Preview / DEV            │
              │                 │
          Automated Test        │
              │                 │
              ▼                 │
          Pull Request           │
              │                 │
              ▼                 │
           STAGING               │
              │                 │
            QA/Test              │
              │                 │
           Approval──────────────┘
                                  │
                                  ▼
                             PRODUCTION
```

Infrastructure:

```text
Cloudflare
├─ DEV
├─ STAGING
└─ PRODUCTION

Database
├─ DEV
├─ STAGING
└─ PRODUCTION

Storage
├─ DEV
├─ STAGING
└─ PRODUCTION

Redis
├─ DEV
├─ STAGING
└─ PRODUCTION

Secrets
├─ DEV
├─ STAGING
└─ PRODUCTION
```

---

# 45. 최소 구성

서비스 초기에는 비용 때문에 3개 서버를 완전히 별도로 운영하기 어려울 수 있다.

그 경우 최소한 다음은 반드시 분리한다.

```text
DEV
├─ Application
├─ Database
└─ Secrets

PRODUCTION
├─ Application
├─ Database
└─ Secrets
```

Staging은 이후 추가한다.

그러나 **Development와 Production Database 공유는 하지 않는 것을 기본 원칙으로 한다.**

---

# 46. 권장 구성

서비스가 실제 사용자를 보유한다면:

```text
Local
DEV / Preview
STAGING
PRODUCTION
```

4단계를 권장한다.

---

# 47. AI 기반 개발 조직에 특히 권장되는 구조

AI Worker를 여러 개 사용하는 경우:

```text
AI Worker A
feature/login
      │
      ▼
Preview A

AI Worker B
feature/dashboard
      │
      ▼
Preview B

AI Worker C
feature/order-api
      │
      ▼
Preview C
```

검수:

```text
Preview
   │
   ▼
Reviewer AI
   │
   ▼
Automated Tests
   │
   ▼
Human / Controller
   │
   ▼
Staging
```

Production은 CI/CD가 담당한다.

```text
AI
❌ Production Password

AI
❌ Production Secret

AI
❌ Production Direct Deployment

CI/CD
✅ Production Deployment
```

---

# 48. 운영 사고 예방 체크리스트

## Environment

- [ ] Local / DEV / STAGING / PROD가 명확히 구분되어 있는가?
- [ ] 각 환경 이름이 화면에서 식별 가능한가?
- [ ] DEV가 Production DB를 바라보지 않는가?
- [ ] Staging이 Production DB를 바라보지 않는가?

## Database

- [ ] DEV DB가 별도로 존재하는가?
- [ ] STAGING DB가 별도로 존재하는가?
- [ ] Production Backup이 활성화되어 있는가?
- [ ] Migration이 Git으로 관리되는가?

## Secrets

- [ ] Production Secret이 Git에 없는가?
- [ ] Production Secret을 AI Agent가 열람할 수 없는가?
- [ ] DEV와 Production API Key가 다른가?

## Access

- [ ] DEV가 Public으로 열려 있지 않은가?
- [ ] STAGING에 인증이 걸려 있는가?
- [ ] API 서버에도 동일한 접근 정책이 걸려 있는가?
- [ ] 관리자 페이지에 MFA가 적용되어 있는가?

## Deployment

- [ ] Production 배포는 main만 가능한가?
- [ ] 자동 테스트를 통과해야 배포 가능한가?
- [ ] Production 배포 승인 절차가 있는가?
- [ ] 배포 전 Backup 또는 Rollback 수단이 있는가?

## Monitoring

- [ ] Production Error Monitoring이 있는가?
- [ ] Health Check가 있는가?
- [ ] 로그에서 Environment를 구분할 수 있는가?
- [ ] 장애 알림이 있는가?

---

# 49. AI Agent 실행 규칙

AI Agent가 이 스킬을 사용할 때 다음 규칙을 따른다.

## 작업 시작 전

1. 현재 환경을 확인한다.
2. Production 여부를 확인한다.
3. 사용 중인 DB를 확인한다.
4. 변경 대상 Secret을 확인한다.
5. Migration 발생 여부를 확인한다.
6. 사용자 데이터에 영향이 있는지 확인한다.

## Production 관련 작업

Production 변경이 포함된 경우:

```text
직접 실행보다
→ 변경안 작성
→ 테스트
→ Staging 검증
→ 승인
→ CI/CD 실행
```

순서를 우선한다.

---

# 50. AI Agent가 중단해야 하는 조건

다음 상황에서는 임의 실행하지 않는다.

```text
현재 DB가 DEV인지 Production인지 구분 불가

Production Secret을 코드에 입력해야 함

DROP / TRUNCATE 명령이 포함됨

Production 사용자 데이터를 대량 수정함

Rollback 방법이 없음

Backup 여부를 확인할 수 없음

Production과 DEV가 동일한 Database URL 사용

환경 변수 출처를 확인할 수 없음
```

---

# 51. 운영 환경 표시 UX

개발자가 실수하지 않도록 Admin/UI에 현재 Environment를 명확하게 표시하는 것이 좋다.

예:

```text
[ DEVELOPMENT ]

[ STAGING ]

[ PRODUCTION ]
```

Production에서는 추가 경고를 표시할 수 있다.

```text
⚠ PRODUCTION ENVIRONMENT
```

Production에서 위험 작업을 실행할 때 확인 단계를 추가한다.

예:

```text
삭제하려면
PRODUCTION
을 입력하십시오.
```

---

# 52. 권장 운영 정책 요약

반드시 지킬 핵심 규칙:

1. Production DB와 Development DB를 공유하지 않는다.
2. Production Secret을 개발자/AI에 직접 제공하지 않는다.
3. Production 배포는 `main`을 기준으로 한다.
4. Feature Branch마다 Preview 환경을 사용할 수 있게 한다.
5. DB 변경은 Migration으로 관리한다.
6. Production 전 Staging 검증을 수행한다.
7. Production DB는 자동 백업한다.
8. Production 배포에는 승인 절차를 둔다.
9. DEV/STAGING에는 Access Control을 적용한다.
10. Frontend뿐 아니라 API/Admin/Storage도 함께 보호한다.
11. Production Scheduler와 DEV Scheduler를 분리한다.
12. 장애 시 즉시 롤백 가능한 상태를 유지한다.

---

# 53. 빠른 의사결정 규칙

## "이 서버를 외부에 공개해도 되는가?"

```text
Production
→ 예

Staging
→ 인증된 사용자에게만

Development
→ 원칙적으로 비공개
```

## "이 DB를 공유해도 되는가?"

```text
DEV ↔ PROD
→ 아니오

STAGING ↔ PROD
→ 아니오
```

## "Production Secret을 AI에 전달해도 되는가?"

```text
기본값
→ 아니오

대신
→ CI/CD 또는 Secret Broker 사용
```

## "운영 DB를 직접 수정해도 되는가?"

```text
기본값
→ 아니오

대신
→ Migration 작성 후 검증 및 배포
```

---

# 54. 이상적인 최종 상태

```text
                          USER
                           │
                           ▼
                    example.com
                           │
                           ▼
                      PRODUCTION
                      ├─ PROD API
                      ├─ PROD DB
                      ├─ PROD Storage
                      └─ PROD Secrets


Developer / QA
      │
      ▼
Cloudflare Access / VPN
      │
      ├───────────────┐
      ▼               ▼
dev.example.com   staging.example.com
      │               │
      ▼               ▼
    DEV            STAGING
    DB               DB


AI Agent
   │
   ▼
Feature Branch
   │
   ▼
Preview Environment
   │
   ▼
Automated Test
   │
   ▼
Review
   │
   ▼
Staging
   │
   ▼
Approval
   │
   ▼
CI/CD
   │
   ▼
Production
```

이 구조의 핵심은 단순하다.

> **개발자는 Development에서 자유롭게 작업하고, Production은 자동화된 배포 시스템을 통해서만 변경한다.**

그리고:

> **Production의 데이터와 Secret은 개발 환경으로 절대 흘러가지 않게 한다.**

이 두 원칙을 유지하면 서비스와 개발 인력이 늘어나더라도 안정적인 운영 구조를 유지할 수 있다.
