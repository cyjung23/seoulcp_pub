# S3-08: WO-021 참의원(clinic_id=6) 데이터 보강

**최종 갱신:** 2026-04-11

## 개요
- 작업일: 2026-04-11. 상태: 완료.
- 커밋: 5288fd8, 5465c88
- 목적: 참의원 페이지 시술 목록이 실제 시술과 불일치하는 문제 해결 및 콘텐츠 보강

## DB 변경: standard_treatments +9건
지방파괴주사 카테고리(category_order=6):
- 눈지방제거주사 (sort_order=4)
- 셀룰라이트주사 (sort_order=5)

부작용/교정 카테고리(category_order=9):
- 소세지눈주사 (sort_order=4)
- 눈지방이식제거주사 (sort_order=5)
- 이마지방이식제거주사 (sort_order=6)
- 얼굴지방이식제거주사 (sort_order=7)
- 바디 유착제거주사 (sort_order=8)
- 이중턱 유착제거주사 (sort_order=9)
- 얼굴 유착제거주사 (sort_order=10)

## DB 변경: treatments +9건
위 9개 시술을 treatments 테이블에도 등록. 전량 name_en 포함.

## DB 변경: treatment_concerns +24건
- 눈지방제거주사 -> 눈두덩이살(154), 지방과다(32)
- 셀룰라이트주사 -> 셀룰라이트(22), 울퉁불퉁한 피부(176)
- 소세지눈주사 -> 소세지눈(159), 눈두덩이살(154)
- 눈지방이식제거주사 -> 지방 과생착(147), 울퉁불퉁함(149), 눈두덩이살(154)
- 이마지방이식제거주사 -> 지방 과생착(147), 울퉁불퉁함(149), 딱딱한 덩어리(151)
- 얼굴지방이식제거주사 -> 지방 과생착(147), 울퉁불퉁함(149), 딱딱한 덩어리(151)
- 바디 유착제거주사 -> 유착(148), 수술 후 유착(152), 바이오본드 유착(153)
- 이중턱 유착제거주사 -> 유착(148), 이중턱(28), 수술 후 유착(152)
- 얼굴 유착제거주사 -> 유착(148), 수술 후 유착(152), 부작용(111)

## DB 변경: clinic_treatments 정비 (최종 12건)
- 삭제: 레거시 중복 2건(id 444, 447), 고민 이동 2건(id 24833, 24834), 미사용 6건
- 교체: 지방흡입->지방흡입유착교정(c538707b), 지방이식->지방이식제거(1c288b73)
- 카테고리 수정: id 445 튼살주사->흉터, id 446/448/449 ->지방파괴주사
- 신규 9건 추가

최종 12건: 지방파괴주사(눈꺼풀/얼굴/바디), 눈지방제거주사, 셀룰라이트주사, 눈지방이식제거주사, 이마지방이식제거주사, 얼굴지방이식제거주사, 얼굴 유착제거주사, 이중턱 유착제거주사, 바디 유착제거주사, 튼살주사.

## DB 변경: clinic_concerns +2건
셀룰라이트(concern_id=22), 튼살(concern_id=178) 이동. 최종 10건.

## DB 변경: clinic_specialties 재정렬
기존 5행(149~153) 삭제, 신규 5행(2019~2023):
1. 지방이식제거주사 (Fat Graft Removal Injection)
2. 유착제거주사 (Adhesion Removal Injection)
3. 눈지방제거주사 (Eye Fat Removal Injection)
4. 지방파괴주사 (Adipocytolysis Injection)
5. 지방이식/지방흡입 부작용치료 (Fat Graft/Liposuction Complication Treatment)

## DB 변경: clinics description
"참의원 | 주사시술 전문 비만클리닉 - 서울 관악구"

## 코드 변경 (3개 파일)
- clinics/[id]/page.tsx: 3경로 매핑, generateMetadata description 반영, JSON-LD MedicalBusiness
- treatments/[slug]/page.tsx: standard_treatment_id 기반 클리닉 조회, name_en
- ClinicListWithFilter.tsx: locale 기반 name_en 영문 표시

## 인덱싱 제출
- Google Search Console: 참의원 ko/en 2건 + 시술 9건 x2언어 = 20건
- Naver Search Advisor: ko URL 11건
