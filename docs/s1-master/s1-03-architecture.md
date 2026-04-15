# S1-03: 아키텍처 및 배포 파이프라인

**최종 갱신:** 2026-04-10

## 배포 파이프라인

1. Codespaces에서 코드 수정
2. git add -> git commit -> git push origin main
3. Vercel 자동 감지 빌드 및 배포 (1~2분)
4. https://seoulcp.com 반영

## DNS 구성

- A 레코드: @ -> 76.76.21.21 (Vercel)
- CNAME: www -> cname.vercel-dns.com
- SSL: Vercel 자동 발급

## 환경변수

Codespaces Secrets: NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY. devcontainer.json postCreateCommand로 .env.local 자동 생성.

## DB 백업

GitHub Actions (.github/workflows/db-backup.yml): 매일 UTC 00:00 (KST 09:00) pg_dump, GitHub Artifacts 90일 보존, Transaction pooler(6543).

## JSON-LD 스키마 적용 현황

- 홈: WebSite + SearchAction + MedicalOrganization
- 클리닉 상세: MedicalBusiness + availableService + hasOfferCatalog
- 시술 상세: MedicalProcedure
- 고민 상세: MedicalCondition + possibleTreatment
- 위키 상세: FAQPage (5개 Q&A)

## AI 크롤러 허용 (robots.txt)

GPTBot, Google-Extended, ChatGPT-User, PerplexityBot, Amazonbot, ClaudeBot

## 표준문서 관리

- 저장소: https://github.com/cyjung23/pubrepo (Public)
- 경로: pubrepo/seoulcp-docs/
- AI 직접 read 가능 (raw.githubusercontent.com 경로)
- write는 Codespaces에서 /workspaces/pubrepo/ clone 후 수정 → push
- 갱신일: 2026-04-14

## Claude Code 환경 (2026-04-14)

- Claude Code v2.1.104 설치 (Windows 노트북)
- Claude Pro 구독, Sonnet 4.6 기본 모델
- claude.ai/code 웹 버전으로 seoulcp 레포 연결 완료
- 기존 프로젝트(seoulcp, cclinic)는 현행 Codespaces + 채팅 방식 유지
- Claude Code는 신규 프로젝트에서 사용 예정

## 커넥터 설정 (2026-04-14)

- Cloudflare Developer Platform: 읽기 도구 항상 허용, 쓰기/삭제 도구 승인 필요
- Supabase: 읽기 도구 항상 허용, 쓰기/삭제 도구 승인 필요
- Vercel: 읽기 도구 항상 허용, 쓰기/삭제(deploy 포함) 승인 필요
