# S3-11: WO-024 병원 파트너 회원제

**최종 갱신:** 2026-04-16

## Phase 1 (v2.7, 2026-04-16)
- Supabase Auth 설정: Email Provider, URL Configuration (seoulcp.com), Confirm email
- clinic_accounts 테이블: user_id, clinic_id (nullable), role, status, RLS 정책
- clinic_submissions 테이블: 수정요청 저장, RLS 정책
- 회원가입 (/partner/signup): 병원 검색, 약관 동의, Supabase Auth + API route 등록
- 로그인 (/partner/login): 상태 확인 (pending/rejected/suspended 처리)
- 비밀번호 재설정 (/partner/reset-password, /partner/update-password)
- 파트너 대시보드 (/partner/dashboard): 관리자/병원회원 메뉴 분리
- API route (/api/partner/register): service_role로 RLS 우회 등록
- Vercel 환경변수 SUPABASE_SERVICE_ROLE_KEY 추가
- 푸터에 병원 관계자, 이용약관, 개인정보처리방침 링크 추가

## Phase 2 (v2.8, 2026-04-16)
- 관리자 패널 (/partner/admin): 가입 승인/거절/정지
- 수정 요청 검토 (/partner/admin/submissions): 병원 정보 + 시술 정보 변경 상세 표시, 승인 시 DB 자동 반영
- 병원 정보 수정 폼 (/partner/edit-clinic): 병원명, 전화, 웹사이트, 진료시간, 소개
- 시술 정보 수정 폼 (/partner/edit-treatments): 가격, 가격 안내(price_note), 설명, 신규 시술 추가
- clinic_treatments 테이블에 price_note 컬럼 추가
- 소비자 페이지에 price_note 표시 (클리닉 상세, 시술 목록, 클리닉 카드)
- 관리자 계정: cyjung23@gmail.com → seoulclinicpick@gmail.com 변경
- 이용약관 (/terms), 개인정보처리방침 (/privacy) 페이지 (한/영)
- API routes: /api/partner/submission, /api/partner/admin/accounts, /api/partner/admin/submissions
