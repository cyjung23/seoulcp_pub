# S3-04: WO-007 clinic_concerns 구축 + WO-007-B 크롤링

**최종 갱신:** 2026-04-10

## WO-007 — clinic_concerns 테이블 구축
- 상태: 완료 (초기 구축)
- 27개 concern 카테고리, 1,255개 클리닉, 11,075건 매핑
- 매핑: auto_exact 9, manual_single 13, manual_multiple 3
- 제외: 다이어트, 가슴성형, 쌩얼성형

## WO-007-B — 클리닉 웹사이트 크롤링
- 기간: 2026-04-01 ~ 2026-04-02. 상태: 완료.
- 배경: 2,727개 중 1,377개(50.5%) 시술 매핑 없음, 916개 웹사이트 URL 보유

### v1 크롤링
822개 대상, 750개 성공, 9,573건 추출. CSV 재실행으로 데이터 유실.

### v2 크롤링
정규식 네비게이션 오분류(클리닉당 62개). 562/822에서 중단. raw_text 미저장.

### v3 크롤링 (2단계 분리)
- 1단계: 775개 대상, 672개 성공(86.7%), raw_texts 672개(12.1MB)
- 2단계: 131개 키워드, 590개 클리닉, 7,699건 추출(클리닉당 13.0개)
- 매칭: standard_treatments 110개 대상, 5단계 규칙, 성공률 93.5%(7,331/7,842건)
- 리프팅 415건 의도적 unmatched
- DB 삽입: 7,331건 시도, 507건 스킵, 6,824건 성공
- 결과: clinic_treatments 16,408->23,232(+6,824), 매핑 1,350->1,936(+586), 49.5%->71.0%

### 산출물
raw_texts/(672), crawl_meta_v3.csv, treatment_keywords.csv(131), crawl_treatments_v3.csv(7,699), match_results_v3.csv(7,842), extract_stats_v3.txt, match_stats_v3.txt, db_insert_report_v3.txt. 보관: backup/scripts/
