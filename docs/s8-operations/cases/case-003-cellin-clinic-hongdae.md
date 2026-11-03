# CASE-003. 셀린의원 홍대점 (Cellin Clinic Hongdae)

- **병원명**: 셀린의원 홍대점 / Cellin Clinic Hongdae
- **clinic_id**: 2899 (신규 등록)
- **요청자**: 셀린의원 홍대점 담당자 (이메일 등록 요청)
- **요청일**: 2026-10-30
- **처리일**: 2026-10-30
- **티켓**: DATA-010 / v2.35
- **상태**: 처리 완료

## 등록 정보
- **주소**: 서울특별시 마포구 양화로 162, 좋은사람들빌딩 15층 (동교동)
- **전화**: 1800-6156
- **홈페이지**: https://hongdae.cellinclinic.com
- **진료시간**: 평일 10:00~20:00, 토·일·공휴일 10:00~17:00
- **상담 언어**: 한국어, 영어, 일본어, 중국어
- **좌표**: 37.556431, 126.9238845 (modoodoc hospital/86419 JSON-LD 기준)
- **의료진**: Lee Song (이송) 원장 (DB 저장만, UI 미노출 정책)

## 등록 처리 (3단계)

### standard_treatments 신규 (2건)
- 볼뉴머 (Volnewmer) / 리프팅 시술 / 041bec93-7311-4883-afb6-71cf3e569b9e
- 색소치료 (Pigmentation Treatment) / 피부질환 / 174317f1-0256-48d5-b2ec-cd3c0ac02a95
- 이후 UPDATE로 ja/zh 번역 입력 (볼뉴머 zh는 간체 "沃纽美"로 통일)

### clinics INSERT
- clinic_id 2899 신규 할당, source_file 'manual_entry_2026-10-30_case003'

### clinic_treatments INSERT (18건, id 39105~39122)
보톡스, 필러, 실리프팅, 지방분해주사(얼굴), 써마지, 울쎄라, 티타늄, 온다, 슈링크, 볼뉴머, 색소치료, 홍조치료, 스킨부스터, 여드름치료, 모공축소, 흉터치료, 바디조각술, 제모

## 본점 vs 지점 처리 원칙 (CASE-003 확정)
- 셀린의원 본점(id 1)과 홍대점(id 2899)은 **별개 clinic**으로 간주
- 네트워크 병원들은 이름만 공유, scp DB에서는 각 clinic 독립 관리
- clinic_treatments 매핑 시 본점/타 지점과 연결시킬 필요 없음, standard_treatment_id만 재사용

## 특이사항
- 명동점(clinic 2889) doctors 필드 데이터 오염("불만 대표원장" 등) 발견 → 별도 티켓 처리 보류
- ja/zh 페이지 시술명 영문 폴백 전역 버그 발견 (셀린 본점 교차검증) → FIX-004 신규 등록
- 좌표 소스: goodoc.co.kr 미등록, modoodoc.com 등록 활용

## 크롤링 자료
- 공식 홈페이지 (ko/en/zh-CN/zh-TW/ja/th/id/ar 8개 언어 지원)
- modoodoc.com/hospital/86419
