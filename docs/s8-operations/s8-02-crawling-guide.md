# s8-02. 병원 홈페이지 크롤링 가이드

**파일 버전:** s8-02
**소속:** 기획1팀
**업데이트:** 2026-09-14

---

## 1. 개요

병원 입점 문의 시 홈페이지 자료를 자동 수집하는 기술 절차. 시술·장비 데이터를 확보해 DB 등록 준비 단계까지 진행합니다.

## 2. 크롤링 도구

두 가지 스크립트를 사용하며, 사이트 특성에 따라 선택합니다.

- scraper_v3.py — 정식 크롤링 (Playwright + DeepSeek), 위치: /workspaces/seoulcp/backup/scraper/
- scraper_static.py — 정적 크롤링 (requests + BeautifulSoup), 위치: /workspaces/seoulcp/backup/scraper/

선택 기준:
- WordPress 등 SSR 사이트 → scraper_static.py (빠르고 안정적)
- SPA·JS 렌더링 필수 사이트 → scraper_v3.py (Playwright 필요)

## 3. 사전 준비

### 3-1. 환경 변수

파일 위치: /workspaces/seoulcp/backup/scraper/.env

필요 항목:
- DEEPSEEK_API_KEY (DeepSeek Chat API 키)
- SUPABASE_URL (프로젝트 URL)
- SUPABASE_KEY (service role key)

권한 설정 명령: chmod 600 .env

### 3-2. 파이썬 라이브러리

설치 명령:
- pip install crawl4ai supabase python-dotenv requests beautifulsoup4 markdownify
- python3 -m playwright install chromium

주의: Playwright 브라우저 실행 시 libatk-1.0.so.0 등 시스템 라이브러리 부족 에러가 발생할 수 있습니다. Codespace 환경에서는 scraper_static.py로 우회를 권장합니다.

## 4. 크롤링 절차

### 4-1. URL 수집 (sitemap 활용)

병원 도메인의 sitemap을 순차 확인:
- /sitemap_index.xml
- /page-sitemap.xml
- /post-sitemap.xml

curl로 다운로드 후 URL 목록 추출.

### 4-2. URL 필터링

리뷰·블로그 페이지는 제외하고 시술·장비·소개 페이지만 남깁니다.

참고 core slug (물방울성형외과 케이스 기준):
premium, shurink-universe, onda-lifting, ulthera, tune, thread-lift, sylfirm-x, titanium, reepot, skin-booster, filler-botox, eye, nipple, man, polymastia, reduction, reoperation, revision, academic-activities

### 4-3. clinic_urls JSON 작성

파일명: clinic_urls_[병원명].json
형식: 배열 안에 clinic_id와 urls(문자열 배열)를 갖는 객체 하나.

예시 (물방울성형외과): clinic_id는 mulbangul_clinic, urls는 20개 내외의 core 페이지 URL.

### 4-4. 크롤러 실행

명령 순서:
1. cd /workspaces/seoulcp/backup/scraper
2. cp clinic_urls_[병원명].json clinic_urls_ko.json
3. time python3 scraper_static.py 2>&1 | tee /tmp/[병원명]_crawl.log

### 4-5. 결과물 위치

- 원본 마크다운: output/raw_v3/[clinic_id].md
- 구조화 JSON: output/clinics_v3/[clinic_id].json

## 5. 결과 검증

### 5-1. JSON 기본 필드 확인

python3로 output/clinics_v3/[clinic_id].json을 읽어 다음 필드를 출력:
- 병원명 (name_ko)
- 주소 (address)
- 전화 (phone)
- 시술 수 (treatments 배열 길이)
- 장비 수 (equipment 배열 길이)

### 5-2. 최소 기준 충족 판정

- 시술 ≥ 1개 또는 장비 ≥ 1개 → 등록 진행
- 둘 다 0 → 병원에 이메일로 자료 요청 또는 robots noindex 처리

## 6. 알려진 이슈 및 대응

### 6-1. Playwright libatk 에러

증상: TargetClosedError, libatk-1.0.so.0: cannot open shared object file
대응: scraper_static.py로 전환

### 6-2. Yarn GPG 키 에러 (apt update 시)

증상: NO_PUBKEY 62D54FD4003F6525
대응: 시스템 라이브러리 설치 대신 정적 스크래퍼 사용

### 6-3. 주소가 JSON에 null로 나오는 경우

원인: DeepSeek 프롬프트가 footer 텍스트를 인식 못 함, 또는 사이트가 주소를 이미지로만 표시
대응:
1. 원본 마크다운(output/raw_v3/*.md)에서 grep으로 주소 키워드 수동 검색
2. 발견되면 JSON에 수동 반영 또는 프롬프트에 명시적 힌트 추가
3. 발견 안 되면 이메일로 병원에 확인 요청 (s8-03 체크리스트의 이메일 문의 필수 4가지)

### 6-4. 시술/장비 카테고리 누락 (성형수술 등)

원인: DeepSeek 프롬프트가 비수술 중심으로 최적화된 경우
대응: scraper_v3.py의 system_instruction에 성형수술 카테고리(가슴성형·눈성형·여유증 등) 명시 추가

## 7. 사이트별 특이사항 기록

향후 신규 병원 처리 시 이 표에 추가.

- 물방울성형외과 / mbwps.com / WordPress + Astra 테마 / footer의 주소가 텍스트로 있으나 DeepSeek 추출 실패 → 본문에서 확보 가능

## 8. 관련 문서

- s8-01-clinic-onboarding-playbook.md — 전체 프로세스
- s8-03-data-collection-checklist.md — 수집 체크리스트
- s2-06-changelog.md — DB 관련 변경 이력
