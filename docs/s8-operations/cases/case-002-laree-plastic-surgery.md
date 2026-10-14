# CASE-002. 라리성형외과의원 (Laree Plastic Surgery)

**케이스 번호:** CASE-002
**상태:** 처리 완료 (2차 반영)
**최종 갱신:** 2026-10-13

---

## 1. 기본 정보

| 항목 | 값 |
|---|---|
| 병원명 (한글) | 라리성형외과의원 |
| 병원명 (영문) | Laree Plastic Surgery |
| clinic_id | 1494 (기존 등록) |
| 문의자 | VENTRIX Co. / ventrixcorp@gmail.com (마케팅 대행) |

## 2. 처리 이력

| 차수 | 요청일 | 처리일 | 요청 내용 | 반영 결과 | changelog |
|---|---|---|---|---|---|
| 1차 | 2026-10-01 | 2026-10-06 | 주소 이전·연락처·영문 상호 정정 | address_ko, phone, name_en, 좌표 UPDATE | v2.32 / DATA-007 |
| 2차 | 2026-10-13 | 2026-10-13 | 진료시간·의료진·소개·수술 항목 추가 | operating_hours, doctors(4명), description(ko) UPDATE / 수술 8건 INSERT | v2.35 / DATA-009 |

## 3. 특이사항

- doctors·description은 UI 정책상 DB 저장만 하고 페이지 미노출 (참의원 clinic 6과 동일).
- description_en 컬럼 미존재로 영문 소개는 보류 (요청자 원문 별도 보관).
- 좌표 소스는 goodoc.co.kr JSON-LD(schema.org)가 가장 정확 (CASE-001 참조).
