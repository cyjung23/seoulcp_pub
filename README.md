# SeoulClinicPick (SCP) 표준보고서

서울 소재 미용/성형 클리닉 정보를 한영 이중언어로 제공하는 웹서비스의 프로젝트 문서 저장소입니다.

- **서비스**: https://seoulcp.com
- **기술스택**: Next.js 16, Supabase, Vercel
- **타겟**: 한국 미용시술에 관심 있는 외국인
- **지원 언어**: 한국어, English, 日本語, 中文(简体)

---

## 저장소 구조 및 작업 방식 (AI 필독)

### 저장소 이원화

| 저장소 | 공개 | 내용 |
|---|---|---|
| `cyjung23/seoulcp` | Private | 서비스 소스코드 |
| `cyjung23/seoulcp_pub` | Public | 표준문서(`docs/`) + DB 덤프(`db-backup/`) + `README.md` |

**원칙**: 표준문서는 `seoulcp_pub`에만 저장한다. `seoulcp`에 표준문서나 DB 백업을 두지 않는다.

### 작업 환경

원장님은 `seoulcp` Codespaces 하나에서 터미널만 사용한다. 그 안에 `seoulcp_pub`가 clone되어 있다.

- `/workspaces/seoulcp/` — 소스코드 (Private)
- `/workspaces/seoulcp_pub/` — 표준문서 저장소 (Public, 현재 사용)
- `/workspaces/pubrepo/` — 과거 문서 저장소 (활성 아님)

### AI가 지켜야 할 규칙

1. **파일 생성/수정은 heredoc 명령으로만 제공**: `cat > /workspaces/seoulcp_pub/docs/... << 'SCP_EOF' ... SCP_EOF` 형식. 원장님은 이 명령을 터미널에 붙여넣기만 한다.
2. **파일 경로는 절대경로 사용**: `/workspaces/seoulcp_pub/docs/...` 형식으로 작성.
3. **긴 파일 수정 시 3단계 안전 방식**: 원본을 `/tmp/`에 백업 → 임시 블록 생성 → `head`+`cat`+`tail`로 재조립.
4. **git 명령은 별도로 제공**: 파일 작업 후 `cd /workspaces/seoulcp_pub && git add ... && git commit -m "..." && git push` 명령을 따로 안내.

### 백업

Supabase DB는 매일 KST 09:00에 GitHub Actions가 `pg_dump` → `seoulcp_pub/db-backup/`에 직접 커밋. 상세: `docs/s3-completed/s3-03-backup.md`.

**관련 의사결정**: DEC-048 (저장소 이원화), DEC-055 (백업 아키텍처).

---

## 문서 구조

| 폴더 | 내용 | 대표 문서 |
|---|---|---|
| `s1-master/` | 프로젝트 개요, DB 스키마, 아키텍처, 파일 구조 | — |
| `s2-data/` | 데이터 현황, 커버리지, changelog | `s2-06-changelog.md` (최신 버전·변경 이력) |
| `s3-completed/` | 완료 작업 이력 (SEO, 크롤링, i18n, GEO 등) | — |
| `s4-active/` | 진행/보류 작업, 요약표, 우선순위 | `s4-06-summary-table.md` (활성 작업 현황), `s4-05-priority.md` (우선순위) |
| `s5-decisions/` | 의사결정 로그 (DEC-001~) | — |
| `s6-roadmap/` | 단기/중기/장기 로드맵, 마일스톤 | `s6-07-milestones.md` (최근 마일스톤) |
| `s7-handover/` | 세션 간 인수인계 문서 | — |
| `s8-operations/` | 운영 가이드, 케이스 로그 | `s8-01-clinic-onboarding-playbook.md` (온보딩 프로세스), `s8-04-onboarding-log.md` (케이스 로그) |

**현재 상태(최신 버전·활성 작업·우선순위·마일스톤)는 각 대표 문서에서 직접 확인**한다. README에는 변동 정보를 두지 않는다.
