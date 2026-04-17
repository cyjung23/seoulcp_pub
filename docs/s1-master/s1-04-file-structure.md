# S1-04: 코드 파일 구조

**최종 갱신:** 2026-04-16

## 주요 파일 경로

    src/
    ├── app/
    │   ├── [locale]/
    │   │   ├── page.tsx                              ← 홈
    │   │   ├── clinics/
    │   │   │   ├── [id]/page.tsx                     ← 클리닉 상세 (3경로 매핑, JSON-LD, price_note)
    │   │   │   └── layout.tsx
    │   │   ├── treatments/[slug]/page.tsx            ← 시술 상세 (std_id 조회, price_note)
    │   │   ├── concerns/[slug]/page.tsx              ← 고민 상세
    │   │   ├── devices/[slug]/page.tsx               ← 장비 상세
    │   │   ├── wiki/[slug]/page.tsx                  ← 백과사전 상세
    │   │   ├── search/page.tsx                       ← 검색
    │   │   ├── surgeries/page.tsx                    ← 수술 목록
    │   │   ├── terms/page.tsx                        ← 이용약관 (v2.8)
    │   │   ├── privacy/page.tsx                      ← 개인정보처리방침 (v2.8)
    │   │   └── partner/                              ← 병원 파트너 회원제 (v2.7~v2.8)
    │   │       └── beauty/
    │   │           └── register/route.ts             ← 뷰티회원 가입 API
    │   │       └── picks/
    │   │           └── route.ts                      ← Pick API (GET/POST)
    │   │       ├── signup/page.tsx                   ← 회원가입
    │   │       ├── login/page.tsx                    ← 로그인
    │   │       ├── reset-password/page.tsx           ← 비밀번호 재설정
    │   │       ├── update-password/page.tsx          ← 비밀번호 변경
    │   │       ├── dashboard/page.tsx                ← 파트너 대시보드 (관리자/병원회원 분리)
    │   │       ├── edit-clinic/page.tsx              ← 병원 정보 수정 폼
    │   │       ├── edit-treatments/page.tsx          ← 시술 정보 수정/추가 폼
    │   │       └── admin/
    │   │   └── beauty/                              ← 뷰티회원제 (v2.9)
    │   │       ├── signup/page.tsx                   ← 뷰티회원 가입
    │   │       ├── login/page.tsx                    ← 뷰티회원 로그인
    │   │       ├── reset-password/page.tsx           ← 비밀번호 재설정
    │   │       └── update-password/page.tsx          ← 비밀번호 변경
    │   │       └── mypage/page.tsx                  ← 뷰티회원 마이페이지
    │   │           ├── page.tsx                      ← 관리자 패널 (가입 승인/거절/정지)
    │   │           └── submissions/page.tsx          ← 수정 요청 검토
    │   ├── api/
    │   │   └── partner/
    │   │       └── beauty/
    │   │           └── register/route.ts             ← 뷰티회원 가입 API
    │   │       └── picks/
    │   │           └── route.ts                      ← Pick API (GET/POST)
    │   │       ├── register/route.ts                 ← 회원가입 API (service_role)
    │   │       ├── submission/route.ts               ← 수정요청 제출 API
    │   │       └── admin/
    │   │   └── beauty/                              ← 뷰티회원제 (v2.9)
    │   │       ├── signup/page.tsx                   ← 뷰티회원 가입
    │   │       ├── login/page.tsx                    ← 뷰티회원 로그인
    │   │       ├── reset-password/page.tsx           ← 비밀번호 재설정
    │   │       └── update-password/page.tsx          ← 비밀번호 변경
    │   │       └── mypage/page.tsx                  ← 뷰티회원 마이페이지
    │   │           ├── accounts/route.ts             ← 가입 승인/거절/정지 API
    │   │           └── submissions/route.ts          ← 수정요청 검토/승인/거절 API
    │   ├── sitemap.ts                                ← 동적 sitemap (3,131+ URLs)
    │   └── robots.ts                                 ← robots.txt (AI 크롤러 허용)
    ├── components/
    │   ├── ClinicListWithFilter.tsx                   ← 클리닉 목록 (name_en, price_note 지원)
    │   ├── NavLinks.tsx
    │   ├── NavMobileMenu.tsx
    │   ├── MedicalDisclaimer.tsx
    │   └── ...
    ├── lib/
    │   ├── supabase-browser.ts                       ← Supabase 브라우저 클라이언트 (v2.7)
    │   └── supabase-server.ts                        ← Supabase 서버 클라이언트 (v2.7)
    ├── i18n/
    │   ├── routing.ts
    │   └── request.ts
    ├── messages/
    │   ├── ko.json
    │   └── en.json
    └── middleware.ts                                  ← 레거시 리다이렉트

## 최근 수정 파일 (2026-04-16, 병원 파트너 회원제)

- src/app/[locale]/partner/ — 전체 신규 생성 (회원가입, 로그인, 대시보드, 수정 폼, 관리자 패널)
- src/app/api/partner/ — 전체 신규 생성 (register, submission, admin)
- src/lib/supabase-browser.ts, supabase-server.ts — 신규 생성
- src/app/[locale]/terms/, privacy/ — 약관, 개인정보 페이지 신규 생성
- src/components/ClinicListWithFilter.tsx — price_note 표시 추가
- src/app/[locale]/clinics/[id]/page.tsx — price_note 표시 추가
- src/app/[locale]/treatments/[slug]/page.tsx — price_note 표시 추가
