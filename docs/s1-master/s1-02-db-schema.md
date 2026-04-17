# S1-02: DB 테이블 스키마

**최종 갱신:** 2026-04-16

## 주요 테이블

### clinics
id, name_ko, name_en, address_ko, address_en, district_ko, district_en, phone, website, description, operating_hours, foreign_languages (ARRAY), source_file, doctors (JSONB), equipment (ARRAY), specialties (ARRAY), latitude, longitude, ad_priority, crawled_at, created_at, updated_at

### standard_treatments
id (UUID), name_ko, name_en, category_ko, category_en, category_order (NOT NULL), sort_order (NOT NULL), match_keywords, created_at, type

### treatments
id (auto-increment), name_ko, name_en, category_ko, category_en, description_ko, price_krw, standard_treatment_id (UUID FK)

### concerns
id, name_ko, name_en, concern_group_ko, concern_group_en

### clinic_treatments
id, clinic_id (FK), treatment_id, treatment_name, category, standard_treatment_id (UUID FK), price_krw, price_note, description, priority, created_at

### clinic_devices
id, clinic_id, device_id, source, created_at

### clinic_concerns
id, clinic_id, concern_id (FK), source, created_at

### clinic_specialties
id, clinic_id, specialty_ko, specialty_en, created_at

### treatment_concerns
treatment_id (FK), concern_id (FK)

### treatment_devices
treatment_id, device_id

### encyclopedia
id, slug, title_ko, title_en, summary_ko, summary_en, overview_ko, overview_en, mechanism_ko, mechanism_en, effects_ko, effects_en, duration_ko, duration_en, recovery_ko, recovery_en, side_effects_ko, side_effects_en, price_range_ko, price_range_en, target_audience_ko, target_audience_en, created_at

### clinic_accounts (신규 v2.7)
id (UUID), user_id (FK → auth.users), clinic_id (FK → clinics, nullable), role (clinic_staff | admin), business_reg_number, contact_name, contact_phone, status (pending | approved | rejected | suspended), created_at, updated_at
- RLS 정책: 자기 계정 조회, 가입 요청, 관리자 조회/수정/삭제

### clinic_submissions (신규 v2.7)
id (UUID), clinic_id (FK), submitted_by (FK → auth.users), type (clinic_info | treatment_info), data (JSONB), changes (JSONB), status (pending | approved | rejected), reviewed_by, reviewed_at, submitted_at
- RLS 정책: 자기 수정요청 조회, 수정요청 등록, 관리자 조회/수정

## 매핑 경로 (clinic → treatment 3가지)

1. 경로1: clinic_treatments.treatment_id → treatments 테이블 조인
2. 경로2: clinic_treatments.standard_treatment_id → standard_treatments 직접 조회
3. 경로3: clinic_treatments.treatment_name 텍스트 폴백 (두 ID 모두 NULL인 경우)

### beauty_accounts (신규 v2.9)
id (UUID), user_id (FK → auth.users, UNIQUE), nickname, preferred_language (ko/en/ja), created_at, updated_at
- RLS 정책: 자기 계정 조회/수정, 가입 등록, 관리자 조회/수정/삭제

### user_picks (신규 v2.9)
id (UUID), user_id (FK → auth.users), target_type (TEXT: clinic/treatment/encyclopedia), target_id (TEXT), created_at
- UNIQUE (user_id, target_type, target_id)
- RLS 정책: 자기 Pick 조회/등록/삭제, 관리자 조회
