# S3-10: WO-022 — 백과사전 확장 + 영문주소 + 광고 우선순위

**완료일:** 2026-04-12

## 백과사전 11건 추가 (id 154~164)
- 그룹A 지방파괴주사: 눈꺼풀, 얼굴, 바디, 눈지방제거 (4건)
- 그룹B: 셀룰라이트주사 (1건)
- 그룹C 지방이식제거: 눈, 이마, 얼굴 (3건)
- 그룹C 유착제거: 바디, 이중턱, 얼굴 (3건)
- 전체 한글/영문 콘텐츠 작성, encyclopedia_treatment_map 매핑 완료

## 영문주소 일괄 변환
- juso.go.kr 영문주소 검색 API 활용
- 2,725/2,727건 변환 완료 (99.9%), 2건은 원본 주소 없음
- 15건 API 실패 → 수동 입력으로 보정

## 영문 페이지 버그 수정
- district_en: wiki, treatments, concerns, ClinicListWithFilter 전체 수정
- address_en: 전 페이지 select 쿼리 및 렌더링 수정

## 광고 우선순위 시스템
- clinics.ad_priority (전역 광고 1~3순위)
- clinic_treatments.priority (시술별 광고 4~6순위)
- ClinicListWithFilter에서 priority 기반 정렬 구현
- 참의원 지방파괴주사(바디) priority=1 테스트 완료

## 기타
- 튼살주사(id=65) 내용 수정, 더마샤인 장비 매핑 삭제
- wiki 관련 클리닉 조회에 standard_treatment_id 경로 추가
- 닥터홈즈 지방파괴주사(바디) 매핑 삭제

## 커밋 이력
- 91a860c, 여러 fix 커밋, feat: ad_priority/priority 커밋

## 2026-04-14 추가 수정
- FAQ Schema(FAQPage JSON-LD) 빈 text 필드 방지 로직 추가 (커밋 6ff2e28)
- Google Search Console "text 입력란 누락" 경고 해결
- Google 리치 결과 테스트 통과 확인
