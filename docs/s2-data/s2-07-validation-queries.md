# S2-07: 검증 쿼리

**최종 갱신:** 2026-04-11

## 쿼리 1: 전체 커버리지

    SELECT 'clinic_treatments' AS tbl, COUNT(*) AS records,
           COUNT(DISTINCT clinic_id) AS clinics,
           ROUND(COUNT(DISTINCT clinic_id)::numeric/2727*100,1) AS pct
    FROM clinic_treatments
    UNION ALL
    SELECT 'clinic_devices', COUNT(*), COUNT(DISTINCT clinic_id),
           ROUND(COUNT(DISTINCT clinic_id)::numeric/2727*100,1)
    FROM clinic_devices
    UNION ALL
    SELECT 'clinic_concerns', COUNT(*), COUNT(DISTINCT clinic_id),
           ROUND(COUNT(DISTINCT clinic_id)::numeric/2727*100,1)
    FROM clinic_concerns
    ORDER BY 1;

## 쿼리 2: 테이블별 레코드 수

    SELECT 'standard_treatments' AS tbl, COUNT(*) FROM standard_treatments
    UNION ALL SELECT 'treatments', COUNT(*) FROM treatments
    UNION ALL SELECT 'treatment_concerns', COUNT(*) FROM treatment_concerns
    UNION ALL SELECT 'concerns', COUNT(*) FROM concerns;

## 쿼리 3: clinic_id=6 상세

    SELECT 'clinic_treatments' AS tbl, COUNT(*)
    FROM clinic_treatments WHERE clinic_id=6
    UNION ALL
    SELECT 'clinic_concerns', COUNT(*)
    FROM clinic_concerns WHERE clinic_id=6
    UNION ALL
    SELECT 'clinic_specialties', COUNT(*)
    FROM clinic_specialties WHERE clinic_id=6;

## 쿼리 4: 영문 입력 현황

    SELECT 'clinics.name_en' AS field, COUNT(*) AS total, COUNT(name_en) AS filled
    FROM clinics
    UNION ALL
    SELECT 'treatments.name_en', COUNT(*), COUNT(name_en)
    FROM treatments
    UNION ALL
    SELECT 'encyclopedia.summary_en', COUNT(*), COUNT(summary_en)
    FROM encyclopedia;
