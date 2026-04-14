# 최종 아키텍처 상세

## 전체 흐름도

```
사용자 브라우저 / PWA
│
├── 정적 자산 (이미지/JS/CSS/폰트)
│   └── CloudFront (CDN, 전 세계 엣지 캐싱)
│       └── S3 버킷 (mfp-assets)
│           ├── 유저 업로드 이미지 (프로필, 포스트 등)
│           └── attached_assets, uploads/banners
│
├── 첫 페이지 로드 (index.html → React SPA)
│   └── ALB (HTTPS 종료, ACM 인증서)
│       └── ECS Fargate (Express 서버, 포트 5000)
│           └── dist/public/index.html 반환
│
├── API 호출 (/api/*)
│   └── ALB
│       └── ECS Fargate (Express 서버)
│           ├── RDS PostgreSQL (데이터 읽기/쓰기)
│           ├── S3 (이미지 업로드 시 Sharp 처리 후 저장)
│           ├── Firebase FCM (푸시 발송)
│           ├── Toss/PayPal (결제)
│           ├── Twilio (SMS 인증)
│           └── Gmail SMTP (이메일)
│
└── 푸시 알림 수신
    └── Firebase FCM → 브라우저/디바이스
```

## NAS 공연정보 수집 파이프라인

```
Instagram (Apify 스크래핑)
    ↓
NAS Backend (Mac Mini, Flask)
    ↓ Gemini AI 분석, event_confidence >= 1.5 필터
NAS PostgreSQL (pomfs-nas:5433) — 스테이징
    ↓ 최종 확인 후 venue_neon_sync (5워커 병렬)
AWS RDS PostgreSQL — 운영
    ↓
MFP 앱 (React) — 사용자 화면
```

## AWS 리소스 구성

| 리소스 | 설정 |
|--------|------|
| VPC | 퍼블릭 서브넷 2 + 프라이빗 서브넷 2 (AZ 2개) |
| ALB | 퍼블릭 서브넷, 443→5000, ACM SSL |
| ECS Fargate | 프라이빗 서브넷, desired_count=1 |
| RDS PostgreSQL | 프라이빗 서브넷, 자동 백업 7일, NAS IP 허용, 저장 암호화(KMS) + SSL 강제 |
| S3 | mfp-assets 버킷, 퍼블릭 차단 + CloudFront OAC |
| CloudFront | S3 오리진, 글로벌 CDN |
| Route 53 | community.prideofmisfits.com → ALB |
| Secrets Manager | 60개 환경변수 |
| ECR | Docker 이미지 저장소 |

## 인증 흐름 (AWS 전환 후)

```
사용자 → "Google로 로그인" 클릭
  → Express (passport-google-oauth20) → Google OAuth 서버
  → redirect_uri: https://community.prideofmisfits.com/auth/google/callback
  → Google 인증 완료 → 콜백으로 코드 전달
  → Express가 토큰 교환 → Google 프로필 획득
  → RDS에서 기존 유저 조회 또는 신규 생성
  → connect-pg-simple → RDS sessions 테이블에 세션 저장
  → 쿠키 설정 (sameSite: 'lax', secure: true)
  → React로 리디렉트 → 로그인 완료
```

## 폐기 서비스 vs 유지 서비스

### 폐기 (4개)
- Replit → ECS Fargate + ALB
- Neon DB → RDS PostgreSQL
- Supabase → RDS 자동 스냅샷
- GCS → S3 + CloudFront

### 유지 (외부 API, 코드 변경 없음)
- Firebase FCM, Google OAuth, Apple Sign-in
- Toss Payments, PayPal, Twilio, Gmail SMTP
- Google Translate, Maps, Kakao Maps
- Gemini AI, OpenAI, YouTube API

### 유지 (Connector 교체 필요)
- Notion — Replit Connector → NOTION_API_KEY
- Google Sheets — Replit Connector → 서비스 계정
