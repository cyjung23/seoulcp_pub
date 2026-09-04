# 병원 개별 변경 요청 대장

병원측이 자발적으로 요청한 정보 변경 이력.
시스템 자동 갱신(크롤링 등)은 s2-06-changelog.md 참조.

| 처리일 | 병원ID | 요청 내용 | 처리 결과 | 참조 |
|--------|--------|-----------|-----------|------|
| 2026-08-11 | 2280 | name_en 오류 정정(엄나구모→Umnagumo), 전화번호 지역번호 추가(02-), description에서 "전문" 삭제 | 적용 완료 | commit 4775941 |
| 2026-08-11 | 2280 | 다국어 페이지 타이틀(EN/JA) 요청 | 적용 완료 (인라인 override 방식) | commit d6e52d3 |

| 2026-09-04 | 6 | 신규 시술 등록: 결절제거주사 (참의원 독점) - standard_treatments + treatments + clinic_treatments + concerns 6건 + treatment_concerns + clinic_concerns + encyclopedia + encyclopedia_treatment_map | 적용 완료 (DATA-006) | 다중 INSERT, 매뉴얼 e31a0dd 첫 적용 |
| 2026-09-04 | 6 | 신규 시술 등록: 지방흡입유착교정 (160cc, 참의원 유착제거주사 바디용) | 적용 완료 | clinic_treatments id 39076 |
