# Seoul Clinic Pick (SCP) 표준보고서

**최종 갱신:** 2026-04-15
**총 파일:** 41개 (6개 섹션)

## 프로젝트 요약

Seoul Clinic Pick은 서울 소재 미용/성형 클리닉 정보를 한영 이중언어로 제공하는 웹서비스입니다.
- 서비스 URL: https://seoulcp.com
- 기술스택: Next.js 16, Supabase, Vercel, Cloudflare
- 타겟: 한국 미용시술에 관심 있는 외국인

## 팀 구성 및 역할

| 팀 | 역할 | 담당 영역 |
|---|---|---|
| 원장 (Chase) | 최종 의사결정권자 | 전체 승인, 시술 콘텐츠 감수 |
| 기획1팀 | 총괄 기획·개발 | 코드 수정, DB 변경, 아키텍처, 배포 |
| 기획2팀 | 보조 기획 | 콘텐츠 초안, 데이터 정리, 번역, 리서치 |
| 마케팅1팀 | 마케팅 자문 | 전략 자문, 고도 분석, 마케팅2팀 지원 |
| 마케팅2팀 | 마케팅 실무 | SEO, 광고, 키워드 리서치, SNS, 콘텐츠 마케팅, 트래픽 분석 |

## 팀별 필수 참고 파일

### 모든 팀 공통
- s1-01-overview.md — 프로젝트 개요, 기술스택
- s2-06-changelog.md — 최근 변경 이력
- s5-05-dec038-043.md — 최신 의사결정 로그

### 기획1팀 (총괄 기획·개발)
- s1-02-db-schema.md — DB 테이블 스키마
- s1-03-architecture.md — 아키텍처, 배포 파이프라인
- s1-04-file-structure.md — 코드 파일 구조
- s4-06-summary-table.md — 활성 작업 요약표
- s6-02-short-term.md — 단기 로드맵

### 기획2팀 (보조 기획)
- s2-01-summary.md — 핵심 수치 총괄
- s2-03-coverage.md — 매핑 커버리지 상세
- s2-04-english-status.md — 영문 데이터 입력 현황
- s4-04-hold.md — 보류 작업 (콘텐츠 작업 대상 확인)

### 마케팅팀
- s3-01-seo.md — SEO 완료 작업 이력
- s3-07-wo015-017-geo.md — GEO 최적화 이력
- s4-02-monitoring.md — 모니터링 항목 (인덱싱, GEO 효과)
- s6-04-long-term.md — 장기 로드맵

## 파일 접근 방법

개별 파일은 아래 패턴으로 직접 읽을 수 있습니다:
`https://raw.githubusercontent.com/cyjung23/pubrepo/main/seoulcp-docs/{폴더}/{파일명}`

예시:
https://raw.githubusercontent.com/cyjung23/pubrepo/main/seoulcp-docs/s1-master/s1-01-overview.md

## 의사결정 구조

모든 의사결정은 원장(Chase)이 최종 승인합니다.
- 코드 수정, DB 변경 → 기획1팀을 통해 처리
- 콘텐츠, 번역 → 기획2팀 작성 후 원장 승인
- 마케팅 실무 → 마케팅2팀 실행 후 원장 승인
- 마케팅 전략 자문 → 마케팅1팀에 요청

## 전체 파일 목록

### s1-master (프로젝트 기본 정보)
| 파일 | 내용 | 최종 갱신 |
|---|---|---|
| s1-01-overview.md | 프로젝트 개요, 기술스택 | 2026-04-10 |
| s1-02-db-schema.md | DB 테이블 스키마 | 2026-04-11 |
| s1-03-architecture.md | 아키텍처, 배포 파이프라인 | 2026-04-10 |
| s1-04-file-structure.md | 코드 파일 구조 | 2026-04-11 |

### s2-data (데이터 현황표)
| 파일 | 내용 | 최종 갱신 |
|---|---|---|
| s2-01-summary.md | 핵심 수치 총괄 | 2026-04-11 |
| s2-02-table-records.md | 테이블별 레코드 수 | 2026-04-11 |
| s2-03-coverage.md | 매핑 커버리지 상세 | 2026-04-11 |
| s2-04-english-status.md | 영문 데이터 입력 현황 | 2026-04-11 |
| s2-05-targets.md | 목표 커버리지 | 2026-04-11 |
| s2-06-changelog.md | 버전별 변경 사항 | 2026-04-11 |
| s2-07-validation-queries.md | 검증 쿼리 | 2026-04-11 |

### s3-completed (완료 작업 이력)
| 파일 | 내용 | 최종 갱신 |
|---|---|---|
| s3-01-seo.md | SEO-001~005 | 2026-04-10 |
| s3-02-domain-env.md | DOMAIN-001, ENV-001~004 | 2026-04-10 |
| s3-03-backup.md | BACKUP-001~002 | 2026-04-10 |
| s3-04-wo007-crawling.md | WO-007, WO-007-B | 2026-04-10 |
| s3-05-wo008-013-mapping.md | WO-008~013 | 2026-04-10 |
| s3-06-wo014-i18n.md | WO-014 (i18n 전체) | 2026-04-10 |
| s3-07-wo015-017-geo.md | WO-015, 016, 017 | 2026-04-10 |
| s3-08-wo021-charm.md | WO-021 (참의원 보강) | 2026-04-11 |
| s3-09-timeline.md | 전체 타임라인 | 2026-04-11 |
| s3-10-wo022-encyclopedia.md | WO-022 (백과사전 확장) | 2026-04-12 |

### s4-active (진행/보류 작업)
| 파일 | 내용 | 최종 갱신 |
|---|---|---|
| s4-01-completed-pending.md | 완료 확인 대기 | 2026-04-11 |
| s4-02-monitoring.md | MON-001, MON-002 | 2026-04-11 |
| s4-03-immediate.md | 즉시 실행 항목 | 2026-04-11 |
| s4-04-hold.md | 보류 작업 | 2026-04-11 |
| s4-05-priority.md | 다음 세션 권장 우선순위 | 2026-04-11 |
| s4-06-summary-table.md | 활성 작업 요약표 | 2026-04-11 |

### s5-decisions (의사결정 로그)
| 파일 | 내용 | 최종 갱신 |
|---|---|---|
| s5-01-dec001-010.md | DEC-001~010 | 2026-04-10 |
| s5-02-dec011-020.md | DEC-011~020 | 2026-04-10 |
| s5-03-dec021-030.md | DEC-021~030 | 2026-04-10 |
| s5-04-dec031-037.md | DEC-031~037 | 2026-04-10 |
| s5-05-dec038-043.md | DEC-038~048 | 2026-04-15 |
| s5-06-summary-table.md | 전체 DEC 요약표 | 2026-04-11 |

### s6-roadmap (잔여 작업·로드맵)
| 파일 | 내용 | 최종 갱신 |
|---|---|---|
| s6-01-backlog.md | 잔여 작업 목록 | 2026-04-11 |
| s6-02-short-term.md | 단기 로드맵 (4월) | 2026-04-11 |
| s6-03-mid-term.md | 중기 로드맵 (5월) | 2026-04-11 |
| s6-04-long-term.md | 장기 로드맵 (6월~) | 2026-04-11 |
| s6-05-coverage-targets.md | 커버리지 목표 대비 현황 | 2026-04-11 |
| s6-06-decisions-needed.md | 의사결정 필요 항목 | 2026-04-11 |
| s6-07-milestones.md | 마일스톤 타임라인 | 2026-04-11 |
