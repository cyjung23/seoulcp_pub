# S2-06: 버전별 변경 사항

**최종 갱신:** 2026-05-01

---

## v2.21 (2026-05-01) — GSC 404/Soft 404 대응 + i18n 정체성 정렬 + 빈 클리닉 noindex

### 진단 (오늘 작업의 시작점)
- GSC "페이지 색인 생성" 점검: 404 10건, Soft 404 2건, 서버 오류(5xx) 21건, 리디렉션 1187건, 크롤링됨-색인 안 됨 115건 확인
- 5xx 21건: Vercel Logs 분석 결과 모두 일시적 오류로 자연 해소 완료 (URL 검사로 1375 등 정상 색인 확인)
- 403 응답: Vercel Middleware에서 Bytespider 등 악성봇 차단 정상 동작 (SEC-001)
- 4/27~4/28 발생한 5xx는 일시적 함수 오류로 추정, 코드 변경 없이 GSC 유효성 검사로 종결 예정

### DATA-001: 클리닉 2225/2226 오염 데이터 정리
- 윤병일성형외과의원(2225), 강남우태하피부과의원(2226)의 website 필드에 가비아 errdoc URL이 저장되어 있던 문제
- 크롤링 시점에 클리닉 홈페이지가 이미 죽어있어, 가비아의 404 안내 페이지를 크롤링한 결과가 그대로 DB에 저장됨
- description 필드에도 "404 Not Found" 텍스트가 들어가 있었음
- Supabase UPDATE로 두 클리닉의 website, description NULL 처리
- Soft 404 분류 원인 1차 차단

### SEO-007: CONCERN_SLUG_MAP 오타 수정
- "넓은 광대/볼살" → "wide-cheekbonescheeks" (오타) → "wide-cheekbones-cheeks" (수정)
- DB의 정상 slug `wide-cheekbones-cheeks`와 매핑 일치
- 영향 URL: /ko/concerns/넓은 광대/볼살, /en/concerns/넓은 광대/볼살, /concerns/넓은 광대/볼살 (404 → 정상)
- **커밋**: 22bae85

### SEO-008: 심술보 → jowl-sagging 슬러그 분리
- concerns 테이블에 marionette-lines slug가 두 행(id=60 입가 처짐, id=162 심술보)에 중복 존재 → 의미 분리 필요
- 의학적 분류 결정 (참클리닉 원장 자문):
  - 입가 처짐 (id=60) = marionette-lines (입가의 깊은 주름)
  - 심술보 (id=162) = jowl-sagging (턱선 처짐)
  - mouth-corner-sagging 케이스는 심술보로 통합
- DB UPDATE: id=162의 slug, name_en, name_ja, name_zh 변경
  - slug: marionette-lines → jowl-sagging
  - name_en: Marionette Lines → Jowl Sagging
  - name_ja: マリオネットライン → ブルドッグライン
  - name_zh: 木偶纹 → 嘴边肉下垂
- CONCERN_SLUG_MAP에서 "심술보" 매핑도 jowl-sagging으로 변경
- **커밋**: 69c7cf9

### SEO-009: defaultLocale ko → en 변경 (SCP 정체성 정렬)
- src/i18n/routing.ts: defaultLocale "ko" → "en"
- 사유: SCP는 해외 사용자 대상 플랫폼이며 영어가 표준 페이지 (한국어 페이지는 크롤링 자료가 한글이라 먼저 만든 것)
- 영향: 브라우저 언어가 ko/en/ja/zh 외인 사용자(예: 프랑스어, 독일어) → 한국어 대신 영어로 fallback
- 미들웨어 prefix 자동 보정 로직도 함께 추가했으나 Next.js 16 middleware deprecation 영향으로 작동하지 않음 → WO-033으로 이월
- **커밋**: 5ff5212

