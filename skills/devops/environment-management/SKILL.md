---
name: environment-management
title: Development / Staging / Production Environment Management
version: "1.2"
language: ko-KR
category: devops
description: >
  웹/앱 서비스의 Local, Development/Preview, Staging, Production 환경을 분리하고
  접근권한, 데이터베이스, 스토리지, 시크릿, Git, CI/CD, Migration, 배포 및 롤백을
  안전하게 관리하기 위한 범용 DevOps 스킬.
---

# Environment Management Skill

## 언제 이 스킬을 사용하는가

다음 작업이 포함되면 이 스킬을 적용한다.

- 개발 서버 / 라이브 서버 분리
- Local / DEV / Preview / Staging / Production 구성
- 로컬 개발환경과 원격 검증환경 설계
- 배포 파이프라인 또는 CI/CD 설계
- 환경변수 및 Secret 관리
- DB, Storage, Redis, Queue 환경 분리
- Database Migration
- Production 접근권한 또는 배포권한 설계
- Preview Deployment
- Production Rollback / Backup / Health Check
- AI Agent의 개발 및 운영 서버 권한 설계

## 핵심 불변 규칙

1. **DEV와 PROD는 데이터베이스를 공유하지 않는다.**
2. **DEV/STAGING 코드가 PROD 자격증명을 참조하지 않게 한다.**
3. **Production Secret은 코드, Git, 일반 개발환경에 저장하지 않는다.**
4. **Production 변경은 가능하면 CI/CD와 승인 경로를 통해 수행한다.**
5. **DB Schema 변경은 Migration으로 버전 관리한다.**
6. **Production 반영 전에 Preview 또는 Staging에서 검증한다.**
7. **Production 배포에는 Backup/Rollback 경로가 존재해야 한다.**
8. **DEV/STAGING의 웹, API, Admin 등 모든 진입점에 적절한 접근제어를 적용한다.**
9. `robots.txt`, `noindex`, 난해한 URL은 접근제어 수단으로 취급하지 않는다.
10. AI Agent에는 기본적으로 Production Secret 및 Production DB 직접 접근권한을 주지 않는다.
11. **환경(Environment)은 반드시 별도의 상시 서버를 의미하지 않는다.** Local DEV는 개발자 PC, Preview는 임시 원격 환경일 수 있다.
12. **Staging은 운영 직전 검증 환경이므로 개인 개발 PC에만 종속시키지 않는다.** 실제 DNS/HTTPS/OAuth/Webhook/네트워크/배포 조건을 확인할 수 있는 독립된 원격 환경을 우선한다.

## 환경과 서버를 구분한다

`Local`, `DEV`, `Preview`, `Staging`, `Production`은 논리적 **환경(Environment)** 이다. 각 환경마다 반드시 상시 서버 한 대가 필요한 것은 아니다.

예를 들어 1인 또는 소규모 프로젝트에서는 다음 구성이 충분할 수 있다.

```text
Local Development
      │
      ▼
Preview (선택, PR별 임시)
      │
      ▼
Remote Staging
      │
      ▼
Production
```

즉 상시 `dev.example.com`을 별도로 유지하지 않고 Local 환경이 일상적인 DEV 역할을 담당할 수 있다.

## 기본 권장 환경 모델

| Environment | 권장 위치 | 목적 | 외부 사용자 접근 | 데이터 | 상시 필요 여부 |
|---|---|---|---|---|---|
| Local DEV | 개발자 PC | 일상 개발/단위 테스트 | 불가 | 로컬/테스트 | 예 |
| Remote DEV | Cloud/사내 서버 | 팀 통합 개발 | 제한 | DEV 전용 | 선택 |
| Preview | 원격 임시 환경 | PR/기능별 실제 배포 검증 | 제한 | Preview 전용 | 아니오 |
| Staging | **독립 원격 환경** | 운영 직전 최종 검증 | 인증된 QA/내부 사용자 | STAGING 전용 | 운영 서비스라면 권장 |
| Production | **독립 원격 환경** | 실제 서비스 | 허용 | 실제 운영 데이터 | 예 |

### 소규모 프로젝트 기본값

별도 Remote DEV 서버가 필요한 이유가 없다면 다음 구성을 기본값으로 우선 검토한다.

```text
LOCAL
├─ Frontend
├─ Backend
├─ Local DB
└─ Test Data
      │
      ▼
PREVIEW (선택)
PR/Feature별 임시 환경
      │
      ▼
STAGING
원격 / 운영과 유사한 환경
      │
      ▼
PRODUCTION
실제 서비스
```

### Remote DEV 서버가 필요한 경우

다음 조건 중 하나 이상이면 Remote DEV를 추가한다.

- 여러 개발자가 하나의 통합 개발환경을 공유해야 함
- 여러 AI Agent가 병렬 작업하고 결과를 원격에서 통합해야 함
- 외부 SaaS/Webhook이 localhost를 호출할 수 없음
- 모바일/외부 기기에서 지속적으로 DEV 환경을 테스트해야 함
- 팀 QA가 Staging 이전의 기능을 상시 확인해야 함

