# 1. AWS 인프라 구성 체크리스트

## 1-1. 네트워크 & 보안
- [ ] AWS 계정 & IAM 설정 (MFP 전용 IAM 역할/정책)
- [ ] VPC & 서브넷 구성 (퍼블릭 2 + 프라이빗 2, AZ 2개+)
- [ ] Security Group: ALB(80/443 인바운드), ECS(5000 ALB에서만), RDS(5432 ECS+NAS IP만)
- [ ] NAT Gateway (프라이빗 서브넷 아웃바운드용)

## 1-2. 컴퓨팅 & 배포
- [ ] ECS Fargate 클러스터 + 태스크 정의 (desired_count=1, 스케줄러 이중실행 방지)
- [ ] ECR 레포지토리 생성 + Docker 이미지 빌드/푸시
- [ ] Dockerfile 작성: FROM node:20-alpine, apk add vips-dev (Sharp 의존), EXPOSE 5000
- [ ] ALB + 타겟 그룹 설정 (443→5000, 헬스체크 /health)
- [ ] ACM SSL 인증서 발급 (community.prideofmisfits.com) + ALB 연결

## 1-3. 데이터베이스
- [ ] RDS PostgreSQL 인스턴스 생성 (프라이빗 서브넷, 자동 백업 7일)
- [ ] Neon DB 풀 덤프: `pg_dump --format=custom --no-owner $NEON_URL > mfp.dump`
- [ ] RDS에 복원: `pg_restore --no-owner --no-acl -d $RDS_URL mfp.dump`
- [ ] 111개 테이블 행 수 검증 (Neon vs RDS 일치 확인)
- [ ] NAS Mac Mini → RDS 접속 테스트 (IP 화이트리스트 확인)

## 1-4. 스토리지 & CDN
- [ ] S3 버킷 생성 (mfp-assets) + CloudFront 배포
- [ ] GCS 버킷 → S3 전량 복사 (rclone 또는 gsutil + aws s3 sync)
- [ ] attached_assets (763MB) + uploads (35MB) S3 업로드
- [ ] DB 내 이미지 URL 일괄 치환 (storage.googleapis.com → CloudFront 도메인)
- [ ] ECS IAM 역할에 S3 읽기/쓰기 권한 부여

## 1-5. 도메인 & DNS
- [ ] Route 53에 community.prideofmisfits.com ALIAS → ALB 등록
- [ ] DNS TTL 60초로 낮춤 (전환 24시간 전)

## 1-6. 비밀 관리
- [ ] AWS Secrets Manager에 60개 환경변수 등록
- [ ] ECS 태스크 정의에서 Secrets Manager 참조 설정
