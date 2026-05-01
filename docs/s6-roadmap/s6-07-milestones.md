# S6-07: 마일스톤 타임라인

**최종 갱신:** 2026-05-01

| 날짜 | 마일스톤 |
|------|---------|
| 2026-05-01 | DATA-001, SEO-007, SEO-008, SEO-009, SEO-010, GSC 5xx/404/Soft 404 진단 및 대응 (v2.21) |
| 2026-04-25 | SEO-005, SEO-006, DX-001, MON-001 2차, MON-002 1차, WO-031, MKT-001 (v2.20) |
| 2026-04-24 | Reddit r/KoreaSeoulBeauty 포스팅 (마케팅1팀 협업) |
| 2026-04-21 | SEC-001 크롤링 방어, PERF-001 wiki 쿼리 병렬화 (v2.19) |
| 2026-04-20 | WO-030 파비콘 SCP 3색 적용, OG이미지 다크 디자인 (v2.19) |
| 2026-04-19 | SEO-003 Bing Webmaster Tools 등록, Baidu 보류 (v2.18), MON-001 1차 |
| 2026-04-18 | WO-029 중국어(zh) 사이트 확장 완료 (v2.15) |
| 2026-04-17 | WO-025 뷰티회원제, WO-026 메인 디자인, WO-027 slug 전환, WO-028 일본어 Phase 1,2,4 |
| 2026-04-16 | WO-024 병원 파트너 회원제 완료 |
| 2026-04-15 | WO-023 구글맵 연동 완료 |
| 2026-04-14 | DOC-001 문서 이관, 팀 구성 확정 |
| 2026-04-12 | WO-022 백과사전 확장 완료 |
| 2026-04-11 | WO-021 완료, 인덱싱 31건, 표준보고서 GitHub 전환 |
| 2026-04-10 | WO-014 전량 완료, WO-015/016/017 완료, Naver 20건 제출 |
| **예정** | |
| 2026-05-02 | MON-003 GEO 추세 비교 (2주 후 재확인) |

---

### 2026-05-01 (v2.21) — GSC 404/Soft 404 대응 + i18n 정체성 정렬 + 빈 클리닉 noindex

- **진단**: GSC "페이지 색인 생성"에서 404 10건, Soft 404 2건, 5xx 21건 발견
  - 5xx 21건: Vercel Logs 분석 결과 모두 자연 해소 확인 (URL 검사로 1375 정상 색인 확인)
  - 403: SEC-001 미들웨어가 Bytespider 등 악성봇 차단 정상 동작
- **DATA-001**: 클리닉 2225, 2226의 가비아 errdoc URL과 "404 Not Found" description NULL 처리
- **SEO-007**: CONCERN_SLUG_MAP 오타 수정 (wide-cheekbones-cheeks, commit 22bae85)
- **SEO-008**: 심술보 → jowl-sagging 슬러그 분리 (의학적 분류: 입가 처짐=marionette-lines, 심술보=jowl-sagging, commit 69c7cf9)
- **SEO-009**: defaultLocale ko → en 변경 (SCP 해외 사용자 표준 정렬, commit 5ff5212)
- **SEO-010**: 빈 클리닉 자동 noindex 처리 — 시술·장비 0개 클리닉 787개 영향 (commit 6ee6624)
- **후속 작업 등록**: WO-033 (middleware → proxy.ts 마이그레이션), WO-034 (GSC 유효성 검사), WO-035 (외부 API 빈 클리닉 보강), WO-037 (changelog 분리)

### 2026-04-25 (v2.20) — SEO 브랜드 강화 + 모니터링 + 관리자 개선 + 마케팅
- **SEO-005**: seoulclinicpick.com 301 리다이렉트 (Vercel + 가비아 DNS)
- **SEO-006**: Organization JSON-LD 추가 (commit 0f12fcf)
- **DX-001**: VS Code CSS 경고 비활성화 (commit e8c6f92)
- **MON-001 2차**: Google 4언어 인덱싱 정상 확인, Naver /ko/ 집중 전략 전환
- **MON-002 1차**: 7일간 6개국 유입 (KR 29, JP 3, PT 3, US 3, CA 1, SA 1)
  - Organic Search 31세션 (참여율 90.3%)
  - Reddit 포스팅 후 미국 1→10명 급증 (4/24→4/25)
- **WO-031**: 관리자 대시보드 pending 건수 뱃지 (commit f0f97d5)
  - 가입 승인 요청, 수정 요청 건수를 빨간색 깜빡이는 뱃지로 표시
  - 참의원 병원계정으로 테스트 완료
- **MKT-001**: 외부 마케팅 포스팅 (마케팅1팀 협업)
  - Reddit r/KoreaSeoulBeauty (영어, 4/24) — 12시간 내 미국 신규 9명 유입
  - Medium @seoulclinicpick (영어 + 일본어) — 글로벌 타겟
  - note.com seoulclinicpick (일본어, 4/25) — 일본 사용자 타겟
- **Vercel 플랜 확인**: Hobby(무료) 일일 ~1,300~2,100명까지 가능, 현재 일평균 7세션으로 충분

### 2026-04-24 — Reddit 마케팅
- r/KoreaSeoulBeauty (28,121 구독자) 서비스 소개 포스팅
- 마케팅1팀 협업, 가이드라인 준수
- 포스팅 후 12시간 내 미국 신규 사용자 9명 유입 확인

### 2026-04-21
- **SEC-001** 크롤링 방어 적용 완료
  - robots.txt: 검색엔진 3종 + GEO봇 6종만 허용, 나머지 차단
  - Rate Limiting: IP당 60 req/min, 초과 시 5분 차단
  - 악성 봇 21종 즉시 403 차단
  - 보안 헤더: noarchive, nosniff, DENY iframe
  - 기존 i18n + concerns 리다이렉트 유지

### 2026-04-21 (성능 개선)
- **PERF-001** wiki 페이지 DB 쿼리 병렬화 완료
  - treatment_device_map + treatments 병렬 조회
  - devices + encyclopedia_device_map 병렬 조회
  - clinic_treatments 경로1 + 경로2 병렬 조회
  - 순차 8~10회 → 병렬 그룹 3개로 축소
- **PERF-002** 서버 측 페이지네이션 등록 (대기)
  - 현재: 전체 클리닉 데이터를 클라이언트에 전달 후 20개씩 표시
  - 개선: API route 분리, 스크롤/버튼 시 서버에서 20개씩 조회
  - 트리거: 유입량 증가 시 적용 검토
