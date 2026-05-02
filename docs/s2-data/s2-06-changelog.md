# S2-06: 버전별 변경 사항 (활성)

**최종 갱신:** 2026-05-01
**범위:** v2.18 (2026-04-19) ~ 현재
**아카이브:** [v2.0 ~ v2.17 보기](./s2-06-changelog-archive-v2.0-v2.17.md)
**전체 인덱스:** [버전 인덱스 보기](./s2-06-changelog-index.md)

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

### SEO-011: zh.json 번역 누락 수정 (2026-05-02 후속 패치)
- 발견 경위: Vercel Function Logs에서 MISSING_MESSAGE 에러 3건 발견 (실제 iPhone Safari 사용자 발생, 02:04 UTC)
  - `Error: MISSING_MESSAGE: home (zh)` 2건
  - `Error: MISSING_MESSAGE: common.siteName (zh)` 1건
- 원인: zh.json이 ko/en/ja와 완전히 다른 키 구조를 가지고 있었음
  - zh.json에만 있던 legacy 키: `disclaimer.*`, `footer.partner/terms/privacy`, `common.loading/showMore/back/relatedClinics` 등
  - ko/en/ja 표준에 있는 `common.siteName`, `common.siteDescription`, `common.footer`, `common.contact`, `home.title`, `home.description` 모두 누락
  - generateMetadata에서 zh 페이지 렌더 시 SSR 500 발생
- 사용처 검증: legacy 키들은 src/ 하위에서 0건 참조 확인 (안전하게 제거 가능)
- 해결: zh.json을 ko/en/ja와 동일한 3섹션 표준 구조(nav + common + home)로 재작성
- 검증: 배포 후 iPhone Safari UA로 https://seoulcp.com/zh 접속 → HTTP 200 + `<title>按困扰搜索首尔美容诊所...| Seoul Clinic Pick</title>` 정상 출력
- **5/1 진단 일부 정정**: GSC 5xx 21건을 "일시적 → 자연 해결"로 결론냈으나, 실제로는 zh 사용자 접속 시 재현되던 버그였음. zh 트래픽이 적어 산발적으로 보였을 뿐. 본 수정으로 근본 원인 제거됨.
- **커밋**: d3e449a (rebase 후 해시, 원래 b203f33)
- 백업: /tmp/zh-backup.json (Codespaces 세션 한정)

### 403 응답 점검 (5/2 오전)
- 10:41~10:49 시간대 403 응답 3건 모두 동일 User-Agent: Bytespider (ByteDance 크롤러)
- SEC-001 BLOCKED_BOTS 리스트에 의한 정상 차단 — 조치 불필요
- 일반 사용자 오탐 사례 없음

### MON-003: GEO 추세 비교 (1주차 4/18~4/24 vs 2주차 4/25~5/1)

#### GA4 트래픽 소스 (2주차)
- Direct 165, Organic Search 57, Referral 3, Unassigned 3, Organic Social 1 — **합계 229 세션**
- 1주차 ~50 세션 대비 **+358% 폭증**
- Organic Search 31 → 57 (**+84%**) — MON-002 30% 성장 목표 초과 달성 ✅
- Organic Search 참여율 84.2% (47/57이 주요 이벤트 발생) — 매우 양호
- ⚠️ Direct 165는 참여율 26%로 낮음 — 봇 트래픽 혼입 가능성, 추가 조사 필요

#### GA4 GEO (활성 사용자 기준)
- 1주차: 한국 29, 일본 3, 포르투갈 3, 미국 3 (총 4개국)
- 2주차: 미국 **85** (1위), 한국 55, 프랑스 12, 캐나다 11, 독일 10, 사우디 6, 일본 3 외 신규 11개국 (총 **22개국**)
- **미국 28배 증가** (3 → 85): SCP 핵심 타겟인 해외 사용자 침투 시작
- 신규 GEO 출현: 프랑스, 캐나다, 독일, 사우디, 네덜란드, 호주, 홍콩, 아일랜드, 영국, 브라질, 핀란드, 조지아, 이탈리아, 뉴질랜드, 폴란드, 러시아, 싱가포르 등
- defaultLocale=en 변경(5/1) 직전 시점인데도 영어권 트래픽 압도적 → 5/1 변경 효과는 MON-004(5/8~5/9 예정)에 누적 측정

#### GSC 국가별 클릭/노출 (2주차)
- 한국 클릭 15 / 노출 1,409 (평균 순위 20.6) — 검색 클릭 중심
- 일본 클릭 2 / 노출 86 (50.4)
- 미국 클릭 0 / 노출 213 (39.0) — 노출 누적 중, 클릭 미발생
- 태국 109, 싱가포르 105, 홍콩 77, 중국 49 등 영어/중국어권 노출 누적 중 (평균 순위 32~85위로 낮아 클릭 미전환)
- **GA4 vs GSC 괴리**: 미국 GA4 85명인데 GSC 클릭 0 — 미국 트래픽은 검색 외 경로(Direct, 외부 링크 등)로 유입

#### GSC 페이지 인덱싱 분포 (다국어)
- 노출 누적: ko 1,409 / en 213+ / ja 86 / zh 49
- 클릭 발생 페이지: ko/clinics 7건, zh/clinics 1건 (zh 첫 클릭 ✅), en/clinics·en/treatments·ja/clinics·ja/wiki 각 1~2건
- 4/25 시점 거의 0이던 ja/zh 인덱싱 의미 있게 진척

#### GSC 일별 추세 (이상 신호)
- 4/25: 클릭 4 / 노출 467
- 4/26: 클릭 5 / 노출 814
- 4/27: 클릭 5 / 노출 788
- 4/28: 클릭 3 / 노출 223
- 4/29: 클릭 0 / 노출 37
- 4/30: 클릭 0 / 노출 19
- 5/1: 클릭 0 / 노출 21
- ⚠️ **4/29부터 노출 -97% 급감** (814 → 19~37): zh 렌더링 버그(MISSING_MESSAGE)로 zh 페이지 5xx 빈발 → Google 크롤 빈도 자동 감소 가능성 매우 높음. 5/2 zh.json 수정(SEO-011)으로 근본 원인 제거됐으므로 5/3~5/5 회복 추이 관찰 필요

#### MON-002 대비 핵심 변화
- ✅ Organic Search +84% (목표 30% 초과 달성)
- ✅ GEO 다양화 (4개국 → 22개국)
- ✅ 미국 트래픽 28배 증가 (해외 침투 시작)
- ⚠️ Reddit Referral 110 → 3 (단기 마케팅 효과 종료, 마케팅1팀과 재포스팅 주기 협의 필요)
- ⚠️ GSC 노출 4/29~ 급감 (zh 버그 인과 의심, 5/2 수정 후 회복 관찰)
- ⚠️ Direct 165 정체 불명 (봇 혼입 가능성)

#### 후속 액션
- **MON-004 예약 (5/8~5/9)**: zh.json 수정 + defaultLocale=en 변경 효과 누적 측정
- **노출 급감 모니터링 (5/3~5/5)**: GSC 일별 노출 회복 안 되면 별도 조사
- **Direct 165 출처 점검**: GA4 Direct 필터링 → 페이지·시간대로 봇 vs 사용자 구분
- **마케팅1팀 협업**: Reddit 재포스팅 주기(예: 격주) 협의

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
