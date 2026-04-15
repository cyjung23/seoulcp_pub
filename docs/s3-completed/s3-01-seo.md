# S3-01: SEO 시리즈 (SEO-001 ~ SEO-005)

**최종 갱신:** 2026-04-10

## SEO-001 — 사이트 기본 SEO 인프라 구축
- 기간: 2026-03-30 ~ 2026-03-31. 상태: 완료.
- sitemap.xml 동적 생성(3,131 URLs, hreflang alternates, /ko/ /en/ locale prefix)
- robots.txt, 전역 메타데이터(title, description, canonical, OG), 각 페이지 동적 title/description
- JSON-LD: WebSite, Article, MedicalBusiness 3유형
- favicon, apple-icon, OG 이미지 동적 생성
- 변경 파일: sitemap.ts, robots.ts, layout.tsx, page.tsx, opengraph-image.tsx 등 9개+

## SEO-002 — Google Search Console 등록
- 작업일: 2026-03-31. 상태: 완료.
- 사이트 등록, HTML 메타 태그 소유권 인증, sitemap 제출, 주요 5개 URL 색인 요청
- 2026-04-07 기준 약 3,125개 URL 색인 확인
- 2026-04-10 sitemap 재제출 및 주요 URL 재색인

## SEO-003 — GA4 설치
- 작업일: 2026-03-31. 상태: 완료.
- GA4 (G-6CX6ER3EX6), layout.tsx에 next/script 삽입, 산업 카테고리 "건강"

## SEO-004 — Naver Search Advisor 등록
- 작업일: 2026-03-31. 상태: 완료.
- 사이트 등록, 메타 태그 소유권 인증, sitemap 제출
- 2026-04-07 기준 색인 1페이지만 확인 (CSR 방식으로 Yeti 본문 수집 불가)
- 2026-04-10 웹 페이지 수집 요청 20개 URL 제출

## SEO-005 — Naver Analytics 설치
- 작업일: 2026-04-01. 상태: 완료.
- 추적 ID wa:1632b4d20412f40, wcslog.js, strategy="afterInteractive"