### SEO-010: 빈 클리닉 자동 noindex 처리
- 배경: 2,727개 클리닉 중 787개(약 29%)가 시술·장비 모두 0개 → Soft 404 잠재 위험
- 원인: 클리닉 홈페이지 크롤링 시 데이터가 부실하거나 형식이 적절하지 않은 케이스 다수
- 구현: src/app/[locale]/clinics/[id]/page.tsx의 generateMetadata 함수에서 clinic_treatments, clinic_devices count 쿼리(head: true)로 빠르게 확인
- 둘 다 0이면 `robots: { index: false, follow: false }` 추가 → 검색엔진 자동 제외
- 정상 클리닉(시술/장비 보유)은 기존 동작 유지 (curl 검증으로 확인)
- 데이터 보강 시 자동으로 다시 인덱싱 가능 (DB 변경 없음, 코드만 수정)
- 검증: clinic 2226 → `<meta name="robots" content="noindex, nofollow"/>` 출력 확인, clinic 1375(정상) → robots 메타 없음
- **커밋**: 6ee6624

### 후속 작업 등록
- WO-033: middleware.ts → proxy.ts 마이그레이션 (Next.js 16 deprecation 대응) + locale prefix 자동보정 디버깅
- WO-034: GSC 유효성 검사 트리거 및 모니터링 (5xx/404/Soft 404 정리 확인, 5/2 MON-003과 함께 처리)
- WO-035: 외부 API 기반 빈 클리닉 데이터 자동 보강 (네이버 플레이스 + 건강보험심사평가원 API 등)
- WO-037: changelog 분리 + 인수인계 효율화 (옵션 A: 활성/아카이브 분리 + 인덱스 신설 + 자동 롤오버 규칙)

### 자연 정리 항목 (별도 조치 없음)
- /ja/treatments/시크릿: standard_treatments 테이블에 "시크릿" 관련 데이터 자체가 없어 매핑 불가, 외부 죽은 링크로 추정 → GSC가 시간 지나면 자연 정리
- locale prefix 누락 URL (/concerns/입가 처짐 등 3건): 미들웨어 자동 보정이 작동 안 했으나, 한국 브라우저 사용자는 next-intl 자동 감지로 정상 페이지 도달, GSC 자연 정리 예상

---

## v2.20 (2026-04-25) — SEO 브랜드 강화 + 도메인 리다이렉트 + 모니터링 2차

### SEO-005: seoulclinicpick.com 301 리다이렉트
- seoulclinicpick.com 도메인 구매 (가비아)
- Vercel 도메인 설정: seoulclinicpick.com + www.seoulclinicpick.com → seoulcp.com 301 리다이렉트
- 가비아 DNS: A @ → 76.76.21.21 (Vercel), CNAME www → cname.vercel-dns.com
- 목적: Google 검색에서 "seoul clinic pick" 입력 시 브랜드 인식 강화
- Cloudflare 미사용 확인 → 기술스택에서 Cloudflare 삭제

### SEO-006: Organization JSON-LD 추가
- src/app/layout.tsx `<head>`에 JSON-LD 구조화 데이터 삽입
- @type: Organization, name: "Seoul Clinic Pick"
- alternateName: ["서울클리닉픽", "SeoulClinicPick", "SCP", "seoulcp"]
- sameAs: [seoulclinicpick.com, YouTube, Naver 블로그]
- Google이 "Seoul Clinic Pick"을 브랜드명으로 인식하도록 유도
- **커밋**: 0f12fcf

