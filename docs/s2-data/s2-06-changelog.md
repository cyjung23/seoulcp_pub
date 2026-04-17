# S2-06: 버전별 변경 사항

**최종 갱신:** 2026-04-17


## v2.13 (2026-04-17)
### WO-028: 일본어 사이트 확장 (Phase 1~6 완료)
- **Phase 1 (i18n 인프라):** routing.ts, middleware.ts, request.ts, layout.tsx에 ja 로케일 추가, messages/ja.json 생성
- **Phase 2 (DB):** standard_treatments.name_ja 120개, concerns.concern_group_ja 13개, encyclopedia 일본어 컬럼 10개 추가
- **Phase 3 (로케일 로직):** 소비자 페이지 14개 파일 isEn→isJa/isEn/ko 3항 구조 전환, locale 판정 로직 6개 파일 수정
- **Phase 4 (번역):** 시술명 카타카나 120개, 고민 카테고리 13개, 메인 페이지 UI 텍스트, 섹션 타이틀 일본어 번역
- **Phase 5 (SEO):** sitemap.xml에 /ja/ hreflang 추가 (ko/en/ja 3개 언어 alternates)
- **Phase 6 (검증):** /ja 메인, /ja/treatments/botox, sitemap.xml 정상 확인
- **보류:** WO-028-B partner 페이지 일본어 (9개 파일 120개소, 일본인 파트너 유입 시 진행)
- **완료:** encyclopedia 본문 일본어 번역 159항목 (device 47 + surgery 38 + treatment 74) 전량 DB 반영

### 기타 수정
- treatments 목록 "Coming soon" 버그 수정 (treatments 테이블에 없는 slug 컬럼 select 제거)
- StandardTreatment 인터페이스에 name_ja 추가 (treatments/page.tsx, surgeries/page.tsx)
- wiki/page.tsx categoryConfig에 labelJa 추가, select에 title_ja/summary_ja 추가

## v2.12 (2026-04-17) — 일본어 사이트 인프라 + 데이터 (WO-028 Phase 1,2,4)

- i18n 인프라: routing.ts, middleware.ts, request.ts, layout.tsx에 ja 로케일 추가
- messages/ja.json 생성 (nav, common, home 일본어 UI 텍스트)
- standard_treatments 테이블에 name_ja 컬럼 추가, 120개 시술 카타카나 번역 입력
- concerns 테이블에 concern_group_ja 컬럼 추가, 13개 고민 카테고리 일본어 번역 입력
- 메인 페이지 UI 텍스트 일본어 번역 확보 (코드 반영은 Phase 3에서)
- 용어 선택: エイジングケア(안티에이징), 美容辞典(백과사전), 美容整形(수술), スキン(피부 뱃지)
- /ja/ 경로 접속 가능 확인 (콘텐츠는 Phase 3 이후 일본어 표시)
- treatments 목록 Coming soon 버그 수정 (treatments 테이블에 없는 slug 컬럼 select 제거)

## v2.11 (2026-04-17) — 시술 URL 영문 slug 전환 (WO-027)

- standard_treatments 테이블에 slug 컬럼 추가 (TEXT UNIQUE)
- name_en 기반 120개 시술 slug 자동 생성 (소문자, 하이픈 변환, 특수문자 제거)
- 용어 구분 준수: 지방파괴주사→adipocytolysis, 지방분해주사→fat-dissolving
- ellansé → ellanse 수동 보정
- treatments/[slug]/page.tsx: slug 우선 조회, name_ko fallback, 301 리다이렉트
- 내부 링크 교체 (10개 파일, 14개소): page.tsx, treatments/page.tsx, clinics/[id]/page.tsx, concerns/[slug]/page.tsx, surgeries/page.tsx, search/page.tsx, wiki/[slug]/page.tsx, DeviceRelatedTreatments.tsx, beauty/mypage/page.tsx, sitemap.ts
- 각 파일 interface/type에 slug 필드 추가, select 쿼리에 slug 포함
- concerns/[slug]/page.tsx: standard_treatments에서 slugMap 조회 추가
- sitemap.xml 전체 영문 slug 기반 URL 출력 확인
- GSC + Naver Search Advisor sitemap 재제출 완료
- 검증: /en/treatments/botox, /en/treatments/onda, /ko/treatments/filler, 한글 URL 리다이렉트 정상

## v2.10 (2026-04-17) — 메인 페이지 디자인 개선 (WO-026)

