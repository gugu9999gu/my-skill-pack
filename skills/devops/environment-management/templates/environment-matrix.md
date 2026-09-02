# Environment Matrix Template

프로젝트 초기 구축 또는 환경 감사 시 복사해서 사용한다.

| Resource | Local | DEV / Preview | Staging | Production |
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

| Role | Local | DEV | Staging | Production |
|---|---:|---:|---:|---:|
| Developer | | | | |
| QA | | | | |
| AI Worker | | | | |
| AI Reviewer | | | | |
| CI/CD Service Account | | | | |
| Administrator | | | | |

## 확인 사항

- [ ] DEV가 Production DB를 참조하지 않는다.
- [ ] Staging이 Production DB를 참조하지 않는다.
- [ ] Production Secret은 DEV/Preview에서 사용할 수 없다.
- [ ] 환경별 Storage/Cache/Queue 충돌 가능성을 검토했다.
- [ ] Scheduler/Webhook이 환경별로 분리되어 있다.
- [ ] Staging/DEV의 Public 노출 여부를 검토했다.
