# clinic 6 — 참의원

- **등록 상태**: 운영 중
- **홈페이지**: https://seoulcp.com/ko/clinics/6
- **최초 등록**: 일자 미상
- **최근 변경**: 2026-09-04

---

## 이력

### 2026-09-04 — 신규 시술 등록 2건 (DATA-006)
- 요청자: 병원 측 직접 요청
- 작업 1 — 결절제거주사 (참의원 독점): standard_treatments 신규 uuid `7580690e-...`, treatment 893, clinic_treatment 39075, concerns 187~192 신규 6건, treatment_concerns·clinic_concerns 매핑, encyclopedia 165, encyclopedia_treatment_map 등록 (4개 언어)
- 작업 2 — 지방흡입유착교정 (160cc, 바디용): clinic_treatment id 39076
- 동시 수정(코드): concerns/[slug]/page.tsx의 treatment_id NULL 조인 버그를 standard_treatment_id 기반으로 수정 (seoulcp commit 4e05c80)
- 동시 수정(검색 개선): 띄어쓰기 무관 양방향 매칭 — Supabase RPC 5개 신설(search_concerns/treatments/devices/clinics/encyclopedia_normalized), `.from().or()`→`.rpc()` 전환 (seoulcp commit ed33355→930ed8a)
- 비고: 매뉴얼(e31a0dd) 첫 신규 시술 등록 적용 사례
- 상세: [changelog](../../s2-data/s2-06-changelog.md)