- Quick Links: 글래스모피즘 카드 + 상단 컬러 바 + 3칸 배열, 배경색 #f5fbf9
- Popular Treatments: Compact Strip 카드 (52px 아이콘바) + 3칸 배열, 그라데이션 배경
- 시술 9개 표시 (필러, 보톡스, 실리프팅, 울쎄라, 리쥬란, 제모, 써마지, 지방이식, 지방흡입)
- 카테고리 뱃지 (Face/Body/Skin/Anti-aging), 클리닉 수, Compare 링크
- max-w-7xl 폭 통일 (Browse by Concern과 동일), 폰트 크기 상향
- 3개 섹션 배경 구분: Browse by Concern(흰색) → Quick Links(#f5fbf9) → Popular Treatments(그라데이션)
- globals.css에 glass-card-ql, pt-card-glass, pt-strip CSS 추가

## v2.9 (2026-04-17) — 뷰티회원제 Phase 1+2+3

- beauty_accounts 테이블 생성 (user_id, nickname, preferred_language, RLS)
- user_picks 테이블 생성 (target_type TEXT, target_id TEXT, Optimistic Update)
- 뷰티회원 가입 (/beauty/signup), 로그인 (/beauty/login) 페이지
- 비밀번호 재설정 (/beauty/reset-password, /beauty/update-password)
- API route: /api/beauty/register (service_role), /api/picks (GET/POST 토글)
- PickButton 공통 컴포넌트 (Optimistic Update, 클릭 즉시 UI 반영)
- 클리닉 상세, 시술 상세, 백과사전 상세 페이지에 Pick 버튼 적용
- 비로그인 시 Pick 클릭 → 로그인 페이지 안내
- 뷰티회원 테스트 계정: cyjung23@gmail.com (beauty)
- 마이페이지 (/beauty/mypage): My Picks 탭(병원/시술/백과사전), 프로필 수정, 비밀번호 변경, 회원 탈퇴
- API routes: /api/beauty/account (GET/PATCH/DELETE), /api/picks/name (Pick 항목 이름 조회)
- AuthNav 컴포넌트: 네비게이션에 로그인 상태 표시 (비로그인: Log In/Sign Up, 로그인: 닉네임/Log Out)
- 데스크톱/모바일 네비게이션 모두 적용
- 로그인 후 페이지 새로고침으로 AuthNav 즉시 갱신

## v2.8 (2026-04-16) — 병원 파트너 회원제 Phase 2

- 관리자 패널: 가입 승인/거절/정지 기능 (/partner/admin)
- 수정 요청 검토: 병원 정보 + 시술 정보 변경 내역 상세 표시, 승인 시 DB 자동 반영 (/partner/admin/submissions)
- 병원 정보 수정 폼: 병원명, 전화, 웹사이트, 진료시간, 소개 수정 (/partner/edit-clinic)
- 시술 정보 수정 폼: 가격, 가격 안내(price_note), 설명 수정 (/partner/edit-treatments)
- clinic_treatments 테이블에 price_note 컬럼 추가
- 소비자 페이지에 price_note 표시 (클리닉 상세, 시술 목록, 클리닉 카드)
- 관리자 계정 cyjung23 → seoulclinicpick@gmail.com 으로 변경
- API routes: /api/partner/submission, /api/partner/admin/accounts, /api/partner/admin/submissions

## v2.7 (2026-04-16) — 병원 파트너 회원제 Phase 1

- Supabase Auth 설정 (Email, URL Configuration, Confirm email)
- clinic_accounts 테이블 생성 (user_id, clinic_id nullable, role, status, RLS)
- clinic_submissions 테이블 생성 (수정요청 저장용, RLS)
- 회원가입 (/partner/signup), 로그인 (/partner/login) 페이지
- 비밀번호 재설정 (/partner/reset-password, /partner/update-password)
- 파트너 대시보드 (/partner/dashboard) — 관리자/병원회원 메뉴 분리
- API route (/api/partner/register) — service_role로 RLS 우회 등록
- Vercel 환경변수 SUPABASE_SERVICE_ROLE_KEY 추가
- 푸터에 병원 관계자, 이용약관, 개인정보처리방침 링크 추가
- 계정: cyjung23@gmail.com (admin), idcharm23@gmail.com (clinic_staff, 참의원)

## v2.6 (2026-04-15) — 데이터 보정

- 닥터홈즈의원 지방파괴주사(얼굴) 매핑 삭제 (id=33578)
- 튼살주사 → 참의원 전용 시술로 변경: 47개 클리닉을 튼살레이저로 이동
- 클리닉 목록 priority 정렬 적용 (clinic_treatments.priority 전달)
- 참의원 전체 시술 priority=1 설정

## v2.5 (2026-04-15) — 구글맵 연동

- clinics 테이블에 latitude, longitude 컬럼 추가
- Kakao 로컬 API로 2,725/2,727건 좌표 변환 완료 (99.9%)
- 클리닉 상세 페이지: "지도에서 보기", "길찾기" 구글맵 버튼 추가
- 클리닉 목록 카드: "지도", "길찾기" 버튼 추가 (treatments, devices 페이지)
- 좌표 없는 2건(ID 4, 12)은 주소 NULL로 버튼 미표시

## v2.4 (2026-04-15) — 인프라 변경

- 표준문서 저장소: pubrepo/seoulcp-docs/ → seoulcp_pub/docs/ 이관 완료
- GitHub PAT 토큰: All repositories 권한으로 재발급
- FAQ Schema 빈 text 필드 수정 배포 확인 (커밋 6ff2e28)
- 팀 구성 확정: 기획1/2팀, 마케팅1/2팀 (DEC-050)
- 구글맵 연동 우선 착수 결정 (DEC-051)

## v2.3 ~ v2.0

(이전 내용 동일 — 생략)
