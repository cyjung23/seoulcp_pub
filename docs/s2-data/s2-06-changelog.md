# S2-06: 변경 이력

**최종 갱신:** 2026-04-15

## v2.6 (2026-04-15) — 데이터 보정
- 닥터홈즈의원 지방파괴주사(얼굴) 매핑 삭제 (id=33578)
- 튼살주사 → 참의원 전용 시술로 변경: 47개 클리닉을 튼살레이저로 이동
- 클리닉 목록 priority 정렬 적용 (clinic_treatments.priority 전달)
- 참의원 전체 시술 priority=1 설정

## v2.5 (2026-04-15) — 구글맵 연동
- clinics 테이블에 latitude, longitude 컬럼 추가
- Kakao 로컬 API로 2,725/2,727건 좌표 변환 완료 (99.9%)
- 클리닉 상세 페이지: "지도에서 보기", "길찾기" 구글맵 버튼 추가
- 클리닉 목록 카드: "지도", "길찾기" 버튼 추가 (treatments, devices 페이지)
- 좌표 없는 2건(ID 4, 12)은 주소 NULL로 버튼 미표시

## v2.4 (2026-04-15) — 인프라 변경
- 표준문서 저장소: pubrepo/seoulcp-docs/ → seoulcp_pub/docs/ 이관 완료
- GitHub PAT 토큰: All repositories 권한으로 재발급
- FAQ Schema 빈 text 필드 수정 배포 확인 (커밋 6ff2e28)
- 팀 구성 확정: 기획1/2팀, 마케팅1/2팀 (DEC-050)
- 구글맵 연동 우선 착수 결정 (DEC-051)

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
