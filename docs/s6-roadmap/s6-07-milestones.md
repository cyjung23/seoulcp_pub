# S6-07: 마일스톤 타임라인

**최종 갱신:** 2026-04-25

| 날짜 | 마일스톤 |
|------|---------|
| 2026-04-25 | SEO-005 seoulclinicpick.com 301 리다이렉트, SEO-006 Organization JSON-LD, DX-001 VS Code 설정 (v2.20) |
| 2026-04-21 | SEC-001 크롤링 방어, PERF-001 wiki 쿼리 병렬화 (v2.19) |
| 2026-04-20 | WO-030 파비콘 SCP 3색 적용, OG이미지 다크 디자인, MON-001 1차 확인 (v2.19) |
| 2026-04-19 | SEO-003 Bing Webmaster Tools 등록, Baidu 보류 (v2.18) |
| 2026-04-18 | WO-029 중국어(zh) 사이트 확장 완료 (v2.15) |
| 2026-04-17 | WO-025 뷰티회원제, WO-026 메인 디자인, WO-027 slug 전환, WO-028 일본어 Phase 1,2,4 |
| 2026-04-16 | WO-024 병원 파트너 회원제 완료 |
| 2026-04-15 | WO-023 구글맵 연동 완료 |
| 2026-04-14 | DOC-001 문서 이관, 팀 구성 확정 |
| 2026-04-12 | WO-022 백과사전 확장 완료 |
| 2026-04-11 | WO-021 완료, 인덱싱 31건, 표준보고서 GitHub 전환 |
| 2026-04-10 | WO-014 전량 완료, WO-015/016/017 완료, Naver 20건 제출 |

---

### 2026-04-25 (v2.20) — SEO 브랜드 강화 + 도메인 리다이렉트
- **SEO-005**: seoulclinicpick.com 도메인 구매 (가비아), Vercel 301 리다이렉트 설정
  - A @ → 76.76.21.21, CNAME www → cname.vercel-dns.com
  - seoulclinicpick.com + www → seoulcp.com 301 permanent redirect
  - Cloudflare 미사용 확인 → 기술스택에서 Cloudflare 삭제
- **SEO-006**: Organization JSON-LD 추가 (commit 0f12fcf)
  - name: "Seoul Clinic Pick", alternateName: ["서울클리닉픽", "SeoulClinicPick", "SCP", "seoulcp"]
  - sameAs: seoulclinicpick.com, YouTube, Naver 블로그
  - Google 브랜드 인식 강화 목적 (검색어 "seoul clinic pick" → seoulcp.com 노출 유도)
- **DX-001**: VS Code .vscode/settings.json 추가 (commit e8c6f92)
  - Tailwind CSS v4 @theme inline 문법 경고 해소

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
