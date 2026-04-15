# S1-04: 코드 파일 구조

**최종 갱신:** 2026-04-11

## 주요 파일 경로

    src/
    ├── app/
    │   ├── [locale]/
    │   │   ├── page.tsx                    ← 홈
    │   │   ├── clinics/
    │   │   │   ├── [id]/page.tsx           ← 클리닉 상세 (3경로 매핑, JSON-LD)
    │   │   │   └── layout.tsx
    │   │   ├── treatments/[slug]/page.tsx  ← 시술 상세 (std_id 조회 추가)
    │   │   ├── concerns/[slug]/page.tsx    ← 고민 상세
    │   │   ├── devices/[slug]/page.tsx     ← 장비 상세
    │   │   ├── wiki/[slug]/page.tsx        ← 백과사전 상세
    │   │   ├── search/page.tsx             ← 검색
    │   │   └── surgeries/page.tsx          ← 수술 목록
    │   ├── sitemap.ts                      ← 동적 sitemap (3,131+ URLs)
    │   └── robots.ts                       ← robots.txt (AI 크롤러 허용)
    ├── components/
    │   ├── ClinicListWithFilter.tsx         ← 클리닉 목록 (name_en 지원)
    │   ├── NavLinks.tsx
    │   ├── NavMobileMenu.tsx
    │   ├── MedicalDisclaimer.tsx
    │   └── ...
    ├── i18n/
    │   ├── routing.ts
    │   └── request.ts
    ├── messages/
    │   ├── ko.json
    │   └── en.json
    └── middleware.ts                        ← 레거시 리다이렉트

## 최근 수정 파일 (2026-04-11, WO-021)

- src/app/[locale]/clinics/[id]/page.tsx — 3경로 매핑, JSON-LD MedicalBusiness
- src/app/[locale]/treatments/[slug]/page.tsx — standard_treatment_id 클리닉 조회, name_en
- src/components/ClinicListWithFilter.tsx — locale 기반 name_en 표시
