# S2-06: 변경 이력

**최종 갱신:** 2026-04-17

## v2.9 (2026-04-17) — 뷰티회원제 Phase 1+2
- beauty_accounts 테이블 생성 (user_id, nickname, preferred_language, RLS)
- user_picks 테이블 생성 (target_type TEXT, target_id TEXT, Optimistic Update)
- 뷰티회원 가입 (/beauty/signup), 로그인 (/beauty/login) 페이지
- 비밀번호 재설정 (/beauty/reset-password, /beauty/update-password)
- API route: /api/beauty/register (service_role), /api/picks (GET/POST 토글)
- PickButton 공통 컴포넌트 (Optimistic Update, 클릭 즉시 UI 반영)
- 클리닉 상세, 시술 상세, 백과사전 상세 페이지에 Pick 버튼 적용
- 비로그인 시 Pick 클릭 → 로그인 페이지 안내
- 뷰티회원 테스트 계정: cyjung23@gmail.com (beauty)

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

## v2.3 (2026-04-12) — WO-022 백과사전 확장 + 광고 우선순위
- encyclopedia: 153 → 164 (+11건: 지방파괴주사 4, 셀룰라이트 1, 지방이식제거 3, 유착제거 3)
- encyclopedia_treatment_map: +11건
- clinics.address_en: 13 → 2,725 (99.9%, juso.go.kr API 일괄 변환)
- clinics.ad_priority 컬럼 추가 (전역 광고 순위, 기본값 0)
- clinic_treatments.priority 컬럼 추가 (시술별 광고 순위, 기본값 0)
- clinic_id=6 지방파괴주사(바디) priority=1 설정
- 닥터홈즈(clinic_id=1513) 지방파괴주사(바디) 매핑 삭제
- clinic_id=6 standard_treatment_id 보정 3건 (눈꺼풀, 얼굴, 바디)
- 튼살주사(id=65) 내용 수정, 더마샤인 장비 매핑 삭제

## v2.2 (2026-04-11) — VAL-001 실측 확정
- clinic_treatments: 23,302 → 23,306 (+4)
- clinic_concerns: 21,517 → 21,516 (-1)
- treatments.name_en: 85 → 113 (100%)

## v2.1 (2026-04-11) — WO-021 참의원 데이터 보강
- standard_treatments: 111 → 120 (+9)
- treatments: 104 → 113 (+9)
- treatment_concerns: 416 → 440 (+24)

## v2.0 (2026-04-10) — 초기 기준선
- clinics 2,727 / clinic_treatments 23,309 / clinic_devices 12,518
- clinic_concerns 21,515 / standard_treatments 111 / treatments 104
