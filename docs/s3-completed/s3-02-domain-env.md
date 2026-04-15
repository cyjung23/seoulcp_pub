# S3-02: 도메인 및 환경 설정 (DOMAIN-001, ENV-001~004)

**최종 갱신:** 2026-04-10

## DOMAIN-001 — 도메인 연결 및 URL 통일
- 작업일: 2026-03-31. 상태: 완료.
- 가비아 seoulcp.com -> Vercel 연결
- DNS: A(@->76.76.21.21), CNAME(www->cname.vercel-dns.com), SSL 자동

## ENV-001 — GitHub 저장소 비공개 전환
- 작업일: 2026-04-01. 상태: 완료.
- Public -> Private. Vercel 자동 배포 정상 확인.

## ENV-002 — GitHub Codespaces 개발 환경 전환
- 작업일: 2026-04-07. 상태: 완료.
- Codespace 생성, npm install/dev 정상, .env.local Supabase 연결 확인
- 환경: Next.js 16.2.1(Turbopack), /workspaces/seoulcp/, Linux Ubuntu bash

## ENV-003 — GitHub Secrets 등록
- 작업일: 2026-04-07. 상태: 완료.
- NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY

## ENV-004 — devcontainer.json 설정
- 작업일: 2026-04-07. 상태: 완료.
- .devcontainer/devcontainer.json, postCreateCommand로 npm install + Secrets->.env.local
