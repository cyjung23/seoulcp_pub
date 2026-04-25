# SeoulClinicPick (SCP) 표준보고서

서울 소재 미용/성형 클리닉 정보를 한영 이중언어로 제공하는 웹서비스의 프로젝트 문서 저장소입니다.

- 서비스: https://seoulcp.com
- 기술스택: Next.js 16, Supabase, Vercel
- 타겟: 한국 미용시술에 관심 있는 외국인
- 지원 언어: 한국어, English, 日本語, 中文(简体)

---

## 최신 버전

**v2.20** (2026-04-25) — SEO 브랜드 강화 + 도메인 리다이렉트

- SEO-005: seoulclinicpick.com → seoulcp.com 301 리다이렉트 (Vercel + 가비아 DNS)
- SEO-006: Organization JSON-LD 추가 (브랜드 "Seoul Clinic Pick" 구조화 데이터)
- DX-001: VS Code CSS 경고 비활성화 (Tailwind v4 @theme 호환)
- Cloudflare 미사용 확인 → 기술스택에서 삭제

이전: v2.19 크롤링 방어 + 성능 개선 (SEC-001, PERF-001), v2.18 Bing 등록, v2.15 중국어 확장 (WO-029)

상세: [docs/s2-data/s2-06-changelog.md](docs/s2-data/s2-06-changelog.md)

---

## 활성 작업 요약

| 작업 ID | 작업명 | 상태 |
|---------|--------|------|
| SEO-006 | Organization JSON-LD (브랜드 인식 강화) | ✅ 완료 |
| SEO-005 | seoulclinicpick.com 301 리다이렉트 | ✅ 완료 |
| SEC-001 | 크롤링 방어 (robots.txt + Rate Limiting) | ✅ 완료 |
| PERF-001 | wiki DB 쿼리 병렬화 | ✅ 완료 |
| WO-030 | 로고 디자인 + 파비콘 적용 | ⏳ 진행중 |
| MON-001 | Naver 인덱싱 확인 | ⏳ 진행중 |
| MON-002 | GEO 효과 확인 | ⏳ 대기 |
| PERF-002 | 서버 측 페이지네이션 | 📋 대기 |
| SEO-004 | Baidu 站长平台 등록 | 🔒 보류 |
| WO-028-B | Partner 페이지 일본어 | 🔒 보류 |
| WO-018 | unmatched 재매칭 | 🔒 보류 |

상세: [docs/s4-active/s4-06-summary-table.md](docs/s4-active/s4-06-summary-table.md)

---

## 다음 작업 우선순위

1. MON-001 — Naver 인덱싱 확인
2. MON-002 — GEO 효과 확인
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
