# GAP 분석: 기존 Notion 체크리스트 vs 실제 코드

> 코드베이스 전수 분석으로 발견한 누락/오류 14건

## CRITICAL (서버 부팅 불가)

| # | 항목 | 파일 |
|---|------|------|
| G1 | **Replit Connectors 대체 필요** — googleSheetsBackup.ts와 notionClient.ts가 REPLIT_CONNECTORS_HOSTNAME으로 OAuth 토큰 획득. AWS에서 100% 크래시 | `server/googleSheetsBackup.ts`, `server/lib/notionClient.ts` |
| G2 | **REPLIT_DOMAINS 부재시 서버 즉시 크래시** — replitAuth.ts:27-29에서 throw Error. 환경변수 없으면 부팅 불가 | `server/replitAuth.ts` |
| G3 | **Replit-Bonsai UA 감지 코드 5곳** — FCM, 푸시 프롬프트, 온보딩, Google OAuth에서 네이티브 앱 감지. 데드코드화 | `firebase.ts`, `PushPromptBottomSheet.tsx`, `OnboardingModal.tsx`, `googleAuth.ts` |
| G4 | **OpenAI BASE_URL이 Replit 프록시일 가능성** — 확인 필요 | `server/routes/botManagement.ts` |

## HIGH (데이터 손실/기능 장애)

| # | 항목 | 파일 |
|---|------|------|
| G5 | **Supabase 백업 동기화 누락** — 시간당 86테이블 동기화 언급 없음 | `server/supabaseSync.ts` |
| G6 | **Scheduler 이중 실행** — ECS 복수 태스크 시 모든 작업 2배 실행 | `server/scheduler.ts` |
| G7 | **emailService.ts 하드코딩 URL 8곳** — 'your-domain.replit.app' | `server/emailService.ts` |
| G8 | **config.ts에 Google OAuth 시크릿 하드코딩** — 보안 이슈 | `server/config.ts:50-51` |

## MEDIUM (기능 저하)

| # | 항목 |
|---|------|
| G9 | 업로드 파일 이전 (attached_assets 763MB + uploads 35MB) |
| G10 | PWA manifest + Service Worker 도메인 검증 |
| G11 | Sharp(이미지 처리) Alpine 컨테이너 호환 — vips-dev 필수 |
| G12 | Gmail SMTP AWS IP 차단 가능성 |
| G13 | Gemini AI 관리자 검색 — GEMINI_API_KEY 환경변수 |
| G14 | client/index.html Replit 배너 스크립트 — 불필요 외부 로드 |
