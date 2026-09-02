---
name: environment-management
title: Development / Staging / Production Environment Management
version: "1.1"
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

## 기본 환경 모델

| Environment | 목적 | 외부 사용자 접근 | 데이터 |
|---|---|---|---|
| Local | 개인 개발/단위 테스트 | 불가 | 로컬/테스트 |
| DEV / Preview | 기능 개발/통합 테스트 | 제한 | DEV 전용 |
| Staging | 운영 직전 검증 | 인증된 QA/내부 사용자 | STAGING 전용 |
| Production | 실제 서비스 | 허용 | 실제 운영 데이터 |

권장 주소 예시는 다음과 같다. 실제 도메인은 프로젝트 상황에 맞춰 변경한다.

```text
localhost:3000
dev.example.com
staging.example.com
example.com

api-dev.example.com
api-staging.example.com
api.example.com
```

## 작업 절차

환경/배포 관련 작업을 수행할 때 다음 순서를 따른다.

1. 현재 대상 환경이 Local/DEV/Staging/Production 중 무엇인지 확인한다.
2. Application, API, DB, Storage, Cache, Queue, Scheduler, Secret의 환경 연결 관계를 확인한다.
3. DEV/STAGING이 Production 리소스를 참조하는지 검사한다.
4. 사용자 데이터와 Production Secret에 미치는 영향을 판단한다.
5. DB 변경이 있으면 Migration 및 하위 호환성을 검토한다.
6. Preview/DEV에서 테스트한다.
7. Staging에서 Production과 유사한 조건으로 검증한다.
8. Production 배포 전 Rollback 및 Backup 상태를 확인한다.
9. 승인된 CI/CD 경로로 Production에 반영한다.
10. 배포 후 Health Check, Smoke Test, Monitoring을 확인한다.

## 권장 배포 흐름

```text
Feature Branch
      │
      ▼
Preview / DEV
      │
      ▼
Automated Tests
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

## 접근제어 규칙

### Production

실제 서비스가 필요한 엔드포인트만 Public으로 노출한다. 관리자 영역은 별도 MFA/SSO/Zero Trust 정책을 적용할 수 있다.

### Staging

SSO, Zero Trust Access, VPN, IP Allowlist 등으로 인증된 개발/QA 사용자만 접근하게 한다.

### DEV / Preview

원칙적으로 비공개 또는 강한 제한 접근을 사용한다. 개발자, CI/CD Service Account, 승인된 AI Agent 등 필요한 주체만 접근한다.

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

- Feature Branch 생성/수정
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
- Production Secret을 코드에 하드코딩해야 함
- `DROP`, `TRUNCATE`, 대량 `DELETE/UPDATE` 등 파괴적 작업이 Production에 적용됨
- Production 변경인데 Backup/Rollback 방법을 확인할 수 없음
- 운영 사용자 데이터를 DEV로 원본 그대로 복제하려 함
- Staging 검증 없이 고위험 Production 변경을 수행하려 함

## 상세 참조

전체 운영 가이드와 상세 예시는 다음을 사용한다.

- [`references/full-guide.md`](references/full-guide.md) — 상세 운영 가이드
- [`templates/environment-matrix.md`](templates/environment-matrix.md) — 프로젝트별 환경/권한 구성표
- [`templates/release-checklist.md`](templates/release-checklist.md) — Production 배포 체크리스트

## 완료 조건

환경 분리 작업은 최소한 다음이 확인되면 완료로 본다.

- [ ] DEV/STAGING/PROD DB가 논리적 또는 물리적으로 분리됨
- [ ] Production Secret이 개발환경과 분리됨
- [ ] DEV/STAGING 접근제어가 적용됨
- [ ] API/Admin 등 우회 진입점도 보호됨
- [ ] Migration 경로가 정의됨
- [ ] Production 배포 경로가 정의됨
- [ ] Rollback/Backup 수단이 존재함
- [ ] Health Check/Monitoring이 존재함
