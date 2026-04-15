# S3-03: 백업 (BACKUP-001 ~ BACKUP-002)

**최종 갱신:** 2026-04-10

## BACKUP-001 — 프로젝트 데이터 파일 GitHub 백업
- 작업일: 2026-04-07. 상태: 완료.
- backup/ 폴더: data/(JSON 57, CSV 4, Python 8, SQL 1), scripts/(raw_texts 672, CSV, Python, 보고서), scraper/
- 총 902개 파일, 16.16MB

## BACKUP-002 — Supabase DB 자동 백업 워크플로우
- 작업일: 2026-04-07. 상태: 완료.
- 커밋: 5233bb8, 7eab8e8, 3c2055c, e4de33d
- .github/workflows/db-backup.yml
- 매일 UTC 00:00 (KST 09:00) pg_dump, GitHub Artifacts 90일 보존
- Transaction pooler(6543), SUPABASE_DB_URL 시크릿, workflow write 권한
