# My Skill Pack

개인 제작 AI/개발 자동화 스킬을 범용적으로 보관하는 저장소입니다.

## 디렉터리 규칙

```text
my-skill-pack/
├── README.md
└── skills/
    └── <category>/
        └── <skill-name>/
            ├── SKILL.md
            ├── references/
            │   └── ...
            └── templates/
                └── ...
```

- `SKILL.md`: AI가 언제 이 스킬을 사용하고 어떤 규칙을 지켜야 하는지 정의하는 핵심 파일
- `references/`: 상세 지식, 설계 기준, 긴 운영 가이드
- `templates/`: 체크리스트, 설정표 등 반복 사용 가능한 양식
- `<category>`: `devops`, `coding`, `database`, `security`, `automation`, `game-dev` 등 범용 카테고리

## Skills

| Category | Skill | Description |
|---|---|---|
| DevOps | [`environment-management`](skills/devops/environment-management/SKILL.md) | Local/DEV/Preview/Staging/Production 환경 분리, 접근제어, DB/Secret 격리, CI/CD, Migration, Rollback 운영 규칙 |

## 설계 원칙

이 저장소의 스킬은 가능한 한 특정 벤더에 종속되지 않도록 작성합니다. Cloudflare, Supabase, GitHub Actions 같은 제품명은 구현 예시로 취급하고, 상위 규칙은 Access Gateway, Database Isolation, Secret Store, Deployment Approval처럼 공급자 중립적으로 정의합니다.
