# S6-07: 마일스톤 타임라인

**최종 갱신:** 2026-04-18

| 날짜 | 마일스톤 |
|------|---------|
| 2026-04-10 | WO-014 전량 완료, WO-015/016/017 완료, Naver 20건 제출 |
| 2026-04-11 | WO-021 완료, 코드 3건, 인덱싱 31건, 표준보고서 GitHub 전환 |
| 2026-04-12 | WO-022 완료: 백과사전 11건, 영문주소 2,725건, 광고 우선순위 |
| 2026-04-14 | DOC-001 완료, Claude Code 설치, 팀 구성 확정 (DEC-050) |
| 2026-04-15 | WO-023 구글맵 연동 완료 (좌표 2,725건) |
| 2026-04-16 | WO-024 병원 파트너 회원제 완료 (v2.7+v2.8), 뷰티회원제 계획 확정 (DEC-052~054) |
| 2026-04-18 | WO-029 중국어(zh) 사이트 확장 완료 (v2.15), Phase 1~6 전량, 414항목+1,113필드 번역, 4언어 지원 체계 구축 |
| 2026-04-17 | WO-025 뷰티회원제 완료 (v2.9), WO-026 메인 디자인 개선 (v2.10), WO-027 slug 전환 완료 (v2.11), WO-028 일본어 Phase 1,2,4 완료 (v2.12) |
| 2026-04-20 | MON-001 Naver 인덱싱 1차 확인 |
| 2026-04-24 | MON-002 GEO 효과 확인 |
| 2026-04월 말 | MON 결과 종합 → 다음 작업 방향 결정 |
| 2026-05월 | WO-028-B Partner 일본어, WO-018(재매칭) 또는 WO-019(Naver 해결) |


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
