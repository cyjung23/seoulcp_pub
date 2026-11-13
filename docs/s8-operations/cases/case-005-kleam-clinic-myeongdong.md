# CASE-005. 클림의원 명동점 (정보 정정)

- **병원명**: 클림의원 명동점 / Kleam Clinic Myeongdong
- **clinic_id**: 2840 (기존 등록)
- **요청자**: VENTRIX Co. / ventrixcorp@gmail.com (마케팅 대행)
- **요청일**: 2026-11-13
- **처리일**: 2026-11-13
- **티켓**: DATA-014
- **버전**: v2.37
- **상태**: 처리 완료, 회신 완료

## 배경
- 외국인 손님 안내 참고용으로 scp 활용 중인 VENTRIX가 clinic 2840 페이지의 결측·오류 항목 지적
- 전화·진료시간 미입력, name_en이 자동생성 혼합표기("클림 Clinic명동점")로 저장되어 있던 상태
- 클림의원 네트워크 홍대점(clinic 32, "Kleam Clinic Hongdae")은 이미 정상 표기로 등록되어 있어 네트워크 공식 영문 표기 "Kleam" 확정

## 접근
- **clinic_id 2840 유지**, 요청받은 항목만 독립 정정 (DATA-005 정책)
- name_ko 띄어쓰기 정정("클림의원명동점" → "클림의원 명동점") 함께 반영, 홍대점 표기와 일관화
- 홍대점(clinic 32)과는 FK/연계 없이 독립 관리 (CASE-003 셀린 선례와 동일 — 네트워크의원은 이름만 공유하는 경우가 많음)
- ja/zh 이름은 clinics 테이블에 컬럼 자체가 없으므로 손대지 않음 (자동 번역 레이어)

## 반영 내역
### clinics UPDATE (id 2840)
- name_ko: "클림의원명동점" → "클림의원 명동점" (띄어쓰기 정정)
- name_en: "클림 Clinic명동점" → "Kleam Clinic Myeongdong"
- phone: NULL → "02-773-5786"
- operating_hours: NULL → "월~금 09:30~20:30, 토·일 09:00~18:00"
- name_en_source: "official_request_2026-11-13"
- name_en_corrected_at: NOW()
- updated_at: NOW()
- 주소는 요청대로 변경 없음 ("서울특별시 중구 명동길 43, 4,5층 (명동1가)")

## 클림 네트워크 다른 지점 (영향 없음)
- clinic 32 클림의원 홍대점 (Kleam Clinic Hongdae, 마포구 양화로 141 롯데호텔L7홍대 3층) — 이미 정상 표기, 변경 없음

## 특이사항 및 후속 과제
- clinics 테이블 name_ja·name_zh 컬럼 부재 재확인 (CASE-004에서도 동일 확인)
- 홍대점 name_ko는 "클림의원 홍대점"(띄어쓰기 있음), 명동점은 원래 "클림의원명동점"(띄어쓰기 없음)이었음 → 네트워크 내 표기 일관성 관점에서 요청 외 1건(name_ko) 함께 정정
- ja/zh 페이지 시술명 영문 폴백은 전역 버그 FIX-004로 분리 (v2.36 등록, 미착수)

## 회신
2026-11-13 ventrixcorp@gmail.com 앞 처리 완료 안내 메일 발송
