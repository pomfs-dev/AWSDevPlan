# 마이그레이션 전략: 2단계 분리 순차 진행

> 2026-04-15 ultraplan 리뷰 기반

## 핵심 원칙

**"프론트 리디자인 + AWS 마이그레이션 동시 진행"은 최대 리스크.**

장애 발생 시 "인프라 문제 vs 코드 문제" 구분이 불가능하고, 롤백 대상도 불명확해진다.

## 권장 순서

### Phase A: 인프라 마이그레이션 (~30일)
```
기존 프론트 코드 그대로 → AWS 이전
↓ 안정화 기간 최소 2주 (모니터링, 성능 확인)
↓ 이 시점에서 Replit 해지
```
- 이 계획서(AWSDevPlan)의 Phase 0~6이 Phase A에 해당
- 기존 코드이므로 롤백이 확실 (DNS만 Replit으로 복귀)

### Phase B: 프론트 리디자인 (~60-90일)
```
AWS 인프라 위에서 새 프론트 개발
↓ 스테이징 환경에서 충분히 검증
↓ Blue/Green 또는 카나리 배포로 점진적 전환
```
- Figma Make 리디자인 코드를 AWS 위에서 개발
- 별도 스테이징 URL (staging-mfp.prideofmisfits.com)
- CI/CD 파이프라인 필수 (Phase B 시작 전 구축)

## 이유 3가지

| # | 이유 | 설명 |
|---|------|------|
| 1 | **장애 격리** | 인프라 문제와 코드 문제를 분리해서 디버깅 가능 |
| 2 | **롤백 단순화** | Phase A 중 문제 시 → DNS만 Replit으로 복귀 (기존 코드이므로 확실히 동작) |
| 3 | **리스크 분산** | Phase A만 성공해도 Replit 종속 탈출 완료 |

## CI/CD 파이프라인 (Phase B 시작 전 구축)

```
GitHub Actions 워크플로우:
  push to main
  → npm run build
  → Docker build
  → ECR push
  → ECS 서비스 업데이트
  → 헬스체크 실패 시 자동 롤백
```

### 필요 설정
- [ ] GitHub Actions 워크플로우 파일 (.github/workflows/deploy.yml)
- [ ] 스테이징 환경: 별도 ECS 서비스 + 별도 ALB 타겟 그룹
- [ ] ECS 배포 Circuit Breaker 활성화 (실패 시 자동 롤백)
- [ ] GitHub Environments: staging / production 분리
