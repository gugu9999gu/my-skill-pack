# Local / Preview / Staging 배치 전략

이 문서는 `environment-management` 스킬에서 **개발환경을 어디에 둘지** 판단할 때 사용하는 세부 기준이다.

## 1. 핵심 원칙

환경(Environment)은 서버 한 대와 동일한 개념이 아니다.

```text
Local DEV
→ 개발자 PC에서 실행 가능

Preview
→ PR/기능별 임시 원격 환경 가능

Staging
→ 운영 직전 검증을 위한 독립 원격 환경 권장

Production
→ 실제 서비스용 독립 운영 환경
```

따라서 소규모 프로젝트에서 `dev.example.com` 같은 Remote DEV 서버를 반드시 상시 운영할 필요는 없다.

## 2. 소규모/1인 개발 권장 기본값

```text
Developer / AI Agent
        │
        ▼
LOCAL DEVELOPMENT
├─ Frontend
├─ Backend
├─ Local DB
└─ Test Data
        │
        ▼
PREVIEW (선택)
PR/Feature별 임시 생성
        │
        ▼
STAGING
독립 원격 환경
        │
        ▼
PRODUCTION
실제 서비스
```

이 구조의 장점:

- DEV Cloud 비용 감소
- 개발 반복 속도 증가
- 인터넷 연결 없이도 상당수 개발 가능
- 운영 자원과 개발 자원 분리
- Staging에서 실제 배포 조건 검증 가능

## 3. Remote DEV가 필요한 조건

다음 조건이 있으면 `dev.example.com` 같은 원격 DEV 환경을 추가할 수 있다.

- 여러 개발자가 하나의 통합 환경을 공유해야 한다.
- 여러 AI Worker가 병렬 작업하고 원격 통합 결과를 지속적으로 확인해야 한다.
- 외부 SaaS나 Webhook 제공자가 localhost를 호출할 수 없다.
- 모바일 또는 사외 기기에서 DEV 기능을 지속적으로 확인해야 한다.
- QA/기획자가 Staging 이전 기능을 상시 확인해야 한다.
- Local 환경에서 재현하기 어려운 네트워크/인프라 의존성이 있다.

그 외에는 Local + Preview가 더 단순하고 비용 효율적일 수 있다.

## 4. Staging을 원격에 두는 이유

Staging은 코드 기능 테스트뿐 아니라 Production 조건의 재현이 목적이다.

대표 검증 대상:

- 실제 DNS
- HTTPS / TLS 인증서
- Reverse Proxy
- CDN
- WAF / Zero Trust Access
- Cookie Domain
- Secure / SameSite Cookie
- CORS
- OAuth Redirect URL
- 결제 Sandbox Callback
- 외부 Webhook
- Email Provider
- Object Storage / CDN
- Background Worker
- Scheduler / Cron
- 환경별 Secret 주입
- Production Build
- CI/CD Deployment
- Health Check

`localhost`만으로는 위 조건 상당수를 완전히 재현하기 어렵다.

## 5. '원격'은 반드시 Public Cloud를 의미하지 않는다

다음도 Staging이 될 수 있다.

```text
사내 Ubuntu Server
사내 VM
사내 Kubernetes
Private Cloud
Cloud VM
Managed PaaS
```

중요한 기준은 위치가 아니라 다음이다.

1. 개인 개발 PC에 종속되지 않는가?
2. 지속적으로 재현 가능한가?
3. Production과 유사한 네트워크 조건을 검증할 수 있는가?
4. 환경별 DB/Secret을 분리할 수 있는가?
5. CI/CD에서 배포 가능한가?

## 6. Database 배치

기본값:

```text
Local DEV
→ Local DB

Remote DEV / Preview
→ DEV/Preview 전용 DB

Staging
→ STAGING DB

Production
→ PROD DB
```

금지할 기본 패턴:

```text
Local DEV ──────┐
                ├→ PROD DB
Staging ────────┘
```

Staging도 원칙적으로 Production DB에 Write 권한으로 연결하지 않는다.

운영 데이터 분포가 필요한 경우 다음을 사용한다.

- Synthetic Data
- Seed Data
- 개인정보를 마스킹한 운영 데이터 복제본
- 목적에 맞는 별도 Performance Staging

## 7. 성능 검증 주의사항

Staging DB가 독립되어 있더라도 Staging과 Production의 성능이 자동으로 동일해지는 것은 아니다.

성능에 영향을 주는 요소:

```text
CPU / Compute Size
RAM
Disk / IOPS
Connection Pool
Index
DB Statistics
데이터 건수
데이터 분포
Cache 상태
Network Latency
동시 사용자 수
```

따라서 작은 Staging에서 빠르다고 Production에서도 동일하게 빠르다고 결론내리지 않는다.

성능 테스트가 목적이면 Production과 유사한 Compute, Index, 데이터 규모 및 데이터 분포를 준비한다.

## 8. Supabase 적용 예시

Supabase는 이 전략을 구현하는 한 가지 예시다.

### Local Development

```text
Application
localhost

Supabase Local
CLI / Docker

Database
Local PostgreSQL
```

일상적인 개발은 Local Supabase로 처리할 수 있다.

### Preview

```text
Feature Branch / Pull Request
        │
        ▼
Supabase Preview Branch
        │
        ▼
테스트
        │
        ▼
PR Merge / Close
        │
        ▼
Preview 삭제
```

Preview는 장기간 유지할 필요가 없는 기능별 검증 환경으로 사용한다.

### Staging

```text
staging.example.com
        │
        ▼
Persistent Branch
또는
별도 Staging Project
```

Staging에는 별도:

```text
Database
Auth
Storage
Secrets
Webhook
OAuth Redirect
```

구성을 사용한다.

### Production

```text
example.com
        │
        ▼
Production Project
├─ PROD DB
├─ PROD Auth
├─ PROD Storage
└─ PROD Secrets
```

## 9. Supabase 성능 관련 해석

Branch/Project가 분리되어 있으면 테스트 Query가 Production DB에 그대로 실행되는 구조는 아니다.

그러나 다음을 구분한다.

```text
리소스 격리
≠
성능 동일성
```

Preview/Staging의 Compute와 데이터 규모가 Production보다 작다면 속도 결과가 다를 수 있다.

따라서:

```text
기능 검증
→ 일반 Staging

성능 검증
→ Production 유사 Performance Staging
```

으로 목적을 분리할 수 있다.

## 10. 권장 의사결정 표

| 상황 | 권장 구성 |
|---|---|
| 1인 개발, 초기 서비스 | Local → Staging → Production |
| 1인 개발 + PR별 실제 배포 확인 | Local → Preview → Staging → Production |
| 소규모 팀 | Local → Preview/Remote DEV → Staging → Production |
| 다중 AI Worker | Local/Feature → Preview → Integration/Staging → Production |
| OAuth/Webhook 테스트 많음 | Remote Preview 또는 Remote DEV 추가 |
| DB 성능 검증 필요 | 별도 Performance Staging 추가 |

## 11. 최종 기본값

특별한 요구가 없다면 다음을 기본값으로 사용한다.

```text
DEV
→ Local 우선

REMOTE DEV
→ 필요할 때만

PREVIEW
→ 임시, 선택적

STAGING
→ 원격, Production 유사

PRODUCTION
→ 원격, 실제 서비스
```

즉 **개발 편의와 비용을 위해 Local을 적극 사용하고, 운영 직전 검증 품질을 위해 Staging은 독립된 원격 환경으로 유지한다.**
