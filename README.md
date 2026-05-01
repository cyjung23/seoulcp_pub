# SeoulClinicPick (SCP) 표준보고서

서울 소재 미용/성형 클리닉 정보를 한영 이중언어로 제공하는 웹서비스의 프로젝트 문서 저장소입니다.

- 서비스: https://seoulcp.com
- 기술스택: Next.js 16, Supabase, Vercel
- 타겟: 한국 미용시술에 관심 있는 외국인
- 지원 언어: 한국어, English, 日本語, 中文(简体)

---

## 최신 버전

**v2.21** (2026-05-01) — GSC 404/Soft 404 대응 + i18n 정체성 정렬

- DATA-001: 클리닉 2225/2226 오염 데이터 정리 (가비아 errdoc URL, "404 Not Found" description NULL 처리)
- SEO-007: CONCERN_SLUG_MAP 오타 수정 (wide-cheekbones-cheeks, commit 22bae85)
- SEO-008: 심술보 → jowl-sagging 슬러그 분리 (concerns DB UPDATE + commit 69c7cf9)
- SEO-009: defaultLocale ko → en 변경 (SCP 정체성 정렬, commit 5ff5212)
- SEO-010: 빈 클리닉(시술·장비 0개) 자동 noindex 처리 (787개 영향, commit 6ee6624)
- 진단: GSC 5xx 21건 자연 해소 확인, 403은 Bytespider 등 악성봇 차단 정상 동작

이전: v2.20 SEO 브랜드 강화 (SEO-005, SEO-006, DX-001), v2.19 크롤링 방어 + 성능 개선, v2.18 Bing 등록

상세: [docs/s2-data/s2-06-changelog.md](docs/s2-data/s2-06-changelog.md)

---

## 활성 작업 요약

| 작업 ID | 작업명 | 상태 |
|---------|--------|------|
| SEO-010 | 빈 클리닉 자동 noindex (787개) | ✅ 완료 |
| SEO-009 | defaultLocale ko → en (정체성 정렬) | ✅ 완료 |
| SEO-008 | 심술보 → jowl-sagging 슬러그 분리 | ✅ 완료 |
| SEO-007 | CONCERN_SLUG_MAP 오타 수정 | ✅ 완료 |
| DATA-001 | 클리닉 2225/2226 오염 데이터 정리 | ✅ 완료 |
| SEO-006 | Organization JSON-LD | ✅ 완료 |
| SEO-005 | seoulclinicpick.com 301 리다이렉트 | ✅ 완료 |
| MON-003 | GEO 추세 비교 (2주차) | 📋 5/2 예정 |
| WO-030 | 로고 디자인 + 파비콘 적용 | ⏳ 진행중 |
| MKT-001 | 외부 마케팅 포스팅 | ⏳ 진행중 |
| WO-033 | middleware → proxy.ts 마이그레이션 | 📋 대기 |
| WO-034 | GSC 유효성 검사 트리거 + 모니터링 | 📋 대기 |
| WO-035 | 빈 클리닉 데이터 자동 보강 (외부 API) | 📋 대기 |
| WO-037 | changelog 분리 + 인수인계 효율화 | 📋 대기 |
| PERF-002 | 서버 측 페이지네이션 | 📋 대기 |
| SEO-004 | Baidu 站长平台 등록 | 🔒 보류 |
| WO-028-B | Partner 페이지 일본어 | 🔒 보류 |
| WO-018 | unmatched 재매칭 | 🔒 보류 |

상세: [docs/s4-active/s4-06-summary-table.md](docs/s4-active/s4-06-summary-table.md)

---

## 다음 작업 우선순위

1. MON-003 — GEO 추세 비교 2주차 (5/2 예정)
2. WO-034 — GSC 유효성 검사 트리거 (5xx/404/Soft 404 정리 확인)
3. WO-037 — changelog 분리 및 인수인계 효율화
4. WO-033 — middleware → proxy.ts 마이그레이션 (Next.js 17 대비)
3. WO-028-B — Partner 일본어 번역
4. SEO-004 — Baidu 站长平台 (해외 등록 재개 시)
5. PERF-002 — 서버 측 페이지네이션 (유입량 증가 시)

상세: [docs/s4-active/s4-05-priority.md](docs/s4-active/s4-05-priority.md)

---

## 최근 마일스톤

| 날짜 | 내용 |
|------|------|
| 04-25 | SEO-005 도메인 리다이렉트, SEO-006 JSON-LD, DX-001 VS Code 설정 (v2.20) |
| 04-21 | SEC-001 크롤링 방어, PERF-001 쿼리 병렬화 (v2.19) |
| 04-20 | WO-030 파비콘 + OG이미지 (v2.19) |
| 04-19 | SEO-003 Bing 등록, Baidu 보류 (v2.18) |
| 04-18 | WO-029 중국어 사이트 확장 (v2.15) |
| 04-17 | WO-025~WO-028 뷰티회원, 디자인, slug, 일본어 |

상세: [docs/s6-roadmap/s6-07-milestones.md](docs/s6-roadmap/s6-07-milestones.md)

---

## 문서 구조

| 폴더 | 내용 |
|------|------|
| s1-master/ | 프로젝트 개요, DB 스키마, 아키텍처, 파일 구조 |
| s2-data/ | 데이터 현황, 커버리지, changelog |
| s3-completed/ | 완료 작업 이력 (SEO, 크롤링, i18n, GEO 등) |
| s4-active/ | 진행/보류 작업, 요약표, 우선순위 |
| s5-decisions/ | 의사결정 로그 (DEC-001~054) |
| s6-roadmap/ | 단기/중기/장기 로드맵, 마일스톤 |

---

*최종 갱신: 2026-04-25*
