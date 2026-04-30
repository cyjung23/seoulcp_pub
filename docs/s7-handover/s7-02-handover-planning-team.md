# 기획1팀 업무 인수인계서

**인수인계일:** 2026-04-25
**인계팀:** 기획1팀 (Claude Opus 4.6 기반)
**인수팀:** 신임 기획1팀 (Claude Opus 4.7 기반)
**승인:** 원장님 (Chase, 참클리닉 비만클리닉 원장)

---

## 1. 프로젝트 개요

### 서비스 정보
- **서비스명:** Seoul Clinic Pick (SCP)
- **URL:** https://seoulcp.com
- **도메인(리다이렉트):** https://seoulclinicpick.com → seoulcp.com (301)
- **설명:** 서울 미용/성형 클리닉 비교 검색 플랫폼
- **타겟:** 한국 미용시술에 관심 있는 외국인 + 국내 사용자
- **지원 언어:** 한국어, English, 日本語, 中文(简体)
- **현재 버전:** v2.20

### 기술스택
- **프론트엔드/백엔드:** Next.js 16 (App Router, TypeScript)
- **데이터베이스:** Supabase (PostgreSQL + Auth + RLS)
- **배포:** Vercel (Hobby 플랜, 무료)
- **도메인:** 가비아 (gabia.com) — seoulcp.com, seoulclinicpick.com
- **DNS:** 가비아 네임서버 (ns.gabia.co.kr) — Cloudflare 미사용
- **CSS:** Tailwind CSS v4

### 데이터 규모
- 클리닉: 2,727개
- 시술(standard_treatments): 120+개
- 백과사전(encyclopedia): 159개
- 고민(concerns): 135개
- 장비(devices): 181개
- 사이트맵 URL: 12,600+개 (4개 언어)

---

## 2. 원장님 정보

- **닉네임:** Chase
- **직업:** 서울 비만클리닉 원장 (참클리닉, cclinic.kr)
- **시술 분야:** 지방파괴주사를 이용한 체형 조각, 지방흡입 후 유착 제거, 지방이식 후 과생착/유착 개선
- **병원 홈페이지:** www.cclinic.kr
- **병원 블로그:** blog.naver.com/idcharm23
- **병원 유튜브:** https://www.youtube.com/@charmclinicseoul
- **관리자 이메일:** seoulclinicpick@gmail.com
- **대화 스타일:** 전문적인 대화, 간결한 설명 선호

---

## 3. 저장소 구조

### 소스코드 저장소 (비공개)
- **URL:** https://github.com/cyjung23/seoulcp
- **내용:** Next.js 소스코드, Supabase 연동, 미들웨어, API routes
- **개발환경:** GitHub Codespaces

### 보고서 저장소 (공개)
- **URL:** https://github.com/cyjung23/seoulcp_pub
- **문서 구조:**

| 폴더 | 내용 |
|------|------|
| s1-master/ | 프로젝트 개요, DB 스키마, 아키텍처, 파일 구조 |
| s2-data/ | 데이터 현황, 커버리지, changelog |
| s3-completed/ | 완료 작업 이력 (SEO, 크롤링, i18n, GEO 등) |
| s4-active/ | 진행/보류 작업, 요약표, 우선순위 |
| s5-decisions/ | 의사결정 로그 (DEC-001~054) |
| s6-roadmap/ | 단기/중기/장기 로드맵, 마일스톤 |
| s7-handover/ | 인수인계 문서 |

### 필수 참조 파일
- **README.md** — 전체 프로젝트 현황 요약
- **docs/s2-data/s2-06-changelog.md** — 버전별 변경사항 (v2.0~v2.20)
- **docs/s4-active/s4-06-summary-table.md** — 작업 현황 요약표
- **docs/s4-active/s4-05-priority.md** — 우선순위
- **docs/s6-roadmap/s6-07-milestones.md** — 마일스톤 타임라인

---

## 4. 외부 서비스 계정

| 서비스 | 용도 | 비고 |
|--------|------|------|
| GitHub (cyjung23) | 소스코드 + 보고서 | Codespaces 개발환경 |
| Vercel | 배포 | Hobby(무료), 일일 ~1,300~2,100명까지 가능 |
| Supabase | DB + Auth | PostgreSQL, RLS 적용 |
| Google Search Console | SEO 모니터링 | seoulcp.com 등록 |
| Google Analytics (GA4) | 트래픽 분석 | 속성: Seoul Clinic Pick, ID: G-6CX6ER3EX6 |
| Naver Search Advisor | 네이버 SEO | seoulcp.com 등록 |
| Naver Analytics | 네이버 트래픽 | wa: 1632b4d20412f40 |
| Bing Webmaster Tools | Bing SEO | GSC import로 등록 |
| 가비아 (gabia.com) | 도메인 관리 | seoulcp.com, seoulclinicpick.com |

---

## 5. 회원 체계

