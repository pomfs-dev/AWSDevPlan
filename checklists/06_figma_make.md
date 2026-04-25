# 6. Figma Make 병합 후속 작업

> MFP 클라이언트에 Figma Make로 생성된 UI(랜딩/온보딩, 디자인 토큰, 컴포넌트 재디자인, 반응형·테마)가 `main`에 병합되면서 Phase 1(02_code_changes.md 2-6)과 동일한 클라이언트 파일 6개를 건드린다. 충돌 해결·정합성 검증·순서 조정·롤백을 한 체크리스트에서 관리한다.

## 6-1. 병합된 작업 범위
- [ ] 랜딩/온보딩 UI 개선 대상 파일 식별 — `client/src/pages/Landing.tsx`, `client/src/components/OnboardingModal.tsx`
- [ ] 디자인 토큰 소스 확정 — `client/src/index.css` `:root` CSS 변수 vs `tailwind.config.ts theme.extend`. 단일 소스 원칙
- [ ] 컴포넌트 재디자인 목록 — `client/src/components/ui/*`(shadcn) 중 교체·수정된 항목 파일별로 정리
- [ ] 반응형 검증 뷰포트 — 375 / 414 / 768 / 1024
- [ ] 다크모드 토큰 누락/깜빡임 여부
- [ ] PWA 안전 영역(`env(safe-area-inset-*)`) 대응 여부

## 6-2. 기존 코드와 통합 검증 (02_code_changes.md 2-6 / 2-1 / 2-2 와 충돌 지점)
- [ ] `client/src/pages/Landing.tsx` — `handleReplitLogin` 제거(02_code_changes.md 2-6) 후 Figma Make 로그인 버튼 레이아웃에 Replit 버튼 마크업이 남아있지 않은지
- [ ] `client/src/components/OnboardingModal.tsx` — `Replit-Bonsai` UA 체크(L39) 제거 후 Figma Make 분기가 네이티브 앱 감지를 가정하지 않는지
- [ ] `client/src/components/PushPromptBottomSheet.tsx` — UA 체크(L18) 제거 후 Figma Make 시트 UI 표시 경로 정상
- [ ] `client/src/lib/firebase.ts` — `isReplitNativeApp*()` 2개 제거가 Figma Make 푸시 프롬프트 호출부와 정합
- [ ] `client/index.html` — Replit 배너(L306) 제거와 Figma Make가 주입한 `<meta>`/폰트/스크립트 중복 없음
- [ ] `vite.config.ts` — `@replit/vite-plugin-*` 2개 제거 후 Figma Make 의존성(radix, lucide-react, framer-motion 등 신규 import) 빌드 정상
- [ ] `server/emailService.ts` 템플릿 — 이메일 UI 색/로고가 Figma Make 디자인 토큰과 동기화
- [ ] `server/routes/auth.ts` — `type === 'replit'` 제거(L390-392) 후 리다이렉트 URL이 Figma Make 랜딩 경로와 일치

## 6-3. UI 테스트
- [ ] `npm run build` 성공, 번들 사이즈 회귀 확인 (이전 대비 +20% 이내)
- [ ] Lighthouse 스코어 90+ (Performance / Best Practices / SEO / A11y) — 랜딩, 온보딩, 메인 피드
- [ ] 크로스 브라우저 수동 QA — Chrome 최신 / Safari iOS 16+ / Samsung Internet / Android Chrome
- [ ] PWA 설치 후 `display:standalone` 레이아웃 정상 (상태바, 하단 제스처 영역)
- [ ] 다크모드 토글 전환 시 깜빡임/누락 컬러 없음
- [ ] 시각적 리그레션 — 주요 페이지(Landing / Onboarding / Feed / Profile) before/after 스크린샷 비교

## 6-4. 스타일 정리 / 리팩터링
- [ ] Figma Make 인라인 `style={{}}` → Tailwind 클래스 또는 `@apply`로 치환
- [ ] 하드코딩 HEX → CSS 변수 또는 `tailwind.config.ts theme.extend.colors`
- [ ] 중복 컴포넌트 — Figma Make가 만든 Button/Card/Input이 `components/ui/*` shadcn 버전과 겹치면 shadcn 쪽으로 흡수
- [ ] 미사용 에셋/컴포넌트 삭제 — `knip` 또는 `ts-prune`으로 orphan 탐지
- [ ] 디자인 토큰 단일 소스 — `client/src/index.css` `:root` CSS 변수 + tailwind theme에서 동일 이름 참조

## 6-5. 마이그레이션과의 순서 조정
- [ ] Figma Make 병합 커밋을 `aws-migration` 브랜치에 **먼저** merge/rebase
- [ ] Phase 1 서버 작업(`02_code_changes.md` 2-1 ~ 2-5)은 Figma Make와 무관 — 병렬 진행
- [ ] Phase 1 클라이언트 작업(`02_code_changes.md` 2-6)은 Figma Make 병합 **이후**에 진행 → 충돌 최소화
- [ ] `package.json`(`02_code_changes.md` 2-7) 의존성 정리 시 Figma Make가 추가한 신규 라이브러리 유지
- [ ] `npm run build` 성공 후에만 Phase 2 AWS 인프라 배포(`01_aws_infra.md`) 진행

## 6-6. 롤백 고려
- [ ] Figma Make 병합 직전 태그 생성 — `git tag pre-figma-make`
- [ ] UI 회귀 발견 시 `aws-migration` 브랜치에서 Figma Make 커밋만 revert 가능하도록 단일 merge commit 유지
- [ ] 전체 마이그레이션 롤백은 `05_rollback.md` Level 1~3 절차 따름
