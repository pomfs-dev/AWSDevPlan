# MFP AWS 마이그레이션 계획

> MusicFeedPlatform(MFP) 앱을 Replit에서 AWS로 전면 이전하는 프로젝트

## 확정 사항 (2026-04-14)

| 폐기 | AWS 대체 |
|------|---------|
| Replit (호스팅) | ECS Fargate + ALB |
| Neon DB | RDS PostgreSQL |
| Supabase (백업) | RDS 자동 스냅샷 |
| GCS (이미지) | S3 + CloudFront |

| 유지 서비스 |
|------------|
| Firebase FCM (푸시 알림) |
| Google OAuth / Apple Sign-in |
| Toss Payments / PayPal |
| Twilio (SMS) / Gmail SMTP |
| Google Translate / Maps / Kakao Maps |
| Gemini AI / OpenAI |
| Notion / Google Sheets (Replit Connector → 직접 API) |

## 아키텍처

```
사용자 브라우저 / PWA
  → (정적 자산: 이미지/JS/CSS) CloudFront → S3
  → (API / 페이지 요청) ALB → ECS Fargate(Express) → RDS(PostgreSQL)
  → (푸시 알림 수신) Firebase FCM

NAS Backend (Mac Mini) — 공연정보 수집
  Instagram(Apify) → Gemini AI → NAS PostgreSQL(스테이징)
  → 최종 확인 후 venue_neon_sync → AWS RDS(운영)
```

## 문서 구조

```
AWSDevPlan/
├── README.md                    ← 이 파일
├── MIGRATION_PLAN.md            ← 최종 마이그레이션 계획서 (전체)
├── checklists/
│   ├── 01_aws_infra.md          ← AWS 인프라 체크리스트
│   ├── 02_code_changes.md       ← 코드 변경 체크리스트 (22파일)
│   ├── 03_external_services.md  ← 외부 서비스 설정 변경
│   ├── 04_testing.md            ← 통합 테스트 체크리스트
│   ├── 05_rollback.md           ← 롤백 계획
│   └── 06_figma_make.md         ← Figma Make 병합 후속 작업
├── env/
│   └── .env.template            ← 60개 환경변수 템플릿
├── docker/
│   └── Dockerfile.template      ← ECS용 Dockerfile 템플릿
└── docs/
    ├── architecture.md          ← 상세 아키텍처 설명
    └── gap_analysis.md          ← 코드 분석 기반 GAP 14건
```

## 수치 요약

- **수정 파일**: 22개
- **환경변수**: 60개
- **npm 제거**: 4개 / 추가: 2개
- **DB 이전**: 111 테이블
- **이미지 이전**: GCS → S3
- **실행 기간**: ~30일 (6단계)

## 관련 링크

- **MFP 소스**: [pomfs-dev/MusicFeedPlatform](https://github.com/pomfs-dev/MusicFeedPlatform)
- **Notion 계획서**: [MFP AWS 마이그레이션 최종 계획서 (v2)](https://app.notion.com/p/MFP-AWS-v2-3427594c8852810c8e45d9d6321dae8d)
- **도메인**: community.prideofmisfits.com
