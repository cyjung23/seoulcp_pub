# AI 온보딩 색인

**목적**: seoulcp 운영 AI 세션 간 context 인수인계용. 신규 AI 세션 시작 시 이 파일부터 읽기.

**원칙**: 상세 파일이 Source of Truth, 색인 파일은 포인터 역할.

---

## 1분 파악 (세션 시작 시 최소 읽기)

- 작업 티켓 대시보드: [../s4-active/s4-06-summary-table.md](../s4-active/s4-06-summary-table.md)
- 병원별 변경 이력: [../s8-operations/clinics/](../s8-operations/clinics/) — 파일명 `clinic-{ID}-{slug}.md`
- 최근 버전 요약: [../s2-data/s2-06-changelog.md](../s2-data/s2-06-changelog.md) 상단

## 5분 파악 (작업 착수 전)

- 문서 작성 규칙: [doc-style-guide.md](./doc-style-guide.md)
- 병원 이력 템플릿: [clinic-file-template.md](./clinic-file-template.md)
- 주요 결정 기록: `grep "DEC-" docs/ -rn`

## 상세 자료 (필요 시)

- 장애 사건: [../s8-operations/incidents/](../s8-operations/incidents/)
- DB 백업 (매일 KST 09:00 자동): [../../db-backup/](../../db-backup/)
- 전체 변경 이력: [../s2-data/s2-06-changelog.md](../s2-data/s2-06-changelog.md)

## 신규 AI 팀 체크리스트

1. 이 파일 읽기
2. s4-06-summary-table.md에서 진행 중 티켓 확인
3. doc-style-guide.md 숙지
4. 사용자 요청 처리 시: clinic-ID 확인 → `ls docs/s8-operations/clinics/clinic-{ID}-*.md` → 있으면 append, 없으면 템플릿 복사해서 신규 작성

## 저장소 역할 분담

- **cyjung23/seoulcp** (Private): Next.js 16 운영 코드, Vercel 자동배포
- **cyjung23/seoulcp_pub** (Public, 이 저장소): 문서 + DB 일일 백업