그렇지 않다면 Local DEV + Preview로 Remote DEV를 대체할 수 있다.

## 왜 Staging은 원격을 우선하는가

Staging의 역할은 단순히 코드 실행 여부를 확인하는 것이 아니라 **Production 배포 조건을 재현하는 것**이다.

Staging에서 확인해야 할 대표 항목:

```text
실제 DNS
HTTPS / TLS
Reverse Proxy / CDN
WAF / Access Gateway
Cookie Domain / Secure / SameSite
CORS
OAuth Redirect URL
외부 Webhook 수신
결제 Sandbox Callback
Email Provider
Storage / CDN
Scheduler / Background Job
CI/CD Deployment
Production Build
환경별 Secret 주입
```

따라서 개인 PC의 `localhost`만으로는 Staging의 목적을 충분히 달성하기 어렵다.

단, 회사 내부의 항상 켜져 있는 서버나 사내 Kubernetes/VM이 위 조건을 재현할 수 있다면 물리적으로 사내에 있어도 Staging으로 사용할 수 있다. 핵심은 **Cloud 여부가 아니라 개인 개발 PC에 종속되지 않고 운영 조건을 재현할 수 있는가**이다.

## 권장 주소 예시

Remote DEV를 사용하는 프로젝트:

```text
localhost:3000

dev.example.com
staging.example.com
example.com

api-dev.example.com
api-staging.example.com
api.example.com
```

Local DEV 중심 프로젝트:

```text
localhost:3000
localhost:8000

staging.example.com
api-staging.example.com

example.com
api.example.com
```

Preview를 사용하는 경우:

```text
pr-152.preview.example.com
pr-153.preview.example.com
```

## 작업 절차

환경/배포 관련 작업을 수행할 때 다음 순서를 따른다.

1. 현재 대상 환경이 Local/DEV/Preview/Staging/Production 중 무엇인지 확인한다.
2. 해당 환경이 Local인지 원격인지, 상시 환경인지 임시 환경인지 확인한다.
3. Application, API, DB, Storage, Cache, Queue, Scheduler, Secret의 환경 연결 관계를 확인한다.
4. DEV/STAGING이 Production 리소스를 참조하는지 검사한다.
5. 사용자 데이터와 Production Secret에 미치는 영향을 판단한다.
6. DB 변경이 있으면 Migration 및 하위 호환성을 검토한다.
7. Local 또는 Preview에서 테스트한다.
8. Staging에서 Production과 유사한 실제 배포 조건으로 검증한다.
9. Production 배포 전 Rollback 및 Backup 상태를 확인한다.
10. 승인된 CI/CD 경로로 Production에 반영한다.
11. 배포 후 Health Check, Smoke Test, Monitoring을 확인한다.

## 권장 배포 흐름

### 소규모 기본 흐름

```text
Local Development
      │
      ▼
Feature Branch
      │
      ▼
Automated Tests
      │
      ▼
Preview (필요 시)
      │
      ▼
Review
      │
      ▼
Staging
      │
      ▼
QA / Approval
      │
      ▼
main
      │
      ▼
CI/CD
      │
      ▼
Production
      │
      ▼
Health Check / Monitoring
```

### 팀 개발 흐름

```text
Local
  │
  ▼
Feature Branch
  │
  ▼
Preview / Remote DEV
  │
  ▼
Automated Tests / Review
  │
  ▼
Staging
  │
  ▼
Approval
  │
  ▼
Production
```

## 접근제어 규칙

### Production

실제 서비스가 필요한 엔드포인트만 Public으로 노출한다. 관리자 영역은 별도 MFA/SSO/Zero Trust 정책을 적용할 수 있다.

### Staging

SSO, Zero Trust Access, VPN, IP Allowlist 등으로 인증된 개발/QA 사용자만 접근하게 한다.

### Remote DEV / Preview

원칙적으로 비공개 또는 강한 제한 접근을 사용한다. 개발자, CI/CD Service Account, 승인된 AI Agent 등 필요한 주체만 접근한다.

### Local DEV

공개 인터넷에 직접 노출하지 않는다. 외부 서비스 연동 테스트 때문에 Tunnel이 필요하면 일시적이고 제한된 접근 정책을 사용한다.

프론트엔드만 보호하고 API를 Public으로 남겨두지 않는다. 예를 들어 다음 엔드포인트를 함께 검토한다.

```text
dev.example.com
api-dev.example.com
admin-dev.example.com
files-dev.example.com
```

## 데이터 및 인프라 격리

가능하면 다음 리소스를 환경별로 분리한다.

```text
Application
Database
Object Storage
Redis / Cache
Queue
Scheduler / Cron
Webhook Endpoint
OAuth Application
Payment Credentials
Email Provider Configuration
Secrets
```

