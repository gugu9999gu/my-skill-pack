# Environment Matrix Template

프로젝트 초기 구축 또는 환경 감사 시 복사해서 사용한다.

> 환경(Environment)은 반드시 별도의 상시 서버를 의미하지 않는다. Local DEV는 개발자 PC, Preview는 임시 원격 환경, Staging/Production은 독립 원격 환경으로 구성할 수 있다.

## Environment Role Matrix

| Environment | 권장 위치 | 목적 | 상시 여부 | 외부 접근 |
|---|---|---|---|---|
| Local DEV | 개발자 PC | 일상 개발 / 단위 테스트 | 상시(개발 시) | 비공개 |
| Remote DEV | Cloud / 사내 서버 | 팀 통합 개발 | 선택 | 제한 |
| Preview | 원격 임시 환경 | PR / 기능별 배포 검증 | 임시 | 제한 |
| Staging | 독립 원격 환경 | 운영 직전 검증 | 권장 | 인증 사용자만 |
| Production | 독립 원격 환경 | 실제 서비스 | 상시 | 서비스 정책에 따름 |

## Resource Matrix

| Resource | Local DEV | Remote DEV / Preview | Staging | Production |
|---|---|---|---|---|
| Frontend URL | | | | |
| API URL | | | | |
| Database | | | | |
| Object Storage | | | | |
| Redis / Cache | | | | |
| Queue | | | | |
| Scheduler / Cron | | | | |
| Webhook | | | | |
| OAuth App | | | | |
| Payment Mode | | | | |
| Email Mode | | | | |
| Secret Store | | | | |
| Access Policy | | | | |
| Deployment Source | | | | |
| Backup | | | | |
| Monitoring | | | | |

## Access Matrix

| Role | Local DEV | Remote DEV / Preview | Staging | Production |
|---|---:|---:|---:|---:|
| Developer | | | | |
| QA | | | | |
| AI Worker | | | | |
| AI Reviewer | | | | |
| CI/CD Service Account | | | | |
| Administrator | | | | |

## Supabase Example

| Environment | Supabase 권장 방식 |
|---|---|
| Local DEV | Supabase Local (CLI / Docker) |
| Remote DEV | 필요할 때만 별도 Branch/Project |
| Preview | PR별 Preview Branch |
| Staging | Persistent Branch 또는 별도 Staging Project |
| Production | Production Project |

## 확인 사항

- [ ] Remote DEV 서버가 실제로 필요한지 검토했다.
- [ ] Local 개발환경만으로 충분한 영역을 불필요하게 Cloud에 상시 운영하지 않는다.
- [ ] Staging이 개인 개발 PC에만 종속되지 않는다.
- [ ] Staging에서 DNS/HTTPS/OAuth/Webhook/CI/CD 등 실제 배포 조건을 검증할 수 있다.
- [ ] DEV가 Production DB를 참조하지 않는다.
- [ ] Staging이 Production DB를 Write 권한으로 참조하지 않는다.
- [ ] Production Secret은 DEV/Preview에서 사용할 수 없다.
- [ ] 환경별 Storage/Cache/Queue 충돌 가능성을 검토했다.
- [ ] Scheduler/Webhook이 환경별로 분리되어 있다.
- [ ] Staging/Remote DEV/Preview의 Public 노출 여부를 검토했다.
- [ ] 성능 테스트가 필요한 경우 Staging의 Compute와 데이터 규모가 Production과 충분히 유사한지 확인했다.