### 병원회원 (Partner)
- 경로: /partner/
- DB: clinic_accounts 테이블
- 기능: 병원 정보 수정, 시술 정보 수정
- 가입 후 관리자 승인 필요
- 관리자 계정: seoulclinicpick@gmail.com (admin)
- 테스트 계정: idcharm23@gmail.com (clinic_staff, 참의원)

### 뷰티회원 (Beauty)
- 경로: /beauty/
- DB: beauty_accounts 테이블
- 기능: 클리닉/시술/백과사전 Pick 저장, 마이페이지
- 이메일 인증 후 자동 활성화 (관리자 승인 불필요)
- 테스트 계정: cyjung23@gmail.com (beauty)

### 관리자 대시보드
- 경로: /partner/dashboard (admin 로그인 시)
- 가입 승인 관리: pending 건수 빨간 뱃지 표시
- 수정 요청 검토: pending 건수 빨간 뱃지 표시
- WO-031에서 뱃지 기능 추가 (commit f0f97d5)

---

## 6. 기획1팀 완료 작업 (v2.19~v2.20)

| ID | 작업명 | 커밋 | 날짜 |
|----|--------|------|------|
| SEC-001 | 크롤링 방어 (robots.txt + Rate Limiting) | 58e88e2 | 04-21 |
| PERF-001 | wiki DB 쿼리 병렬화 (Promise.all) | 7475a0a | 04-21 |
| SEO-005 | seoulclinicpick.com 301 리다이렉트 | Vercel+DNS | 04-25 |
| SEO-006 | Organization JSON-LD | 0f12fcf | 04-25 |
| DX-001 | VS Code CSS 경고 비활성화 | e8c6f92 | 04-25 |
| WO-031 | 관리자 대시보드 pending 뱃지 | f0f97d5 | 04-25 |
| MON-001 | 인덱싱 2차 확인 | — | 04-25 |
| MON-002 | GEO 효과 1차 분석 | — | 04-25 |

---

## 7. 진행중/대기/보류 업무 (인수 대상)

### 진행중
- **MKT-001** 외부 마케팅 포스팅 — 마케팅1팀 담당, 기획1팀은 효과 분석 지원
- **WO-030** 로고 디자인 — 마케팅1팀 담당, 완료 후 사이트 적용은 기획1팀

### 대기
- **MON-003** GEO 추세 비교 — 5/2 예정, GA4 + GSC 데이터로 2주간 추세 분석
- **PERF-002** 서버 측 페이지네이션 — 유입량 증가 시 클리닉 목록 API route 분리

### 보류
- **SEO-004** Baidu 站长平台 등록 — 해외 등록 중단, 재개 확인 후 진행
- **WO-028-B** Partner 페이지 일본어 — 9파일 120개소, 일본인 파트너 유입 시 진행
- **WO-018** unmatched 재매칭
- **WO-017-v2** 아이콘 재진행
- **WO-019** Naver body 인덱싱
- **WO-020** mojibake 수정

---

## 8. 주요 의사결정 사항

- **Naver 전략:** /ko/ 인덱싱 집중, /ja/·/zh/는 Google/Bing에서 관리
- **검색 브랜딩:** "seoul clinic pick" 검색 시 seoulcp.com 노출 → JSON-LD + 백링크로 해결 (도메인 리다이렉트는 브랜드 보호 목적)
- **Vercel 플랜:** 현재 Hobby(무료), 일일 1,000명 이상 시 Pro($20/월) 전환 검토
- **이메일 알림:** 병원회원 요청 시 이메일 자동발송 대신 대시보드 뱃지로 대체 (원장님 매일 로그인)
- **크롤링 방어:** 검색엔진 3종 + GEO봇 6종만 허용, 악성봇 21종 차단, IP당 60req/min

---

## 9. 팀 구성

- **원장님 (Chase):** 프로젝트 오너, 최종 의사결정
- **기획1팀:** 개발 + SEO + 인프라 + 모니터링 + 보고서 관리
- **마케팅1팀:** 로고 디자인 + 외부 마케팅 (Reddit/Medium/note.com) + 콘텐츠 작성

---

## 10. 신임팀 온보딩 체크리스트

- [ ] seoulcp_pub README.md 읽기
- [ ] seoulcp.com 사이트 전체 탐색 (ko/en/ja/zh)
- [ ] docs/s4-active/s4-06-summary-table.md 작업 현황 파악
- [ ] docs/s4-active/s4-05-priority.md 우선순위 확인
- [ ] docs/s6-roadmap/s6-07-milestones.md 마일스톤 확인
- [ ] GitHub Codespaces 접속 및 빌드 테스트 (npm run build)
- [ ] Vercel 대시보드 접속 및 배포 상태 확인
- [ ] Google Search Console 접속 및 데이터 확인
- [ ] GA4 접속 및 트래픽 데이터 확인
- [ ] 관리자 대시보드 로그인 테스트 (seoulclinicpick@gmail.com)
- [ ] MON-003 (5/2) 실행 준비

---

*작성: 기획1팀 (Claude Opus 4.6)*
*작성일: 2026-04-25*