Production 데이터를 개발환경으로 복제해야 한다면 개인정보 및 민감정보를 비식별화/마스킹하고, 재현 가능한 Seed Data를 우선 사용한다.

Staging 역시 기본적으로 Production DB를 직접 사용하지 않는다. 운영 수준의 데이터 분포가 필요하면 마스킹된 복제본, Synthetic Data 또는 목적에 맞는 별도 성능 테스트 환경을 사용한다.

## Supabase 사용 시 권장 패턴

Supabase는 구현 예시이며 이 스킬 자체는 특정 벤더에 종속되지 않는다.

일반적인 소규모 Supabase 프로젝트에서는 다음 패턴을 우선 검토한다.

```text
Local Development
→ Supabase Local (CLI/Docker)

Preview
→ Supabase Preview Branch
→ PR/기능별 임시 생성
→ 작업 종료 후 삭제

Staging
→ Persistent Branch 또는 별도 Staging Project
→ STAGING 전용 DB/Auth/Storage/Secrets

Production
→ Production Project
→ 실제 운영 DB/Auth/Storage/Secrets
```

주의:

- Preview/Staging Branch가 있다고 해서 Production DB를 직접 사용하는 것으로 간주하지 않는다.
- 환경마다 DB/자격증명을 분리한다.
- Staging의 Compute 크기와 데이터 규모가 Production과 다르면 성능 결과도 동일하다고 가정하지 않는다.
- 성능 검증이 목적이면 Production과 유사한 Compute, Index, 데이터 규모/분포를 별도 준비한다.
- Production 원본 개인정보를 Preview/DEV로 무분별하게 복제하지 않는다.

자세한 선택 기준은 [`references/local-preview-staging-strategy.md`](references/local-preview-staging-strategy.md)를 참조한다.

## Secret 처리 규칙

금지:

```text
Production .env Git Commit
API Key 하드코딩
DB Password 하드코딩
Production Secret을 AI Prompt에 평문 삽입
SSH Private Key Repository 저장
```

권장:

```text
Secret Manager
CI/CD Environment Secret
Short-lived Credential
Service Identity
Secret Broker
Workload Identity
```

벤더 제품은 구현 예시일 뿐이다. 프로젝트 환경에 맞는 공급자를 사용한다.

## AI Agent 권한 기본값

AI Agent에 기본 허용 가능한 작업:

- Local/Feature Branch 작업
- DEV/Preview 배포
- 테스트 실행
- Lint/Build
- Migration 초안 작성
- Pull Request 생성

기본 제한:

- Production Secret 열람
- Production DB 직접 Write
- Production 사용자 데이터 대량 접근
- Production Storage 삭제
- Production 배포 우회 실행

Production 작업은 `Agent → Review/Test → Staging → Approval → CI/CD → Production` 경로를 우선한다.

## 즉시 중단/재검토 조건

다음 조건에서는 자동 실행을 중지하거나 안전한 경로로 전환한다.

- 현재 DB가 DEV인지 Production인지 구분할 수 없음
- DEV와 PROD가 동일한 DB URL을 사용함
- Staging이 의도치 않게 Production DB에 Write 권한으로 연결됨
- Production Secret을 코드에 하드코딩해야 함
- `DROP`, `TRUNCATE`, 대량 `DELETE/UPDATE` 등 파괴적 작업이 Production에 적용됨
- Production 변경인데 Backup/Rollback 방법을 확인할 수 없음
- 운영 사용자 데이터를 DEV로 원본 그대로 복제하려 함
- Staging 검증 없이 고위험 Production 변경을 수행하려 함

## 상세 참조

전체 운영 가이드와 상세 예시는 다음을 사용한다.

- [`references/full-guide.md`](references/full-guide.md) — 상세 운영 가이드
- [`references/local-preview-staging-strategy.md`](references/local-preview-staging-strategy.md) — Local/Preview/Staging 배치 및 Supabase 적용 전략
- [`templates/environment-matrix.md`](templates/environment-matrix.md) — 프로젝트별 환경/권한 구성표
- [`templates/release-checklist.md`](templates/release-checklist.md) — Production 배포 체크리스트

## 완료 조건

환경 분리 작업은 최소한 다음이 확인되면 완료로 본다.

- [ ] Local/DEV/Preview/Staging/Production 각각의 역할이 정의됨
- [ ] Remote DEV가 정말 필요한지 검토됨
- [ ] Staging이 개인 개발 PC에만 종속되지 않음
- [ ] DEV/STAGING/PROD DB가 논리적 또는 물리적으로 분리됨
- [ ] Production Secret이 개발환경과 분리됨
- [ ] DEV/STAGING 접근제어가 적용됨
- [ ] API/Admin 등 우회 진입점도 보호됨
- [ ] Migration 경로가 정의됨
- [ ] Production 배포 경로가 정의됨
- [ ] Rollback/Backup 수단이 존재함
- [ ] Health Check/Monitoring이 존재함
