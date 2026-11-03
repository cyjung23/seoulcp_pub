# CASE-004. 닥터파마니의원 (상호 변경, 구 메이린청담의원)

- **병원명**: 닥터파마니의원 / Dr.PARMANI CLINIC
- **clinic_id**: 485 (기존 ID 유지)
- **구 명칭**: 메이린청담의원 / 메이린청담 Clinic
- **요청자**: 주식회사 행복경영연구소 오창석 / marketing@geowise.co.kr (닥터파마니의원 위임 마케팅 대행)
- **요청일**: 2026-11-03
- **처리일**: 2026-11-03
- **티켓**: DATA-011 (clinic 수정) + DATA-012 (standard_treatments 확장) + DATA-013 (devices 확장)
- **버전**: v2.36
- **상태**: 처리 완료, 회신 완료

## 배경
- 2023년 메이린 네트워크 탈퇴 및 보건소 개설신고증명서상 명칭 변경 완료 (의료기관 자체는 연속성 유지, 대표원장·주소·전화 변동 없음)
- 리뉴얼 후 "항노화·피부항노화 특화 (Dr.PARMANI Revitalization Center)"로 포지셔닝 전환
- 2026-11-03 scp DB 반영 (뒤늦게 따라간 사례)

## 접근
- **clinic_id 485 유지** (DELETE+신규 INSERT 아님) — 네이버 색인·백링크 자연 승계
- 메이린 흔적 전면 제거 (website, description, 레거시 clinic_treatments)
- 검색 동의어 "메이린청담→닥터파마니" 등록 **하지 않음** (요청자 지시)

## 반영 내역
### clinics UPDATE (id 485)
name_ko, name_en, website, operating_hours, specialties, doctors, description 교체

### standard_treatments 체계 확장 (DATA-012)
- 신규 등록 62건 (영문·일문·중문 번역 포함, 음차·공식명 혼재)
- 신규 카테고리 3종 신설:
  - category_order=18 항노화 진단 (5종)
  - category_order=19 항노화 치료 (8종)
  - category_order=20 피부재생 (11종)

### clinic_treatments 재구성
- 레거시 3건 DELETE (리프팅/리쥬란/secret, standard_treatment_id NULL)
- 신규 72건 INSERT (id 39123~39194)

### devices 체계 확장 (DATA-013)
- 신규 등록 40건 (영문·카테고리·manufacturer 포함)
- 신규 카테고리 3종 신설 (standard_treatments와 동일 명칭): 항노화 진단·항노화 치료·피부재생

### clinic_devices 재구성
- 레거시 1건 DELETE (Genesis, id=164)
- 신규 56건 INSERT (제네시스 169 포함, 기존 활용 17 + 신규 39)

## 첨부자료 수신 확인
1. 의료기관 개설신고증명서 (개인정보 일부 가림)
2. 새 간판 사진
3. 온라인 정보 수정 요청 위임 확인서 (닥터파마니의원 직인)

## 메이린 네트워크 다른 지점 (영향 없음)
- clinic 1170 메이린의원 (잠실 롯데월드)
- clinic 875 메이린의원 (성수)
- clinic 1995 메이린의원 (여의도 파크원)
- 모두 scp DB 상 clinic 485와 FK/연계 없음, 독립 관리

## 특이사항 및 후속 과제
- **폐업 아님**. 의료기관 연속성 유지 (2023년 명칭 변경, scp 반영이 뒤늦게 진행된 사례)
- ja/zh 페이지 시술명 일부 영문 폴백 (전역 버그 FIX-004로 분리)
- Genesis(164)·제네시스(169) 중복 레코드 발견 → 전체 중복 조사 티켓 후보
- 신규 카테고리 "항노화 진단/치료/피부재생"은 다른 clinic 매핑 체계에도 영향 가능 (추후 활용 여지)
- clinics.equipment 레거시 컬럼은 56개 그대로 유지 (clinic_devices와 중복 저장, 추후 DATA-014 후보로 정리)

## 회신
2026-11-03 marketing@geowise.co.kr 앞 처리 완료 안내 메일 발송
