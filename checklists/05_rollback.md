# 5. 롤백 계획

## 롤백 수준

### Level 1: DNS 롤백 (< 5분)
- DNS 레코드를 Replit 배포로 복귀
- Replit은 스탠바이로 계속 실행 중이므로 즉시 복구
- TTL 60초이므로 전파 빠름

### Level 2: 코드 롤백 (< 15분)
- `git checkout replit-prod-final` → Replit 재배포
- Replit Secrets에 환경변수 그대로 유지되어 있음

### Level 3: 데이터 롤백 (필요 시)
- 전환 전: Neon DB가 원본, RDS가 복사본
- 전환 후: RDS가 운영, Neon에 마지막 덤프 보존
- 롤백 시: Neon 덤프에서 복원 가능

## 핵심 원칙
- **Replit + Neon을 최소 2주 스탠바이로 유지**
- Replit 환경변수 삭제 금지 (30일까지)
- Neon DB 계정 해지 금지 (RDS 안정성 확인될 때까지)
- DNS 전환 전 반드시 스테이징에서 전체 테스트 완료

## 롤백 판단 기준
| 상황 | 조치 |
|------|------|
| 서버 부팅 실패 | Level 1 즉시 실행 |
| OAuth 로그인 불가 | Level 1, 외부 서비스 설정 재확인 |
| DB 연결 실패 | Level 1, RDS Security Group 확인 |
| 이미지 로드 실패 | S3/CloudFront 설정 확인, 심각하면 Level 1 |
| 결제 실패 | Toss webhook 확인, Level 1은 마지막 수단 |
| 성능 저하 | ECS 스펙 조정, 롤백 불필요 |
