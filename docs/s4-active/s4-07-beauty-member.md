# S4-07: 뷰티회원제 구현 계획

**최종 갱신:** 2026-04-16
**상태:** 계획 확정, Phase 1 착수 예정 (4/17~)
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

## Phase 1 — 인프라 및 가입/로그인

### DB
- beauty_accounts 테이블 생성
  - id (UUID PK)
  - user_id (FK → auth.users, UNIQUE)
  - nickname
  - preferred_language (ko/en/ja 등)
  - created_at, updated_at
  - RLS: 자기 계정 조회/수정

### 페이지
- /beauty/signup — 회원가입 (영문 우선 UI)
- /beauty/login — 로그인
- /beauty/reset-password — 비밀번호 재설정

### API
- /api/beauty/register — 가입 처리 (service_role)

### 검토 사항
- 소셜 로그인 (Google, Kakao) 도입 여부 결정 필요
- 외국인 타겟이므로 Google 로그인 우선 검토

## Phase 2 — Pick 기능

### DB
- user_picks 테이블 생성
  - id (UUID PK)
  - user_id (FK → auth.users)
  - target_type (clinic / treatment / encyclopedia)
  - target_id (INTEGER)
  - created_at
  - UNIQUE (user_id, target_type, target_id)
  - RLS: 자기 Pick 조회/등록/삭제

### UI
- 클리닉 상세, 시술 상세, 백과사전 페이지에 Pick 버튼 추가
- 로그인 상태: Pick / Picked 토글
- 비로그인 상태: 클릭 시 로그인 페이지로 안내
- 버튼 텍스트: "Pick" / "Picked" (모든 언어 동일)

## Phase 3 — 마이페이지

### 페이지
- /beauty/mypage — 뷰티회원 마이페이지
  - My Picks 탭: Pick한 병원 목록 / Pick한 시술 목록 / Pick한 백과사전 목록
  - 프로필 수정: 닉네임, 선호 언어 변경
  - 비밀번호 변경
  - 회원 탈퇴

## Phase 4 — 부가 기능 (추후 검토)

- 병원 문의하기 (연락처 연결 또는 문의 폼)
- 시술 비교하기 (Pick한 시술 간 비교)
- 최근 본 병원/시술 이력
- 알림 (Pick한 병원의 가격 변경 등)
