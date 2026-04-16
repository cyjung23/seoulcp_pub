# S1-01: 프로젝트 개요 및 기술스택

**최종 갱신:** 2026-04-16

## 프로젝트 개요

Seoul Clinic Pick(SCP)은 서울 소재 미용/성형 클리닉 정보를 구조화하여 제공하는 한영 이중언어 웹서비스입니다. 시술(treatments), 고민(concerns), 장비(devices), 백과사전(encyclopedia/wiki) 4가지 축으로 클리닉을 탐색할 수 있으며, AI 검색엔진(GEO) 최적화를 통해 생성형 검색에서의 인용을 목표로 합니다.

- URL: https://seoulcp.com
- 저장소: https://github.com/cyjung23/seoulcp (Private)
- 문서 저장소: https://github.com/cyjung23/seoulcp_pub (Public)
- 도메인: seoulcp.com (가비아 등록, Vercel DNS)

## 기술스택

- 프레임워크: Next.js 16 (App Router, React 19, Turbopack)
- i18n: next-intl (ko/en, [locale] 라우팅)
- DB: Supabase PostgreSQL
- 인증: Supabase Auth (Email Provider, Confirm email ON)
- 호스팅: Vercel (자동 배포, GitHub main push 트리거)
- 개발환경: GitHub Codespaces (Linux Ubuntu, bash)
- 분석: GA4 (G-6CX6ER3EX6), Naver Analytics (wa:1632b4d20412f40)
- SEO: Google Search Console, Naver Search Advisor
- 백업: GitHub Actions (매일 KST 09:00 pg_dump, 90일 보존)

## Vercel 환경변수

- NEXT_PUBLIC_SUPABASE_URL
- NEXT_PUBLIC_SUPABASE_ANON_KEY
- SUPABASE_SERVICE_ROLE_KEY

## 계정 구조

- 관리자: seoulclinicpick@gmail.com (role: admin, clinic_id: null)
- 병원회원 (참의원): idcharm23@gmail.com (role: clinic_staff, clinic_id: 6)

## 참의원(Charm Clinic) 연계

- SCP 내 참의원 페이지: https://seoulcp.com/ko/clinics/6
- 참의원 자체 사이트: https://www.cclinic.kr
- 참의원 블로그: https://blog.naver.com/idcharm23
- 참의원 유튜브: https://www.youtube.com/@charmclinicseoul
