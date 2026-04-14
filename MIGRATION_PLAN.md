# MFP AWS 마이그레이션 최종 계획서

> 최종 수정: 2026-04-14 | 상태: 확정판
> 분석 기반: MFP GitHub 코드베이스 전수 분석 (506 TS파일, 111 테이블, 70+ API, ~80K LoC)

---

## 실행 순서 (6단계)

### Phase 0: 백업 + DB 마이그레이션 (Day 1-2)
**DB 마이그레이션 절차 (데이터 무결성 보장):**
- [ ] 유지보수 공지 — 최소 24시간 전 앱 내 배너 게시
- [ ] D-Day: Replit 앱을 유지보수 모드로 전환 (POST/PUT/PATCH/DELETE → 503 반환)
- [ ] Neon DB에 활성 쓰기 작업 없음 확인
- [ ] Neon DB 풀 덤프: `pg_dump --format=custom --no-owner --serializable-deferrable $NEON_URL > mfp_final.dump`
- [ ] GCS 이미지 전량 다운로드 (rclone 또는 gsutil)
- [ ] attached_assets (763MB) + uploads (35MB) 백업
- [ ] Replit Secrets에서 환경변수 전체 export
- [ ] Google Console OAuth redirect URI 스크린샷
- [ ] Apple Developer Console 설정 스크린샷
- [ ] Git 태그: `git tag replit-prod-final && git push origin replit-prod-final`
- [ ] 마이그레이션 브랜치: `git checkout -b aws-migration`

### Phase 1: 코드 변경 (Day 3-7)
순서가 중요 — 아래 순서대로 진행:
1. [ ] `server/config.ts` — 모든 파일이 참조하는 기반 설정
2. [ ] `server/replitAuth.ts` — 부팅 크래시 방지
3. [ ] `server/index.ts` — 배포 감지 + initializeS3
4. [ ] `server/db.ts` — Neon → pg Pool, Supabase 제거
5. [ ] `server/googleCloudStorage.ts` — GCS → S3 전면 재작성
6. [ ] `server/googleSheetsBackup.ts` — Replit Connector → 서비스 계정
7. [ ] `server/lib/notionClient.ts` — Replit Connector → API 키
8. [ ] 나머지 서버 파일 4개 (병렬 가능)
9. [ ] 클라이언트 파일 6개 (병렬 가능)
10. [ ] package.json 의존성 정리
11. [ ] Supabase 코드 삭제 (supabaseSync.ts, supabaseDb.ts)
12. [ ] `npm run build` 성공 확인

### Phase 2: AWS 인프라 구축 (Day 5-10, Phase 1과 병렬)
→ 상세: `checklists/01_aws_infra.md`

### Phase 3: 외부 서비스 설정 변경 (Day 8-12)
→ 상세: `checklists/03_external_services.md`

### Phase 4: 통합 테스트 (Day 13-16)
→ 상세: `checklists/04_testing.md`

### Phase 5: DNS 전환 & 라이브 (Day 17-18)
- [ ] DNS TTL을 60초로 낮춤 (최소 24시간 전)
- [ ] community.prideofmisfits.com → AWS ALB ALIAS 레코드 변경
- [ ] SSL 인증서 정상 동작 확인
- [ ] Replit은 계속 실행 (스탠바이) — 최소 2주 유지

### Phase 6: 사후 검증 & 정리 (Day 19-30)
- [ ] Day 1 후: 5xx 에러 0건, OAuth 로그인 성공, 이미지 정상
- [ ] Day 3 후: Google Sheets 백업 2회/일 정상
- [ ] Day 7 후: 전체 스케줄러 안정, 결제 최소 1건 검증
- [ ] Day 30 후: Replit 해지 가능 → pomfs-community.replit.app 리디렉트 후 종료
