# clinic 2280 — 엄나구모성형외과의원 (Umnagumo)

- **등록 상태**: 운영 중
- **홈페이지**: https://seoulcp.com/ko/clinics/2280
- **최초 등록**: 일자 미상
- **최근 변경**: 2026-08-11

---

## 이력

### 2026-08-11 — 기본 정보 정정 (DATA-003)
- 요청자: 병원 측 직접 요청
- 작업: name_en 정정(엄나구모→Umnagumo), phone 지역번호 추가(02-), description에서 "전문" 삭제
- DB 영향: clinics UPDATE 3건, curl 검증 완료, 병원 회신 발송
- 비고: 이 건을 계기로 clinics.name_en_source·name_en_corrected_at 컬럼 신설 (DATA-005 보완책 1)
- 상세: commit 4775941, [v2.25 changelog](../../s2-data/s2-06-changelog.md)

### 2026-08-11 — 다국어 페이지 타이틀 적용 (후속)
- 작업: EN/JA 페이지 타이틀 인라인 override 방식 적용
- DB 영향: 없음 (코드 레벨)
- 상세: commit d6e52d3
