# 3. 외부 서비스 설정 변경

## 인증 서비스
- [ ] **Google OAuth** — Google Cloud Console에서 redirect URI에 `https://community.prideofmisfits.com/auth/google/callback` 확인. Replit URI는 30일 후 제거
- [ ] **Apple Sign-in** — Apple Developer Console에서 authorized domains + return URL 확인
- [ ] **Firebase** — Firebase Console > authorized domains에 `community.prideofmisfits.com` 추가

## 결제 서비스
- [ ] **Toss Payments** — 머천트 대시보드에서 webhook URL 확인, IP 화이트리스트 업데이트 (AWS ALB IP)
- [ ] **PayPal** — webhook URL 확인 (도메인 기반이면 변경 불필요)

## 신규 설정 (Replit Connector 대체)
- [ ] **Notion** — notion.so/my-integrations에서 내부 통합 생성 → NOTION_API_KEY 발급 → Users/Rank Requests DB에 접근 권한 부여
- [ ] **Google Sheets** — GCP에서 서비스 계정 생성 → Google Sheets API + Drive API 활성화 → 백업 폴더에 Editor 공유 → GOOGLE_SHEETS_SERVICE_ACCOUNT 환경변수 설정

## 확인 필요
- [ ] **OpenAI** — AI_INTEGRATIONS_OPENAI_BASE_URL 확인. Replit AI 프록시 주소면 `https://api.openai.com/v1`로 변경
- [ ] **Kakao Maps** — 도메인 제한 설정 확인 (community.prideofmisfits.com 허용)
- [ ] **Gmail SMTP** — AWS IP에서 접속 테스트. Google 보안 경고 시 승인 필요. 실패 시 SendGrid 대체 고려
