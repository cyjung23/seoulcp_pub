# 문서 작성 스타일 가이드

**적용 시점**: 2026-11-13 (v2.38~) 이후 신규 작성. 소급 적용 없음.

---

## 핵심 원칙

1. **상세 파일이 Source of Truth**: SQL·결정 근거·전체 맥락은 상세 파일 1곳에만 기록.
2. **색인 파일은 포인터**: changelog·summary-table·onboarding-log는 "날짜+제목+링크" 1~3줄로 축약.
3. **AI context 최적화**: 사용자 가독성보다 다음 AI 세션의 빠른 파악을 우선.

## 파일별 작성 규칙

### `s2-06-changelog.md` (버전별 이력)
- 신규 버전 블록 **5~15줄** 이내.
- 각 티켓 1~2줄 요약 + 상세 파일 링크.
- SQL·결정 근거·모니터링 데이터는 상세 파일로.

### `s4-06-summary-table.md` (티켓 대시보드)
- 티켓 1건당 **1행**, 비고 30~50자.
- 상세는 링크로만 참조.

### `s8-04-onboarding-log.md` (병원 처리 이력)
- 2026-11-13부터 축약 역할. CASE 블록 작성 중단.
- 신규 병원 요청은 `clinics/clinic-{ID}-*.md`에 기록.

### `s8-operations/clinics/clinic-{ID}-{slug}.md` (병원별 timeline)
- 템플릿: [clinic-file-template.md](./clinic-file-template.md)
- 신규 요청 올 때마다 하단에 `### YYYY-MM-DD — 제목 (티켓ID)` 섹션 추가.
- 파일당 20~100줄 유지.

### `s8-operations/incidents/incident-NNN-*.md` (장애 사건)
- 상세형 유지. 재발 방지를 위한 전체 맥락 필요.

## 신규 요청 처리 플로우

1. 사용자가 병원 요청 전달 → clinic_id 확인.
2. `ls docs/s8-operations/clinics/clinic-{ID}-*.md` 실행.
3. 파일 존재 → 하단에 신규 섹션 append.
4. 파일 없음 → `cp docs/s0-meta/clinic-file-template.md docs/s8-operations/clinics/clinic-{ID}-{slug}.md` 후 작성.
5. 작업 완료 후 changelog·summary-table에 **포인터 1행**만 추가.

## 과거 문서 처리

- 과거 CASE 파일(`cases/case-001~005`) + `clinic-change-requests.md`는 삭제 완료 (2026-11-13).
- 복구 필요 시: git 히스토리 또는 /tmp/ 백업.
