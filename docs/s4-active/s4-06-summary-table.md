# 작업 현황 요약 (Summary Table)
> 최종 업데이트: 2026-05-02

| ID | 작업명 | 상태 | 비고 |
|----|--------|------|------|
| SEO-011 | zh.json 번역 누락 수정 | ✅ 완료 | MISSING_MESSAGE (home/common.siteName) 해결, ko/en/ja 표준 구조로 재작성, commit d3e449a (2026-05-02) |
| SEO-010 | 빈 클리닉 자동 noindex 처리 | ✅ 완료 | 시술·장비 0개 클리닉 787개 영향, commit 6ee6624 |
| SEO-009 | defaultLocale ko → en 변경 | ✅ 완료 | SCP 정체성 정렬 (해외 사용자 표준), commit 5ff5212 |
| SEO-008 | 심술보 → jowl-sagging 슬러그 분리 | ✅ 완료 | concerns DB UPDATE + commit 69c7cf9 |
| SEO-007 | CONCERN_SLUG_MAP 오타 수정 | ✅ 완료 | wide-cheekbones-cheeks, commit 22bae85 |
| DATA-001 | 클리닉 2225/2226 오염 데이터 정리 | ✅ 완료 | website/description NULL 처리 (가비아 errdoc URL 제거) |
| MON-003 | GEO 추세 비교 (2주차) | 📋 5/2 예정 | GA4 + GSC 데이터 비교 |
| WO-033 | middleware → proxy.ts 마이그레이션 | 📋 대기 | Next.js 16 deprecation 대응 + locale prefix 자동보정 |
| WO-034 | GSC 유효성 검사 트리거 + 모니터링 | 📋 대기 | 5xx/404/Soft 404 정리 확인, MON-003과 함께 진행 |
| WO-035 | 빈 클리닉 데이터 자동 보강 | 📋 대기 | 네이버 플레이스 + 심평원 API 활용 |
| WO-037 | changelog 분리 + 인수인계 효율화 | 📋 대기 | 활성/아카이브 분리 + 인덱스 신설 |
| SEO-006 | Organization JSON-LD (브랜드 인식 강화) | ✅ 완료 | layout.tsx에 구조화 데이터 추가, commit 0f12fcf |
| SEO-005 | seoulclinicpick.com → seoulcp.com 301 리다이렉트 | ✅ 완료 | Vercel + 가비아 DNS |
| DX-001 | VS Code CSS 경고 비활성화 | ✅ 완료 | Tailwind v4 @theme 호환, commit e8c6f92 |
| SEC-001 | 크롤링 방어 (robots.txt + Rate Limiting) | ✅ 완료 | 2026-04-21 |
| PERF-001 | wiki DB 쿼리 병렬화 (Promise.all) | ✅ 완료 | 2026-04-21 |
| MON-001 | Naver/Google 인덱싱 확인 | ✅ 2차 완료 | Google 4언어 정상, Naver /ko/ 집중 전략 전환 |
| MON-002 | GEO 효과 분석 | ✅ 1차 완료 | 6개국 유입, Reddit 포스팅 효과 확인 |
| WO-031 | 관리자 대시보드 pending 뱃지 | ✅ 완료 | 가입승인/수정요청 건수 표시, commit f0f97d5 |
| MKT-001 | 외부 마케팅 포스팅 | ⏳ 진행중 | Reddit(영어), Medium(영어+일본어), note.com(일본어) |
| MON-003 | GEO 추세 비교 (2주 후) | 📋 대기 | 5/2 예정 |
| WO-030 | 로고 디자인 + 파비콘 적용 | ⏳ 진행중 | 파비콘 완료, 로고 산출물 진행중 |
| PERF-002 | 서버 측 페이지네이션 | 📋 대기 | 유입량 증가 시 검토 |
| SEO-004 | Baidu 站长平台 등록 | 🔒 보류 | 해외 등록 일시 중단, 재개 시 진행 |
| WO-028-B | Partner 페이지 일본어 | 🔒 보류 | 9파일 120개소, 일본인 파트너 유입 시 진행 |
| WO-018 | unmatched 재매칭 | 🔒 보류 | — |
| WO-017-v2 | 아이콘 재진행 | 🔒 보류 | — |
| WO-019 | Naver body 인덱싱 | 🔒 보류 | — |
| WO-020 | mojibake 수정 | 🔒 보류 | — |
| WO-029-SEO | HTML hreflang 4언어 + 인덱싱 요청 | ✅ 완료 | 9개 페이지 alternates |
| SEO-003 | Bing Webmaster Tools 등록 | ✅ 완료 | GSC import, sitemap 제출 |
| WO-029 | 중국어(zh) 사이트 확장 Phase 1~6 | ✅ 완료 | v2.15 |
| WO-028 | 일본어 사이트 확장 Phase 1~6 | ✅ 완료 | v2.13~v2.14 |
| WO-027 | 시술 URL 영문 slug 전환 | ✅ 완료 | v2.11 |
| WO-026 | 메인 페이지 디자인 개선 | ✅ 완료 | v2.10 |
| WO-025 | Beauty Member Phase 1~3 | ✅ 완료 | v2.9 |
| WO-024 | 병원 파트너 회원제 Phase 1+2 | ✅ 완료 | v2.8 |
| WO-023 | 구글맵 연동 | ✅ 완료 | v2.5 |
