]633;E;head -n 5 /tmp/s4-06.backup2.md;9a330de6-4ee7-402a-bc98-44f36734681d]633;C]633;E;head -n 5 /tmp/s4-06.backup.md;05fb77dd-4a17-435c-a8d3-ba1b6222969b]633;C# 작업 현황 요약 (Summary Table)
> 최종 업데이트: 2026-08-11

| ID | 작업명 | 상태 | 비고 |
|----|--------|------|------|
| DATA-005 | 병원명 데이터 부채 진단 + 방향 C 채택 | ✅ 완료 | name_en 한글 혼재 2,627/2,727건(96%) 진단, 4언어 스키마 확장 보류, 개별 요청 시만 정정하는 방향 C 채택. 보완책 1(clinics.name_en_source·name_en_corrected_at 컬럼 신설, clinic 2280 소급 기록) + 보완책 3(clinic-change-requests.md 신설, commit 37f200e) 시행. v2.25 changelog 반영 (commit bab8839, 2026-08-11) |
| DATA-003 | 엄나구모성형외과의원(clinic 2280) 개별 정보 수정 요청 처리 | ✅ 완료 | name_en/phone/description 3건 UPDATE, curl 검증 완료, 병원 회신 발송 (2026-08-11) |
| DATA-004 | 사이트 전체 다국어 필드 구조 개선 + 미매핑 clinic_treatments 다국어화 | 📋 대기 | (1) clinics 언어별 title/description 컬럼 신설 (SEO-013 후보), (2) 언어별 phone 컬럼 검토, (3) 표준 마스터 미매핑 clinic_treatments 규모 조사 및 처리 (엄나구모 20223 가슴성형 등) |
| BACKUP-003 | Supabase DB 백업 `db-backup/` 직접 커밋 방식 전환 | ✅ 완료 | Artifact 방식 → seoulcp_pub 직접 커밋으로 전환, 워크플로 #84 race condition 해결, 임시 테이블 _backup_data002_descriptions 정리 (48줄 감소), Egress 39%, DEC-055 (2026-07-04) |
| DATA-002 | clinics.description HTML 엔티티 정리 | ✅ 완료 | 19건 정정 (8종 엔티티 일괄 + 한글 수치 엔티티 수동 복원), 백업 _backup_data002_descriptions, GA4 표시 이슈는 React 표준 인코딩으로 사이트 영향 없음 (2026-06-14) |
| WO-039 | 404 처리 통합 (다국어 not-found + LEGACY_SLUG_MAP + middleware 일원화) | ✅ 완료 | next.config redirects 제거, 한글/구영문 slug 백링크 보호, 4언어 404 페이지, commits 9504a62/eb09a1d (2026-05-07) |
| SEO-011 | zh.json 번역 누락 수정 | ✅ 완료 | MISSING_MESSAGE (home/common.siteName) 해결, ko/en/ja 표준 구조로 재작성, commit d3e449a (2026-05-02) |
| SEO-010 | 빈 클리닉 자동 noindex 처리 | ✅ 완료 | 시술·장비 0개 클리닉 787개 영향, commit 6ee6624 |
| SEO-009 | defaultLocale ko → en 변경 | ✅ 완료 | SCP 정체성 정렬 (해외 사용자 표준), commit 5ff5212 |
| SEO-008 | 심술보 → jowl-sagging 슬러그 분리 | ✅ 완료 | concerns DB UPDATE + commit 69c7cf9 |
| SEO-007 | CONCERN_SLUG_MAP 오타 수정 | ✅ 완료 | wide-cheekbones-cheeks, commit 22bae85 |
| DATA-001 | 클리닉 2225/2226 오염 데이터 정리 | ✅ 완료 | website/description NULL 처리 (가비아 errdoc URL 제거) |
| MON-003 | GEO 추세 비교 (2주차) | 📋 5/2 예정 | GA4 + GSC 데이터 비교 |
| WO-033 | middleware → proxy.ts 마이그레이션 | 📋 대기 | Next.js 16 deprecation 대응 + locale prefix 자동보정 |
| WO-034 | GSC 유효성 검사 트리거 + 모니터링 | 📋 대기 | 5xx/404/Soft 404 정리 확인, MON-003과 함께 진행 |
| WO-035 | 빈 클리닉 데이터 자동 보강 | 📋 대기 | 네이버 플레이스 + 심평원 API 활용 |
| WO-040 | Supabase Data API explicit-grant 정책 대응 | 📋 대기 | 2026-10-30 강제 적용 전 완료, Security Advisor 점검 + 신규 테이블 GRANT 템플릿 SOP화 |
| WO-037 | changelog 분리 + 인수인계 효율화 | 📋 대기 | 활성/아카이브 분리 + 인덱스 신설 |
| SEO-006 | Organization JSON-LD (브랜드 인식 강화) | ✅ 완료 | layout.tsx에 구조화 데이터 추가, commit 0f12fcf |
| SEO-005 | seoulclinicpick.com → seoulcp.com 301 리다이렉트 | ✅ 완료 | Vercel + 가비아 DNS |
| DX-001 | VS Code CSS 경고 비활성화 | ✅ 완료 | Tailwind v4 @theme 호환, commit e8c6f92 |
| SEC-001 | 크롤링 방어 (robots.txt + Rate Limiting) | ✅ 완료 | 2026-04-21 |
| PERF-001 | wiki DB 쿼리 병렬화 (Promise.all) | ✅ 완료 | 2026-04-21 |
| MON-001 | Naver/Google 인덱싱 확인 | ✅ 2차 완료 | Google 4언어 정상, Naver /ko/ 집중 전략 전환 |
| MON-002 | GEO 효과 분석 | ✅ 1차 완료 | 6개국 유입, Reddit 포스팅 효과 확인 |
| WO-031 | 관리자 대시보드 pending 뱃지 | ✅ 완료 | 가입승인/수정요청 건수 표시, commit f0f97d5 |
| MKT-001 | 외부 마케팅 포스팅 | ⏳ 진행중 | Reddit(영어), Medium(영어+일본어), note.com(일본어) |
| MON-003 | GEO 추세 비교 (2주 후) | 📋 대기 | 5/2 예정 |
| WO-030 | 로고 디자인 + 파비콘 적용 | ⏳ 진행중 | 파비콘 완료, 로고 산출물 진행중 |
| PERF-002 | 서버 측 페이지네이션 | 📋 대기 | 유입량 증가 시 검토 |
| SEO-004 | Baidu 站长平台 등록 | 🔒 보류 | 해외 등록 일시 중단, 재개 시 진행 |
| WO-028-B | Partner 페이지 일본어 | 🔒 보류 | 9파일 120개소, 일본인 파트너 유입 시 진행 |
| WO-018 | unmatched 재매칭 | 🔒 보류 | — |
| WO-017-v2 | 아이콘 재진행 | 🔒 보류 | — |
| WO-019 | Naver body 인덱싱 | 🔒 보류 | — |
| WO-020 | mojibake 수정 | 🔒 보류 | — |
| WO-029-SEO | HTML hreflang 4언어 + 인덱싱 요청 | ✅ 완료 | 9개 페이지 alternates |
| SEO-003 | Bing Webmaster Tools 등록 | ✅ 완료 | GSC import, sitemap 제출 |
| WO-029 | 중국어(zh) 사이트 확장 Phase 1~6 | ✅ 완료 | v2.15 |
| WO-028 | 일본어 사이트 확장 Phase 1~6 | ✅ 완료 | v2.13~v2.14 |
| WO-027 | 시술 URL 영문 slug 전환 | ✅ 완료 | v2.11 |
| WO-026 | 메인 페이지 디자인 개선 | ✅ 완료 | v2.10 |
| WO-025 | Beauty Member Phase 1~3 | ✅ 완료 | v2.9 |
| WO-024 | 병원 파트너 회원제 Phase 1+2 | ✅ 완료 | v2.8 |
| WO-023 | 구글맵 연동 | ✅ 완료 | v2.5 |
