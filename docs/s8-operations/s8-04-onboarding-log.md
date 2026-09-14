# s8-04. 병원 입점 이력 로그

**파일 버전:** s8-04
**소속:** 기획1팀
**업데이트:** 2026-09-14

---

## 1. 개요

병원 입점 문의부터 사이트 공개까지의 처리 이력을 시간순으로 기록. 향후 유사 케이스 대응 시 참고 자료로 활용.

## 2. 기록 형식

각 케이스마다 아래 항목을 기록:

- 병원명 / 도메인
- 문의일 / 문의자
- 크롤링 결과 요약 (시술 N종, 장비 N종)
- 회신일 / 회신 요약
- 병원 회신일 / 보강 자료
- DB 등록일 / 공개일 / clinic_id
- 특이사항 및 트러블슈팅

---

## 3. 처리 이력

### CASE-001. 물방울성형외과

**상세 데이터:** [cases/case-001-mulbangul-clinic.md](cases/case-001-mulbangul-clinic.md)

- **병원명**: 물방울 성형외과 (Stilla Plastic Surgery & Anti-Aging)
- **도메인**: https://mbwps.com
- **문의일**: 2026-09-10
- **문의자**: 강가혜 팀장 (마케팅, 경력 27년)
- **문의 채널**: 이메일

**크롤링 결과 (2026-09-14 실행)**

- 사용 도구: scraper_static.py (WordPress + Astra 테마이므로 정적 크롤링 선택)
- 대상 URL: sitemap에서 81개 수집 → core slug 필터링 후 20개로 축소
- 수집 마크다운: 130,964자
- 소요시간: 46초 (크롤링) + DeepSeek 추출 약 30초

수집 데이터:
- 병원명(한/영): 확보
- 전화: +82-2-511-7333 확보
- 홈페이지: https://mbwps.com 확보
- 시술: 25종 확보 (가슴 9종, 눈 6종, 리프팅 8종, 주사 2종)
- 장비: 7종 확보 (온다·소프라노·튠페이스·울쎄라·슈링크·실펌·리팟)
- 시술-장비 매핑: 정확히 연결됨
- **주소**: JSON에서 null (본문에는 "서울시 서초구 강남대로 557, 8층 (잠원동, 성한빌딩)"이 존재)
- 지역구: null
- 다국어 상담 지원: 빈 배열
- 운영시간: 미추출

**회신 (2026-09-14)**

- 발신: SCP 기획팀 / seoulclinicpick@gmail.com
- 형식: 8개 섹션 (플랫폼 소개 / 입점 절차 / 비용 / 수집 결과 / 추가 확인 4가지 / 등록 범위 / 수정 프로세스 / 제출 서류)
- 추가 확인 요청 4가지:
  1. 주소 확인 ("서초구 강남대로 557, 8층" 인용)
  2. 운영시간
  3. 다국어 상담 지원 여부
  4. 가격 공개 정책

**병원 회신 및 등록**

- 병원 회신일: (대기 중)
- DB 등록일: (대기 중)
- 공개일: (대기 중)
- clinic_id: (대기 중)

**특이사항 및 트러블슈팅**

1. Playwright 실행 시 libatk-1.0.so.0 라이브러리 부족 에러 발생 → scraper_static.py로 전환하여 해결
2. Yarn GPG 키(62D54FD4003F6525) 부재로 apt update 실패 → 시스템 라이브러리 설치 대신 정적 스크래퍼 사용
3. footer 셀렉터 확장(footer, .site-footer, .ast-small-footer, .ast-footer-overlay)에도 DeepSeek이 주소를 추출하지 못함 → 원본 마크다운에서 grep으로 주소 확인 후 이메일로 재확인 요청
4. DeepSeek 프롬프트 개선: 성형수술 카테고리(가슴성형·눈성형·여유증 등) 명시 추가 → 시술 10종에서 25종으로 증가
5. 의료진 정보(doctors)는 SCP UI에 표시되지 않으므로 크롤링에서 제외

**참고 파일 (seoulcp 저장소, 비공개)**

- 크롤링 결과: /workspaces/seoulcp/backup/scraper/output/clinics_v3/mulbangul_clinic.json
- 원본 마크다운: /workspaces/seoulcp/backup/scraper/output/raw_v3/mulbangul_clinic.md
- URL 목록: /workspaces/seoulcp/backup/scraper/clinic_urls_mbwps.json

---

## 4. 통계

- 총 처리 케이스: 1건
- 등록 완료: 0건
- 진행 중: 1건 (CASE-001)
- 보류: 0건

## 5. 관련 문서

- s8-01-clinic-onboarding-playbook.md — 전체 프로세스
- s8-02-crawling-guide.md — 크롤링 절차
- s8-03-data-collection-checklist.md — 수집 체크리스트