### DX-001: VS Code CSS 경고 비활성화

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- .vscode/settings.json 생성: css.validate=false, unknownAtRules=ignore

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Tailwind CSS v4 `@theme inline` 문법에 대한 VS Code 노란색 경고 해소

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **커밋**: e8c6f92

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### MON-001: Naver/Google 인덱싱 2차 확인 (4/25)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Google 인덱싱 (정상)**

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - /ja/: 인덱싱 확인 (wiki/agnes, wiki/facelift, wiki/rejuran, treatments 등)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - /zh/: 인덱싱 확인 (메인, wiki/ellanse, wiki/warts, wiki/juvelook, treatments 등)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - /en/: 인덱싱 확인 (treatments, wiki/facelift, treatments/agnes 등)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - /ko/: 인덱싱 확인 (clinics 다수 노출)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - → Google 4개 언어 모두 정상 인덱싱

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Naver 인덱싱**

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - /ko/: 인덱싱 확인 (site:seoulcp.com 결과 존재)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - /ja/: 0건 (미인덱싱)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - /zh/: 0건 (미인덱싱)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - → Naver는 한국어 콘텐츠 중심 검색엔진, 외국어 페이지 인덱싱 우선순위 극히 낮음

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **전략 전환**: Naver는 /ko/ 인덱싱에 집중, /ja/·/zh/는 Google/Bing에서 관리

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### MON-002: GEO 효과 분석 (4/18~4/25)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **국가별 사용자 (4/18~4/24, 7일)**

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - South Korea: 29명 (72.5%), 참여율 90.3%

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - Japan: 3명, Portugal: 3명, United States: 3명

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - Canada: 1명, Saudi Arabia: 1명

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - → 6개국에서 유입 발생, 다국어 SEO 작동 확인

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **트래픽 채널**

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - Organic Search: 31세션 (62%), 참여율 90.3% — 핵심 채널

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - Unassigned: 10세션 (참여 시간 92.4초, 봇/크롤러 혼재 추정)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - Direct: 8세션, Referral: 1세션

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Reddit 포스팅 효과 (4/24 포스팅, r/KoreaSeoulBeauty 28,121구독자)**

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - 4/24: 미국 1명 → 4/25: 미국 10명 (신규 9명) — 즉시 유입 효과 확인

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
  - 단, 참여율 20%, 주요 이벤트 0건 — Reddit 특성상 탐색 후 이탈 패턴

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **판단**: 서비스 출시 초기로 트래픽 규모 작음 (일평균 ~7세션), GEO 효과 확정은 이르나 다국어 인덱싱 + 해외 유입 발생은 긍정 신호

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **다음 확인**: MON-003 (5/2) — 2주 후 동일 데이터 추세 비교

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.19 (2026-04-20) — WO-030 파비콘 교체 + OG이미지 업데이트

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 파비콘 교체

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 기존 동적 생성(icon.tsx, apple-icon.tsx) 삭제 → 정적 파비콘으로 전환

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- SCP 3색 파비콘 적용: S(#4AE0FF) C(#FF5CAE) P(#A0E85A), 배경 #1E272E

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 적용 파일: favicon.ico, favicon-16x16.png, favicon-32x32.png, apple-touch-icon.png, android-chrome-192x192.png, android-chrome-512x512.png

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- site.webmanifest 생성: name "Seoul Clinic Pick", short_name "SCP", theme_color/background_color #1E272E

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- layout.tsx에 icons + manifest 메타데이터 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### OG이미지 업데이트

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- opengraph-image.tsx: 밝은 배경 → 다크 배경(#1E272E) + SCP 3색 로고 + 태그라인

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- SNS 공유 시 새 브랜드 디자인 반영

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 디자인 리포트

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 마케팅1팀과 협업: 통합 디자인 리포트 v3 작성 (색상 확정, 파비콘 스펙, 산출물 체크리스트)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 로고 색상 확정: 현재 사이트 색상 유지, 다크 배경 단일 전략

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 파비콘 스펙 확정: SCP 3글자, Black(900), 비율 0.50, 자간 -0.06em

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.18 (2026-04-19) — Bing Webmaster Tools 등록 + Baidu 보류

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### Bing Webmaster Tools

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Google Search Console import 방식으로 cclinic.kr + seoulcp.com 동시 등록

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Microsoft 계정 → GSC idcharm23 계정 연동, 별도 사이트 인증 불필요

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- seoulcp.com/sitemap.xml 제출 완료 (ko/en/ja/zh 4개 언어, 12,600+ URL)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- cclinic.kr/sitemap.xml 제출 완료 (영문사이트 포함)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Bing 검색엔진 + Microsoft Copilot + Edge 브라우저 노출 기대

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### Baidu 站长平台 (보류)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 해외 사용자 등록 페이지(passport.baidu.com/v2/?reg&overseas=1) 일시 중단

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 재개 시기 미정, 재개 확인 후 등록 진행 예정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- GFW 테스트 완료: seoulcp.com 중국 5개 도시에서 접속 가능 확인 (Beijing, Shenzhen, Inner Mongolia, Heilongjiang, Yunnan)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 일본 검색엔진 현황

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Google Japan 82% + Yahoo! Japan 9% (Google 엔진 사용) → GSC 등록으로 커버

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Bing Japan 8~9% → Bing Webmaster Tools 등록 완료

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 일본 검색엔진 3사 전체 커버 완료

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.17 (2026-04-18) — WO-029 hreflang HTML + 검색/클리닉 최종 수정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### SEO — HTML hreflang 태그 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- layout.tsx에 alternates 메타데이터 추가 (canonical + 4개 언어 + x-default)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 9개 페이지 일괄 적용: layout, page, wiki, wiki/[slug], clinics/[id], treatments, treatments/[slug], concerns, concerns/[slug]

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Google Search Console 라이브 URL 테스트로 hreflang 6개 태그 인식 확인

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 검색 (search/page.tsx)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- encyclopedia 테이블 검색 추가 (title_zh/ko/en/ja 검색)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- encyclopedia 컬럼명 수정 (summary_ko → summary)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 백과사전 검색 카드 UI 통일 (grid 3열, 밝은 배경, 보라색 hover)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- SectionHeader에 isZh 카운트 표시 추가 ("条")

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- totalCount 중국어 표시 ("共N条结果")

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 클리닉 상세 (clinics/[id]/page.tsx)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 기본정보 헤딩 버그 수정 (isZh 분기 중복 제거)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 주소/specialties isZh 분기 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 영업시간 요일 다국어 변환 함수 translateHours() 추가 (zh/ja/en)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 메인페이지 (page.tsx)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 고민카드 styles 변수에 isZh 분기 추가 (아이콘/스타일 복원)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 서브타이틀 중국어 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 인덱싱 요청

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Google Search Console: sitemap.xml 재제출, /ko/ /ja/ /zh/ 색인 생성 요청

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Naver Search Advisor: sitemap.xml 재제출, /ko/ /ja/ /zh/ 웹 페이지 수집 요청

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 커밋

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 1f433a9 메인페이지 고민카드 스타일 복원

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 05850ef 검색 zh 지원 (name_zh + encyclopedia)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 2098a96 clinics 기본정보/주소/specialties isZh

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 565055d 영업시간 요일 다국어 변환

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 686746b HTML head hreflang alternates 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 전체 페이지 hreflang 4개 언어 + x-default 일괄 적용

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.16 (2026-04-18) — WO-029 중국어 품질개선 7~8차

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 메인페이지

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 고민카드 스타일 복원: `styles` 변수에 `isZh` 분기 추가 (zh → CATEGORY_STYLES_EN)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 서브타이틀 중국어 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 검색 (search/page.tsx)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- encyclopedia 테이블 검색 추가 (title_zh, title_ko, title_en, title_ja)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- concerns name_zh 검색 조건 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- encyclopedia 컬럼명 수정 (summary_ko → summary)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- SectionHeader isZh 카운트 표시 ("条")

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 백과사전 검색 카드 스타일 통일 (grid 3열, 밝은 배경, 보라색 hover)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- totalCount 중국어 표시 ("共N条结果")

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 클리닉 상세 (clinics/[id]/page.tsx)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 기본정보 헤딩 버그 수정 (isZh 분기 중복 제거)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 주소 isZh 분기 추가 (영문 fallback)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- specialties isZh 분기 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 영업시간 요일 다국어 변환 함수 `translateHours()` 추가 (zh/ja/en)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 영업시간 라벨 다국어 추가 ("营业时间:" / "営業時間:" / "Hours:")

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 커밋

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 1f433a9 메인페이지 고민카드 스타일 복원

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 05850ef 검색 zh 지원 (name_zh + encyclopedia)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 2098a96 clinics 기본정보 헤딩/주소/specialties isZh

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 565055d 영업시간 요일 다국어 변환

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 검색 encyclopedia 컬럼명 수정 + 카드 UI 개선

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.15 중국어(zh) 사이트 확장 (2026-04-18)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### WO-029: 중국어 사이트 확장 (Phase 1~6 전량 완료)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 1 (i18n 인프라):** routing.ts, middleware.ts에 zh 로케일 추가, messages/zh.json 생성, LanguageToggle EN/KO/JA/ZH 4언어 스위처, NavLinks·NavMobileMenu zh 메뉴 라벨, Footer·MedicalDisclaimer zh 텍스트

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 2 (DB):** encyclopedia 10개 zh 컬럼(title_zh, summary_zh, overview_zh + 상세 7필드), concerns name_zh, standard_treatments name_zh 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 3 (페이지 적용):** 18개 페이지 isZh 분기, 11개 페이지 텍스트 분기, wiki/[slug] 하위 컴포넌트 3개(RelatedClinics, DeviceRelatedClinics, DeviceRelatedTreatments) zh 적용

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 4 (번역):** standard_treatments 120개 name_zh, concerns 135개 name_zh, encyclopedia 159개 title_zh/summary_zh/overview_zh, encyclopedia 159개 × 7상세필드(mechanism_zh, effects_zh, duration_zh, recovery_zh, side_effects_zh, price_range_zh, target_audience_zh) — 기획2팀 협업, 전량 완료

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 5 (SEO):** sitemap.ts /zh/ hreflang 추가, layout.tsx zh_CN locale 메타

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 6 (검증):** 3차 품질개선, /zh/wiki·concerns·treatments 전 페이지 중국어 표시 확인 완료

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **번역 현황:** 414항목 기본 + 1,113필드 상세 = 전량 완료

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **폴백 순서:** _zh → _en → ko (번역 없으면 영어, 영어도 없으면 한국어)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **주요 커밋:** 78d9fc2(Phase 1), b9379bf(Phase 2-3-5), f9afb31(품질개선 1차), de60721(Phase 4-2)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
---

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.14 — 일본어 사이트 품질 개선 + 언어 스위처 + concerns 영문 slug (2026-04-17)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 언어 전환 UI

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 3언어 스위처 (EN/KO/JA) 구현 — LanguageToggle, NavLinks, NavMobileMenu, Footer

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 순서: EN / KO / JA (원장님 요청)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Footer 클라이언트 컴포넌트 분리 (locale 기반 텍스트: 病院関係者·利用規約·プライバシーポリシー)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 일본어 품질 개선 (6개 이슈 해결)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 클리닉 카드: ClinicListWithFilter ja 대응 — 클리닉명·주소·지역 영어 표시, 지도/経路/もっと見る 일본어

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 백과사전 안내사항: MedicalDisclaimer 일본어 (ご案内 + 본문 + 情報修正のご依頼)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 고민 카드: DB concerns.name_ja 135개 번역 추가, 카드에 일본어명 우선 표시

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- concerns/[slug] 시술카드: name_en 우선, clinicCount 일본어 (件のクリニック)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- devices/[slug] 클리닉카드: address_en 우선 표시

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- RelatedClinics / DeviceRelatedClinics: title 일본어

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### concerns 영문 slug URL

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- DB concerns.slug 컬럼 추가, 135개 영문 slug 자동 생성 (name_en 기반)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 전체 링크 name_ko → slug 전환 (concerns, search, treatments, sitemap)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- concerns/[slug] slug 우선 조회 + name_ko 폴백 (기존 한글 URL 호환)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 예: /ja/concerns/눈두덩이살 → /ja/concerns/eyelid-fat

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### DB 변경

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- concerns.name_ja 컬럼 추가 + 135개 일본어 번역

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- concerns.slug 컬럼 추가 + 135개 영문 slug 생성

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 커밋 (8건)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 4ea281a feat: 언어 전환 UI 3언어 스위처

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- c0f7357 feat: WO-028 Phase 5 sitemap hreflang

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 6253556 fix: MedicalDisclaimer 일본어, 고민카드 name_ja

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 85aa119 fix: ClinicListWithFilter/RelatedClinics ja 대응

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 171e87d fix: concerns/[slug] clinicCount, devices/[slug] 주소 영어

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 8278c4a fix: concerns/[slug] 시술카드 영어표시

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 9a1f95e feat: concerns 영문 slug URL 전환

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.13 (2026-04-17)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### WO-028: 일본어 사이트 확장 (Phase 1~6 완료)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 1 (i18n 인프라):** routing.ts, middleware.ts, request.ts, layout.tsx에 ja 로케일 추가, messages/ja.json 생성

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 2 (DB):** standard_treatments.name_ja 120개, concerns.concern_group_ja 13개, encyclopedia 일본어 컬럼 10개 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 3 (로케일 로직):** 소비자 페이지 14개 파일 isEn→isJa/isEn/ko 3항 구조 전환, locale 판정 로직 6개 파일 수정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 4 (번역):** 시술명 카타카나 120개, 고민 카테고리 13개, 메인 페이지 UI 텍스트, 섹션 타이틀 일본어 번역

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 5 (SEO):** sitemap.xml에 /ja/ hreflang 추가 (ko/en/ja 3개 언어 alternates)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **Phase 6 (검증):** /ja 메인, /ja/treatments/botox, sitemap.xml 정상 확인

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **보류:** WO-028-B partner 페이지 일본어 (9개 파일 120개소, 일본인 파트너 유입 시 진행)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **완료:** encyclopedia 본문 일본어 번역 159항목 (device 47 + surgery 38 + treatment 74) 전량 DB 반영

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### 기타 수정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- treatments 목록 "Coming soon" 버그 수정 (treatments 테이블에 없는 slug 컬럼 select 제거)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- StandardTreatment 인터페이스에 name_ja 추가 (treatments/page.tsx, surgeries/page.tsx)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- wiki/page.tsx categoryConfig에 labelJa 추가, select에 title_ja/summary_ja 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.12 (2026-04-17) — 일본어 사이트 인프라 + 데이터 (WO-028 Phase 1,2,4)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- i18n 인프라: routing.ts, middleware.ts, request.ts, layout.tsx에 ja 로케일 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- messages/ja.json 생성 (nav, common, home 일본어 UI 텍스트)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- standard_treatments 테이블에 name_ja 컬럼 추가, 120개 시술 카타카나 번역 입력

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- concerns 테이블에 concern_group_ja 컬럼 추가, 13개 고민 카테고리 일본어 번역 입력

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 메인 페이지 UI 텍스트 일본어 번역 확보 (코드 반영은 Phase 3에서)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 용어 선택: エイジングケア(안티에이징), 美容辞典(백과사전), 美容整形(수술), スキン(피부 뱃지)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- /ja/ 경로 접속 가능 확인 (콘텐츠는 Phase 3 이후 일본어 표시)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- treatments 목록 Coming soon 버그 수정 (treatments 테이블에 없는 slug 컬럼 select 제거)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.11 (2026-04-17) — 시술 URL 영문 slug 전환 (WO-027)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- standard_treatments 테이블에 slug 컬럼 추가 (TEXT UNIQUE)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- name_en 기반 120개 시술 slug 자동 생성 (소문자, 하이픈 변환, 특수문자 제거)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 용어 구분 준수: 지방파괴주사→adipocytolysis, 지방분해주사→fat-dissolving

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- ellansé → ellanse 수동 보정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- treatments/[slug]/page.tsx: slug 우선 조회, name_ko fallback, 301 리다이렉트

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 내부 링크 교체 (10개 파일, 14개소): page.tsx, treatments/page.tsx, clinics/[id]/page.tsx, concerns/[slug]/page.tsx, surgeries/page.tsx, search/page.tsx, wiki/[slug]/page.tsx, DeviceRelatedTreatments.tsx, beauty/mypage/page.tsx, sitemap.ts

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 각 파일 interface/type에 slug 필드 추가, select 쿼리에 slug 포함

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- concerns/[slug]/page.tsx: standard_treatments에서 slugMap 조회 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- sitemap.xml 전체 영문 slug 기반 URL 출력 확인

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- GSC + Naver Search Advisor sitemap 재제출 완료

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 검증: /en/treatments/botox, /en/treatments/onda, /ko/treatments/filler, 한글 URL 리다이렉트 정상

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.10 (2026-04-17) — 메인 페이지 디자인 개선 (WO-026)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Quick Links: 글래스모피즘 카드 + 상단 컬러 바 + 3칸 배열, 배경색 #f5fbf9

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Popular Treatments: Compact Strip 카드 (52px 아이콘바) + 3칸 배열, 그라데이션 배경

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 시술 9개 표시 (필러, 보톡스, 실리프팅, 울쎄라, 리쥬란, 제모, 써마지, 지방이식, 지방흡입)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 카테고리 뱃지 (Face/Body/Skin/Anti-aging), 클리닉 수, Compare 링크

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- max-w-7xl 폭 통일 (Browse by Concern과 동일), 폰트 크기 상향

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 3개 섹션 배경 구분: Browse by Concern(흰색) → Quick Links(#f5fbf9) → Popular Treatments(그라데이션)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- globals.css에 glass-card-ql, pt-card-glass, pt-strip CSS 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.9 (2026-04-17) — 뷰티회원제 Phase 1+2+3

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- beauty_accounts 테이블 생성 (user_id, nickname, preferred_language, RLS)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- user_picks 테이블 생성 (target_type TEXT, target_id TEXT, Optimistic Update)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 뷰티회원 가입 (/beauty/signup), 로그인 (/beauty/login) 페이지

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 비밀번호 재설정 (/beauty/reset-password, /beauty/update-password)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- API route: /api/beauty/register (service_role), /api/picks (GET/POST 토글)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- PickButton 공통 컴포넌트 (Optimistic Update, 클릭 즉시 UI 반영)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 클리닉 상세, 시술 상세, 백과사전 상세 페이지에 Pick 버튼 적용

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 비로그인 시 Pick 클릭 → 로그인 페이지 안내

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 뷰티회원 테스트 계정: cyjung23@gmail.com (beauty)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 마이페이지 (/beauty/mypage): My Picks 탭(병원/시술/백과사전), 프로필 수정, 비밀번호 변경, 회원 탈퇴

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- API routes: /api/beauty/account (GET/PATCH/DELETE), /api/picks/name (Pick 항목 이름 조회)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- AuthNav 컴포넌트: 네비게이션에 로그인 상태 표시 (비로그인: Log In/Sign Up, 로그인: 닉네임/Log Out)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 데스크톱/모바일 네비게이션 모두 적용

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 로그인 후 페이지 새로고침으로 AuthNav 즉시 갱신

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.8 (2026-04-16) — 병원 파트너 회원제 Phase 2

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 관리자 패널: 가입 승인/거절/정지 기능 (/partner/admin)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 수정 요청 검토: 병원 정보 + 시술 정보 변경 내역 상세 표시, 승인 시 DB 자동 반영 (/partner/admin/submissions)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 병원 정보 수정 폼: 병원명, 전화, 웹사이트, 진료시간, 소개 수정 (/partner/edit-clinic)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 시술 정보 수정 폼: 가격, 가격 안내(price_note), 설명 수정 (/partner/edit-treatments)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- clinic_treatments 테이블에 price_note 컬럼 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 소비자 페이지에 price_note 표시 (클리닉 상세, 시술 목록, 클리닉 카드)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 관리자 계정 cyjung23 → seoulclinicpick@gmail.com 으로 변경

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- API routes: /api/partner/submission, /api/partner/admin/accounts, /api/partner/admin/submissions

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.7 (2026-04-16) — 병원 파트너 회원제 Phase 1

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Supabase Auth 설정 (Email, URL Configuration, Confirm email)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- clinic_accounts 테이블 생성 (user_id, clinic_id nullable, role, status, RLS)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- clinic_submissions 테이블 생성 (수정요청 저장용, RLS)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 회원가입 (/partner/signup), 로그인 (/partner/login) 페이지

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 비밀번호 재설정 (/partner/reset-password, /partner/update-password)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 파트너 대시보드 (/partner/dashboard) — 관리자/병원회원 메뉴 분리

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- API route (/api/partner/register) — service_role로 RLS 우회 등록

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Vercel 환경변수 SUPABASE_SERVICE_ROLE_KEY 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 푸터에 병원 관계자, 이용약관, 개인정보처리방침 링크 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 계정: cyjung23@gmail.com (admin), idcharm23@gmail.com (clinic_staff, 참의원)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.6 (2026-04-15) — 데이터 보정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 닥터홈즈의원 지방파괴주사(얼굴) 매핑 삭제 (id=33578)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 튼살주사 → 참의원 전용 시술로 변경: 47개 클리닉을 튼살레이저로 이동

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 클리닉 목록 priority 정렬 적용 (clinic_treatments.priority 전달)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 참의원 전체 시술 priority=1 설정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.5 (2026-04-15) — 구글맵 연동

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- clinics 테이블에 latitude, longitude 컬럼 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- Kakao 로컬 API로 2,725/2,727건 좌표 변환 완료 (99.9%)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 클리닉 상세 페이지: "지도에서 보기", "길찾기" 구글맵 버튼 추가

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 클리닉 목록 카드: "지도", "길찾기" 버튼 추가 (treatments, devices 페이지)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 좌표 없는 2건(ID 4, 12)은 주소 NULL로 버튼 미표시

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.4 (2026-04-15) — 인프라 변경

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 표준문서 저장소: pubrepo/seoulcp-docs/ → seoulcp_pub/docs/ 이관 완료

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- GitHub PAT 토큰: All repositories 권한으로 재발급

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- FAQ Schema 빈 text 필드 수정 배포 확인 (커밋 6ff2e28)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 팀 구성 확정: 기획1/2팀, 마케팅1/2팀 (DEC-050)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 구글맵 연동 우선 착수 결정 (DEC-051)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.3 ~ v2.0

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
(이전 내용 동일 — 생략)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
---

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.19 (2026-04-21) — 크롤링 방어 적용

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### SEC-001: 크롤링 방어

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
**robots.txt 강화**

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 기존: `User-Agent: * → Allow: /` (전체 허용)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 변경: 검색엔진 3종(Googlebot, Yeti, Bingbot) + GEO/AI봇 6종(GPTBot, Google-Extended, ChatGPT-User, PerplexityBot, ClaudeBot, Amazonbot)만 허용

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 나머지 모든 봇 `Disallow: /` 차단

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- `/api/`, `/partner/` 경로는 허용 봇에게도 차단

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
**Rate Limiting 미들웨어 (src/middleware.ts 통합)**

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 악성 봇 21종 User-Agent 기반 즉시 403 차단 (AhrefsBot, SemrushBot, MJ12bot, DotBot, Scrapy, python-requests 등)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- IP당 60 req/min Rate Limit, 초과 시 5분간 429 차단

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 보안 헤더 추가: X-Robots-Tag: noarchive, X-Content-Type-Options: nosniff, X-Frame-Options: DENY

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 기존 i18n 라우팅 + concerns 리다이렉트 정상 유지

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
**커밋**: eb7e50d → 58e88e2

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
---

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
## v2.19 (2026-04-21) — 성능 개선

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### PERF-001: wiki 페이지 DB 쿼리 병렬화

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 순차 실행 8~10회 쿼리를 Promise.all 3개 그룹으로 병렬화

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 그룹1: treatment_device_map + treatments

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 그룹2: devices + encyclopedia_device_map

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 그룹3: clinic_treatments 경로1 + 경로2

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- **커밋**: 7475a0a

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유


### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
### PERF-002: 서버 측 페이지네이션 (대기 등록)

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
- 유입량 증가 시 클리닉 목록을 API route로 분리하여 서버 측 페이지네이션 적용 예정

### WO-031: 관리자 대시보드 pending 건수 뱃지
- 병원회원 가입 승인 요청(clinic_accounts status=pending) 건수 표시
- 병원 정보 수정 요청(clinic_submissions status=pending) 건수 표시
- Promise.all로 두 테이블 병렬 조회
- 건수 1 이상 시 빨간색 깜빡이는 뱃지 (text-red-500 animate-pulse)
- 건수 0이면 뱃지 미표시
- 참의원 병원계정으로 테스트 완료
- **커밋**: f0f97d5

### MKT-001: 외부 마케팅 포스팅 (마케팅1팀 협업)
- Reddit r/KoreaSeoulBeauty (영어, 4/24) — 포스팅 12시간 내 미국 신규 9명 유입
- Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
- note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- Vercel Hobby 플랜 한도 확인: 일일 ~1,300~2,100명까지 무료 운영 가능, 현재 일평균 7세션으로 여유
