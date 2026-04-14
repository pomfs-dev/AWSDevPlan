# 2. 코드 변경 체크리스트 (22개 파일)

## 2-1. Replit 종속성 제거 (서버 7개)
- [ ] `server/config.ts` — REPL_* 환경변수 → NODE_ENV 단일화, 하드코딩 Google 시크릿 제거
- [ ] `server/replitAuth.ts` — OIDC 제거, REPLIT_DOMAINS 가드 제거, 세션 쿠키 단순화 (sameSite:'lax', secure:true), type:'replit' 분기 제거, isAuthenticated에서 Replit 토큰 리프레시 제거
- [ ] `server/index.ts` — 배포 감지 블록 NODE_ENV로 교체, trust proxy 단순화, initializeGCS → initializeS3
- [ ] `server/googleAuth.ts` — Replit-Bonsai UA 제거(L77), REPLIT_DEPLOYMENT 체크 제거(L81, L104)
- [ ] `server/admin/adminDb.ts` — isProductionEnvironment() NODE_ENV 단일 체크
- [ ] `server/emailService.ts` — 'your-domain.replit.app' 8곳 → 'community.prideofmisfits.com'
- [ ] `server/routes/auth.ts` — type === 'replit' 분기 제거 (L390-392)

## 2-2. Replit Connector 교체 (2개)
- [ ] `server/googleSheetsBackup.ts` — Replit Connector → Google 서비스 계정 인증 (google.auth.GoogleAuth)
- [ ] `server/lib/notionClient.ts` — Replit Connector → `new Client({ auth: process.env.NOTION_API_KEY })`

## 2-3. DB 드라이버 교체 — Neon → pg (2개)
- [ ] `server/db.ts` — @neondatabase/serverless NeonPool → 표준 pg Pool. Supabase 폴백 로직 전체 제거
- [ ] `drizzle.config.ts` — Neon serverless → 표준 PostgreSQL 드라이버

## 2-4. Supabase 코드 제거 (3개)
- [ ] `server/supabaseSync.ts` — **삭제** (86테이블 시간별 동기화 제거, RDS 자동 스냅샷으로 대체)
- [ ] `server/supabaseDb.ts` — **삭제** (Supabase 연결 코드 전체 제거)
- [ ] `server/scheduler.ts` — Supabase 동기화 스케줄 제거

## 2-5. 이미지 저장소 교체 — GCS → S3 (1개)
- [ ] `server/googleCloudStorage.ts` — **전면 재작성**. @google-cloud/storage → @aws-sdk/client-s3. 모든 함수 교체:
  - uploadImageToGCS → uploadImageToS3
  - deleteImageFromGCS → deleteImageFromS3
  - generateSignedUrl (S3 presigned URL)
  - listFiles, getFileMetadata, cleanupOldFiles

## 2-6. 클라이언트 Replit 제거 (6개)
- [ ] `vite.config.ts` — @replit/* 플러그인 2개 import + 사용 제거
- [ ] `client/index.html` — Replit 배너 스크립트 태그 제거 (L306)
- [ ] `client/src/lib/firebase.ts` — isReplitNativeApp*() 함수 2개 + 호출부 제거
- [ ] `client/src/components/PushPromptBottomSheet.tsx` — Replit-Bonsai UA 체크 제거 (L18)
- [ ] `client/src/components/OnboardingModal.tsx` — Replit-Bonsai UA 체크 제거 (L39)
- [ ] `client/src/pages/Landing.tsx` — handleReplitLogin 함수 + Replit 로그인 버튼 제거

## 2-7. 의존성 정리 (package.json)
- [ ] **제거**: @replit/vite-plugin-runtime-error-modal, @replit/vite-plugin-cartographer, @neondatabase/serverless, @google-cloud/storage
- [ ] **추가**: @aws-sdk/client-s3, @aws-sdk/s3-request-presigner

## 2-8. NAS Backend 변경
- [ ] venue_neon_sync의 DATABASE_URL을 Neon → RDS URL로 변경
- [ ] NAS의 모든 Neon 연결 설정에서 URL 업데이트

## 2-9. 신규 파일
- [ ] `Dockerfile` — Node 20 Alpine + vips-dev + 빌드 아티팩트
- [ ] `.env.template` — 60개 환경변수 전체 템플릿
- [ ] `.replit` 파일 삭제
- [ ] `replit.md` 파일 삭제
