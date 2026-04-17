# S4-07: 뷰티회원제 구현 계획

**최종 갱신:** 2026-04-17
**상태:** Phase 1+2+3 완료
**관련 의사결정:** DEC-052 (명칭), DEC-053 (Pick 명칭), DEC-054 (단계별 계획)

## 용어 정의

| 용어 | 설명 | role 값 |
|---|---|---|
| 병원회원 (Partner) | 병원 관계자 (원장, 스태프) | clinic_staff, admin |
| 뷰티회원 (Beauty Member) | 서비스 이용자 (외국인 관광객 등) | beauty |
| Pick | 찜/북마크 기능 (모든 언어에서 동일 명칭) | — |

## 인증 구조

- Supabase Auth를 병원회원과 공유 (단일 auth.users 테이블)
- role로 구분: admin, clinic_staff, beauty
- 뷰티회원 전용 테이블 beauty_accounts로 프로필 관리
- 병원회원 테이블 clinic_accounts는 기존 유지

## Phase 1 — 인프라 및 가입/로그인 ✅ 완료 (4/17)

### DB
- beauty_accounts 테이블: id (UUID PK), user_id (FK, UNIQUE), nickname, preferred_language, created_at, updated_at
- RLS: 자기 계정 조회/수정, 가입 등록, 관리자 조회/수정/삭제

### 페이지
- /beauty/signup — 회원가입 (한/영, 닉네임, 약관 동의)
- /beauty/login — 로그인 (병원회원이면 partner dashboard로 리다이렉트)
- /beauty/reset-password — 비밀번호 재설정
- /beauty/update-password — 새 비밀번호 설정

### API
- /api/beauty/register — 가입 처리 (service_role, 중복 확인)

## Phase 2 — Pick 기능 ✅ 완료 (4/17)

### DB
- user_picks 테이블: id (UUID PK), user_id (FK), target_type (TEXT), target_id (TEXT), UNIQUE (user_id, target_type, target_id)
- RLS: 자기 Pick 조회/등록/삭제, 관리자 조회

### API
- /api/picks — GET (Pick 여부 확인, 목록 조회) + POST (Pick/Unpick 토글)
- /api/picks/name — GET (Pick 항목의 이름/slug 조회)

### 컴포넌트
- PickButton (src/components/PickButton.tsx): Optimistic Update, 비로그인 시 로그인 안내, sm/md 크기

### 적용 페이지
- 클리닉 상세 (/clinics/[id]) — 병원명 옆
- 시술 상세 (/treatments/[slug]) — 시술명 아래
- 백과사전 상세 (/wiki/[slug]) — 제목 아래

## Phase 3 — 마이페이지 + 네비게이션 ✅ 완료 (4/17)

### 페이지
- /beauty/mypage — 뷰티회원 마이페이지
  - My Picks 탭: Pick한 병원/시술/백과사전 목록 (필터 전환, Remove 버튼)
  - 프로필 탭: 닉네임 변경, 선호 언어 변경, 비밀번호 변경, 회원 탈퇴

### API
- /api/beauty/account — GET (조회), PATCH (수정), DELETE (탈퇴: picks+account+auth 삭제)

### 네비게이션 (AuthNav)
- AuthNav 컴포넌트 (src/components/AuthNav.tsx)
- 비로그인: Log In / Sign Up 버튼
- 로그인(뷰티회원): 닉네임 표시 → 클릭 시 마이페이지, Log Out 버튼
- 로그인(병원회원): 닉네임 표시 → 클릭 시 파트너 대시보드, Log Out 버튼
- 데스크톱/모바일 네비게이션 모두 적용
- 로그인 후 window.location.href로 페이지 새로고침하여 AuthNav 즉시 갱신

## Phase 4 — 부가 기능 (추후 검토)

- 소셜 로그인 (Google, Kakao)
- 병원 문의하기 (연락처 연결 또는 문의 폼)
- 시술 비교하기 (Pick한 시술 간 비교)
- 최근 본 병원/시술 이력
- 알림 (Pick한 병원의 가격 변경 등)
