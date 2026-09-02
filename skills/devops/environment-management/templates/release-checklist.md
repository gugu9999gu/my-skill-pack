# Production Release Checklist

## 변경 전

- [ ] 대상 환경이 Production임을 명확히 확인했다.
- [ ] 변경사항이 Git에 커밋되어 있다.
- [ ] 테스트가 통과했다.
- [ ] Preview/DEV 검증을 완료했다.
- [ ] Staging 검증을 완료했다.
- [ ] DB Migration 영향을 검토했다.
- [ ] 파괴적 SQL이 포함되는지 확인했다.
- [ ] Production Secret 변경 여부를 확인했다.
- [ ] 최근 Backup 또는 복구 지점이 존재한다.
- [ ] Rollback 가능한 이전 Artifact/Version이 존재한다.

## 배포

- [ ] 승인된 브랜치/태그에서 배포한다.
- [ ] 승인된 CI/CD Identity를 사용한다.
- [ ] Migration을 정해진 순서로 적용한다.
- [ ] 배포 로그와 변경 버전을 기록한다.

## 배포 후

- [ ] `/health` 또는 동등한 Health Check가 정상이다.
- [ ] 핵심 사용자 경로 Smoke Test를 통과했다.
- [ ] API Error Rate가 비정상적으로 증가하지 않았다.
- [ ] Latency가 정상 범위다.
- [ ] DB/Cache/Queue 연결이 정상이다.
- [ ] Background Job/Scheduler가 정상이다.
- [ ] 로그와 알림을 확인했다.

## 이상 발생 시

- [ ] 신규 트래픽을 중단하거나 이전 버전으로 전환한다.
- [ ] 코드 롤백과 DB 롤백을 구분해 판단한다.
- [ ] 데이터 손상 가능성을 먼저 확인한다.
- [ ] 장애 발생 시점/버전/원인을 기록한다.
