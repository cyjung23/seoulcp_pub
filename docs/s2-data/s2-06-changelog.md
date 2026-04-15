# S2-06: 변경 이력

**최종 갱신:** 2026-04-12

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
