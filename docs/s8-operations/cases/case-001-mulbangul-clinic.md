# CASE-001. 물방울성형외과 (Stilla Plastic Surgery & Anti-Aging)

**케이스 번호:** CASE-001
**상태:** 병원 회신 대기 중
**최종 갱신:** 2026-09-14

---

## 1. 기본 정보

| 항목 | 값 |
|---|---|
| 병원명 (한글) | 물방울 성형외과 |
| 병원명 (영문) | Stilla Plastic Surgery & Anti-Aging |
| 도메인 | https://mbwps.com |
| 전화 | +82-2-511-7333 |
| 문의일 | 2026-09-10 |
| 문의자 | 강가혜 팀장 (마케팅, 경력 27년) |
| 문의 채널 | 이메일 |

## 2. 크롤링 실행 정보

- 실행일: 2026-09-14
- 사용 도구: scraper_static.py (WordPress + Astra 테마)
- 대상 URL: sitemap 81개 → core slug 필터링 후 20개
- 수집 마크다운: 130,964자
- 소요시간: 46초 (크롤링) + DeepSeek 추출 약 30초

## 3. 크롤링으로 확보된 시술 목록 (25종)

### 가슴성형 (9종)

| 번호 | 한글명 | 영문명 |
|---|---|---|
| 1 | 가슴확대 | Breast Augmentation |
| 2 | 가슴 재수술 | Breast Reoperation |
| 3 | 가슴축소 | Breast Reduction |
| 4 | 가슴리프팅 | Breast Lifting |
| 5 | 부유방 제거 | Accessory Breast Removal |
| 6 | 유륜성형 | Areola Surgery |
| 7 | 유두성형 | Nipple Surgery |
| 8 | 함몰유두 교정 | Inverted Nipple Correction |
| 9 | 여유증 수술 | Gynecomastia Surgery |

### 눈성형 (6종)

| 번호 | 한글명 | 영문명 |
|---|---|---|
| 10 | 쌍꺼풀 매몰법 | Non-incisional Double Eyelid |
| 11 | 쌍꺼풀 재수술 | Double Eyelid Revision |
| 12 | 눈매교정 | Ptosis Correction |
| 13 | 눈트임 | Canthoplasty |
| 14 | 상안검 | Upper Blepharoplasty |
| 15 | 하안검 | Lower Blepharoplasty |

### 리프팅 (8종)

| 번호 | 한글명 | 영문명 | 사용 장비 |
|---|---|---|---|
| 16 | 온다 리프팅 | ONDA Lifting | 온다 |
| 17 | 소프라노 리프팅 | Soprano Lifting | 소프라노 티타늄 |
| 18 | 튠페이스 리프팅 | Tune Face Lifting | 튠페이스 |
| 19 | 울쎄라 리프팅 | Ulthera Lifting | 울쎄라 |
| 20 | 슈링크 유니버스 리프팅 | Shurink Universe Lifting | 슈링크 유니버스 |
| 21 | 실펌 엑스 | Sylfirm X | 실펌 엑스 |
| 22 | 리팟 | Reepot | 리팟 |
| 23 | 실리프팅 | Thread Lifting | - |

### 주사 (2종)

| 번호 | 한글명 | 영문명 |
|---|---|---|
| 24 | 필러 | Filler |
| 25 | 보톡스 | Botox |

## 4. 크롤링으로 확보된 장비 목록 (7종)

| 번호 | 한글명 | 영문명 | 제조사 |
|---|---|---|---|
| 1 | 온다 리프팅 | ONDA Lifting | (미확인) |
| 2 | 소프라노 티타늄 | Soprano Titanium | Alma Lasers |
| 3 | 튠페이스 | Tune Face | (미확인) |
| 4 | 울쎄라 | Ulthera | Merz |
| 5 | 슈링크 유니버스 | Shurink Universe | Classys |
| 6 | 실펌 엑스 | Sylfirm X | Viol |
| 7 | 리팟 | Reepot | (미확인) |

## 5. 병원 회신 대기 항목

이메일 회신(2026-09-14)에 포함된 4가지 확인 요청:

- [ ] **정확한 주소**: 크롤링 본문에서 "서울시 서초구 강남대로 557, 8층 (잠원동, 성한빌딩)" 확인됨 → 이 주소가 맞는지 병원 확인 필요
- [ ] **운영시간**: 평일 / 토요일 / 일요일·공휴일
- [ ] **다국어 상담 지원**: 영어·일본어·중국어 상담 가능 여부
- [ ] **가격 공개 정책**: SCP 페이지 표시 여부 (비공개 가능)

## 6. 병원 회신 수령 시 기입란

병원에서 회신 도착 시 여기에 추가:

- 회신일: (미수령)
- 회신 내용:
  - 주소:
  - 운영시간:
  - 다국어 상담:
  - 가격 공개:
- 기타 병원 요청 사항:

## 7. DB 등록 진행 상황

- [ ] 병원 회신 수령
- [ ] clinics 테이블 등록
- [ ] treatments·clinic_treatments 등록 (25종)
- [ ] devices·clinic_devices 등록 (7종)
- [ ] 시술-장비 매핑 등록
- [ ] 4개 언어 번역 확인
- [ ] 병원 페이지 URL 접속 테스트
- [ ] 병원에 등록 완료 안내 발송
- [ ] s8-04 이력 로그 상태 업데이트

**등록 정보**

- clinic_id: (미등록)
- 등록일: (미등록)
- 공개일: (미등록)
- 페이지 URL:
  - 한국어: (미등록)
  - 영어: (미등록)
  - 일본어: (미등록)
  - 중국어: (미등록)

## 8. 원본 참고 파일 (seoulcp 저장소, 비공개)

- 크롤링 결과 JSON: `/workspaces/seoulcp/backup/scraper/output/clinics_v3/mulbangul_clinic.json`
- 원본 마크다운: `/workspaces/seoulcp/backup/scraper/output/raw_v3/mulbangul_clinic.md`
- URL 목록: `/workspaces/seoulcp/backup/scraper/clinic_urls_mbwps.json`

## 9. 특이사항 및 트러블슈팅

1. Playwright libatk 라이브러리 부족 에러 → scraper_static.py로 전환
2. Yarn GPG 키 부재로 apt update 실패 → 시스템 라이브러리 설치 대신 정적 스크래퍼 사용
3. footer 셀렉터 확장에도 DeepSeek이 주소 추출 실패 → 본문 grep으로 확인 후 이메일 재확인 요청
4. DeepSeek 프롬프트 개선(성형수술 카테고리 명시)으로 시술 10종 → 25종 증가
5. 의료진 정보(doctors)는 SCP UI에 표시되지 않으므로 크롤링에서 제외

## 10. 관련 문서

- s8-01-clinic-onboarding-playbook.md — 전체 프로세스
- s8-02-crawling-guide.md — 크롤링 절차
- s8-03-data-collection-checklist.md — 수집 체크리스트
- s8-04-onboarding-log.md — 전체 이력 로그
