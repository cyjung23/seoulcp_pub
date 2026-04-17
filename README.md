# SeoulClinicPick (SCP) 표준보고서

서울 소재 미용/성형 클리닉 정보를 한영 이중언어로 제공하는 웹서비스의 프로젝트 문서 저장소입니다.

- 서비스: https://seoulcp.com
- 기술스택: Next.js 16, Supabase, Vercel, Cloudflare
- 타겟: 한국 미용시술에 관심 있는 외국인

---

## 최신 버전

**v2.14** (2026-04-17) — 일본어 사이트 인프라 + 데이터 (WO-028 Phase 1,2,4)
- i18n 인프라 ja 로케일 추가, messages/ja.json 생성
- standard_treatments name_ja 120개, concerns concern_group_ja 13개 입력
- /ja/ 경로 접속 가능 (콘텐츠 일본어 표시는 Phase 3 이후)

이전: v2.11 slug 전환 (WO-027), v2.10 메인 디자인 (WO-026), v2.9 뷰티회원제 (WO-025)

상세: [docs/s2-data/s2-06-changelog.md](docs/s2-data/s2-06-changelog.md)

---

## 활성 작업 요약

| 작업 ID | 작업명 | 상태 |
|---------|--------|------|
| WO-028 | 일본어 사이트 확장 | 🔧 진행중 (Phase 1,2,4 완료) |
| WO-027 | 시술 URL 영문 slug 전환 | ✅ 완료 (v2.11) |
| WO-026 | 메인 페이지 디자인 개선 | ✅ 완료 (v2.10) |
| WO-025 | 뷰티회원제 Phase 1+2+3 | ✅ 완료 (v2.9) |
| WO-024 | 병원 파트너 회원제 Phase 1+2 | ✅ 완료 (v2.8) |
| WO-023 | 구글맵 연동 | ✅ 완료 (v2.5) |
| MON-001 | Naver 인덱싱 확인 | ⏳ 대기 (4/20 이후) |
| MON-002 | GEO 효과 확인 | ⏳ 대기 (4/24 이후) |
| 일본어 사이트 | 다국어 확장 | 📋 계획 (즉시 착수 가능) |
| WO-018 | unmatched 재매칭 | 🔒 보류 |
| WO-017-v2 | 아이콘 재진행 | 🔒 보류 |
| WO-019 | Naver body 인덱싱 | 🔒 보류 |
| WO-020 | mojibake 수정 | 🔒 보류 |

상세: [docs/s4-active/s4-06-summary-table.md](docs/s4-active/s4-06-summary-table.md)

---

## 다음 작업 우선순위

1. **WO-028 Phase 3** — 일본어 페이지 로케일 로직 변경
2. MON-001 — Naver 인덱싱 확인 (4/20 이후)
3. MON-002 — GEO 효과 확인 (4/24 이후)
4. WO-018 — unmatched 재매칭
5. WO-017-v2 — 아이콘 디자인 교체

상세: [docs/s4-active/s4-05-priority.md](docs/s4-active/s4-05-priority.md)

---

## 최근 마일스톤

| 날짜 | 내용 |
|------|------|
| 04-17 | WO-025 뷰티회원제, WO-026 메인 디자인, WO-027 slug 전환, WO-028 일본어 Phase 1,2,4 완료 |
| 04-16 | WO-024 병원 파트너 회원제 완료 |
| 04-15 | WO-023 구글맵 연동 완료 |
| 04-14 | DOC-001 문서 이관, 팀 구성 확정 |
| 04-12 | WO-022 백과사전 확장 완료 |

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

*최종 갱신: 2026-04-17*
