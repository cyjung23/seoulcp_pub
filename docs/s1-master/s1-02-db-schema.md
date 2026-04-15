# S1-02: DB 테이블 스키마

**최종 갱신:** 2026-04-11

## 주요 테이블

### clinics
id, name_ko, name_en, address_ko, address_en, district_ko, district_en, phone, website, description, latitude, longitude, created_at

### standard_treatments
id (UUID), name_ko, name_en, category_ko, category_en, category_order (NOT NULL), sort_order (NOT NULL), match_keywords, created_at, type

### treatments
id (auto-increment), name_ko, name_en, category_ko, category_en, description_ko, price_krw, standard_treatment_id (UUID FK)

### concerns
id, name_ko, name_en, concern_group_ko, concern_group_en

### clinic_treatments
id, clinic_id (FK), treatment_id, treatment_name, category, standard_treatment_id (UUID FK), price_krw, description, created_at

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

## 매핑 경로 (clinic -> treatment 3가지)

1. 경로1: clinic_treatments.treatment_id -> treatments 테이블 조인
2. 경로2: clinic_treatments.standard_treatment_id -> standard_treatments 직접 조회
3. 경로3: clinic_treatments.treatment_name 텍스트 폴백 (두 ID 모두 NULL인 경우)
