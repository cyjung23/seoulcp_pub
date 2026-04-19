# S6-07: 마일스톤 타임라인

**최종 갱신:** 2026-04-19

| 날짜 | 마일스톤 |
|------|---------|
| 2026-04-10 | WO-014 전량 완료, WO-015/016/017 완료, Naver 20건 제출 |
| 2026-04-11 | WO-021 완료, 코드 3건, 인덱싱 31건, 표준보고서 GitHub 전환 |
| 2026-04-12 | WO-022 완료: 백과사전 11건, 영문주소 2,725건, 광고 우선순위 |
| 2026-04-14 | DOC-001 완료, Claude Code 설치, 팀 구성 확정 (DEC-050) |
| 2026-04-15 | WO-023 구글맵 연동 완료 (좌표 2,725건) |
| 2026-04-16 | WO-024 병원 파트너 회원제 완료 (v2.7+v2.8), 뷰티회원제 계획 확정 (DEC-052~054) |
| 2026-04-19 | SEO-003 Bing Webmaster Tools 등록, Baidu 보류 (v2.18), MON-001 Naver 인덱싱 1차 확인 |
| 2026-04-18 | WO-029 중국어(zh) 사이트 확장 완료 (v2.15), Phase 1~6 전량, 414항목+1,113필드 번역, 4언어 지원 체계 구축 |
| 2026-04-17 | WO-025 뷰티회원제 완료 (v2.9), WO-026 메인 디자인 개선 (v2.10), WO-027 slug 전환 완료 (v2.11), WO-028 일본어 Phase 1,2,4 완료 (v2.12) |
| 2026-04-20 | MON-001 Naver 인덱싱 1차 확인 |
| 2026-04-24 | MON-002 GEO 효과 확인 |
| 2026-04월 말 | MON 결과 종합 → 다음 작업 방향 결정 |
| 2026-05월 | WO-028-B Partner 일본어, WO-018(재매칭) 또는 WO-019(Naver 해결) |




### 2026-04-19 (v2.18) — Bing Webmaster Tools 등록 + Baidu 보류 + MON-001 1차 확인
- Bing Webmaster Tools: GSC import로 cclinic.kr + seoulcp.com 동시 등록
- sitemap.xml 제출 완료 (seoulcp.com 4언어 12,600+ URL, cclinic.kr 영문 포함)
- 일본 검색엔진 전체 커버 달성 (Google 82% + Yahoo! Japan 9% + Bing 9%)
- Baidu 站长平台: 해외 이용자 등록 일시 중단, 보류
- GFW 접속 테스트 통과 (중국 5개 도시: Beijing, Shenzhen, Inner Mongolia, Heilongjiang, Yunnan)
- **MON-001 Naver 인덱싱 1차 확인 (4/19)**
  - sitemap 제출: 2026-04-18 22:42:55, /ja/ /zh/ 웹페이지 수집 수동 요청 완료 (4/18)
  - site:seoulcp.com 전체: 6,061개
  - site:seoulcp.com/ko/: 약 150~200개 (대부분 기존 크롤링분, 실제 /ko/ 인덱싱은 6,061개 중 대부분으로 추정)
  - site:seoulcp.com/en/: 12개
  - site:seoulcp.com/ja/: 0개 (정상 — 제출 후 1일, 수집 대기중)
  - site:seoulcp.com/zh/: 0개 (정상 — 제출 후 1일, 수집 대기중)
  - 판정: sitemap 제출 후 1일 경과, /ja/ /zh/ 인덱싱은 1~2주 소요 예상
  - 2차 확인 예정: 4/24

### 2026-04-18 (v2.17) — HTML hreflang + 검색/클리닉 최종 수정 + 인덱싱 요청
- 9개 페이지에 HTML <head> hreflang alternates 추가 (ko/en/ja/zh/x-default)
- Google Search Console 라이브 URL 테스트로 6개 태그 인식 확인
- encyclopedia 검색 추가 + 컬럼명 수정 + 카드 UI 통일
- 클리닉 영업시간 요일 다국어 변환 (translateHours)
- 메인페이지 고민카드 스타일 복원
- GSC/Naver sitemap 재제출 + /ko/ /ja/ /zh/ 인덱싱 요청 완료

### 2026-04-18 (추가) — WO-029-QA 중국어 품질개선 완료
- 메인페이지 고민카드 스타일 복원 (isZh → CATEGORY_STYLES_EN)
- 검색에 encyclopedia 테이블 추가 + 컬럼명 수정
- 클리닉 상세: 기본정보 헤딩 버그 수정, 주소/specialties isZh, 영업시간 다국어 변환
- 검색 결과 UI 통일 (SectionHeader isZh, 백과사전 카드)

### 2026-04-18 — v2.15 중국어(zh) 사이트 확장
- WO-029 Phase 1~6 전량 완료
- i18n 인프라(routing, middleware, 4언어 스위처), DB 12개 컬럼 추가
- 18개 페이지 isZh 분기, 11개 페이지 텍스트 분기, 하위 컴포넌트 3개 zh 적용
- 번역: standard_treatments 120, concerns 135, encyclopedia 159(기본) + 159×7(상세) 전량 완료
- 기획2팀 협업: device 47 + surgery 38 + treatment 74 (3파트 분할 납품)
- SEO: sitemap /zh/ hreflang, zh_CN locale 메타
- 라이브 검증 완료: /zh/wiki, /zh/concerns, /zh/treatments, /zh/wiki/botox 등

### 2026-04-17 — v2.13 일본어 사이트 확장
- WO-028 Phase 1~6 완료
- 소비자 페이지 14개 일본어 적용, sitemap hreflang /ja/ 추가
- /ja 메인, /ja/treatments/botox 등 정상 동작 확인
- encyclopedia 일본어 컬럼 10개 추가 (본문 번역 진행중)
- WO-028-B (partner 페이지) 보류 분리

### 2026-04-17 — v2.14 일본어 품질 개선 + 언어 스위처 + concerns slug
- 3언어 스위처 (EN/KO/JA) 구현
- 일본어 품질 개선 6건 해결
- concerns.name_ja 135개 번역
- concerns.slug 135개 영문 URL 전환
- DB: concerns 테이블에 name_ja, slug 컬럼 추가
