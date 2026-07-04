# S3-03: 백업 (BACKUP-001 ~ BACKUP-003)

**최종 갱신:** 2026-07-04

## 저장소 구조 (2026-05 이후 이원화)

| 저장소 | 공개 여부 | 저장 내용 |
|--------|-----------|-----------|
| `cyjung23/seoulcp` | Private | 실제 서비스 소스코드 (Next.js 앱) |
| `cyjung23/seoulcp_pub` | Public | SCP 프로젝트 표준문서 저장소 (docs/ + db-backup/) |

- 표준문서(`docs/`)는 `seoulcp_pub`에만 존재하며 `seoulcp`에는 저장하지 않는다.
- Supabase DB 덤프(`db-backup/`)도 `seoulcp_pub`에만 커밋된다. `seoulcp`에는 별도 백업을 두지 않는다.
- 소스코드 자체의 백업은 Git 히스토리(`seoulcp` 저장소)가 그 역할을 겸한다.

## BACKUP-001 — 프로젝트 데이터 파일 GitHub 백업
- 작업일: 2026-04-07. 상태: 완료.
- 위치: `seoulcp` 저장소 `backup/` 폴더
- 구성: data/(JSON 57, CSV 4, Python 8, SQL 1), scripts/(raw_texts 672, CSV, Python, 보고서), scraper/
- 총 902개 파일, 16.16MB

## BACKUP-002 — Supabase DB 자동 백업 워크플로우 (초기 방식)
- 작업일: 2026-04-07. 상태: 종료(BACKUP-003으로 대체).
- 커밋: 5233bb8, 7eab8e8, 3c2055c, e4de33d
- 방식: `.github/workflows/db-backup.yml`이 매일 UTC 00:00 (KST 09:00) `pg_dump` 실행 후 결과를 GitHub Artifacts로 90일 보존.
- 한계: Artifact는 저장소 외부 스토리지이므로 raw 접근 불가, 90일 후 자동 삭제, 변경 이력 추적 불가.

## BACKUP-003 — `db-backup/` 직접 커밋 방식 (현재 운영 방식)
- 전환일: 2026-06. 상태: 운영 중.
- 방식: 워크플로가 `pg_dump` 결과를 `seoulcp_pub/db-backup/` 디렉토리에 직접 커밋/푸시.
- 산출물:
  - `db-backup/data.sql` — 테이블 데이터
  - `db-backup/schema.sql` — 스키마
- 장점: Git 히스토리로 변경 추적, raw URL로 즉시 접근, 영구 보존.
- 최근 이슈 및 조치:
  - 워크플로 #84 race condition 발생 → 재실행으로 해결.
  - 임시 테이블 `_backup_data002_descriptions` 제거 → 백업에서 총 48줄 감소.
  - Supabase Egress 사용량 39% (정상 범위).
- 트리거: 매일 KST 09:00 자동 실행 + GitHub Actions 탭에서 수동 실행 가능.

## 백업 검증 방법

최신 백업 pull:
`cd /workspaces/seoulcp_pub && git pull origin main`

특정 테이블 존재 여부 확인:
`grep "테이블명" db-backup/data.sql`

스키마 확인:
`grep "CREATE TABLE" db-backup/schema.sql | head -20`

## 관련 의사결정
- DEC-048: 표준문서 저장소를 `seoulcp_pub`로 이관
- DEC-055: 백업 방식을 Artifact → `db-backup/` 직접 커밋으로 전환
