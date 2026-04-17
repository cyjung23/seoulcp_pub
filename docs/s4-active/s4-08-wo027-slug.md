# S4-08: WO-027 시술 URL 영문 slug 전환

**최종 갱신:** 2026-04-17
**상태:** 완료 (Phase 1~6 전체 완료, v2.11 배포)

## 목표
시술 URL을 한글 인코딩에서 영문 slug로 전환
- 변경 전: `/treatments/%EC%98%A8%EB%8B%A4`
- 변경 후: `/treatments/onda`

## 용어 주의
- 지방파괴주사 → adipocytolysis injection (NOT fat dissolving)
- 지방분해주사 → fat dissolving injection
- 이 구분을 slug 생성 시 반드시 준수

## Phase 1 — DB 준비
1. `standard_treatments` 테이블에 `slug TEXT UNIQUE` 컬럼 추가
2. `name_en` 기반 slug 자동 생성 (소문자, 공백→하이픈, 괄호→하이픈, 특수문자 제거)
3. 전수 검수 (충돌, 오역, 참의원 관련 시술 용어 확인)

### slug 예시
| name_ko | name_en | slug |
|---------|---------|------|
| 온다 | Onda | onda |
| 보톡스 | Botox | botox |
| 지방파괴주사(바디) | Adipocytolysis Injection (Body) | adipocytolysis-injection-body |
| 지방파괴주사(얼굴) | Adipocytolysis Injection (Face) | adipocytolysis-injection-face |
| 지방분해주사 | Fat Dissolving Injection | fat-dissolving-injection |
| 울쎄라 | Ulthera | ulthera |

## Phase 2 — 라우팅 변경
1. `src/app/[locale]/treatments/[slug]/page.tsx` 조회 로직: slug 컬럼 우선, name_ko fallback
2. wiki(백과사전)는 이미 영문 slug 사용 중 — 변경 불필요

## Phase 3 — 내부 링크 교체
대상 파일 (사전조사 필요):
- 메인 페이지 Popular Treatments (PT_ORDER → slug 매핑)
- 클리닉 상세 시술 목록
- 고민 상세 관련 시술
- 검색 결과
- 사이트맵 생성
- PickButton 관련 링크
- 조사 명령: `grep -rn "encodeURIComponent.*name_ko\|treatments/" src/app/ --include="*.tsx"`

## Phase 4 — 301 리다이렉트
- 기존 한글 URL → 새 slug URL 301 리다이렉트
- Next.js middleware에서 name_ko → slug 매핑 조회
- Google/Naver에 이미 인덱싱된 URL 보호

## Phase 5 — SEO 대응
- 사이트맵 재생성 (slug 기반 URL)
- Google Search Console 새 사이트맵 제출
- Naver Search Advisor 새 사이트맵 제출

## Phase 6 — 빌드·배포·검증
- npm run build
- git push → Vercel 배포
- 확인: /en/treatments/onda, /ko/treatments/onda 정상 표시
- 확인: /en/treatments/온다 → /en/treatments/onda 리다이렉트
- 확인: 메인, 클리닉 상세, 사이트맵 링크
- 보고서 업데이트 (changelog v2.11)

## 예상 소요
| Phase | 소요 |
|-------|------|
| 1. DB 준비 | 50분 |
| 2. 라우팅 변경 | 15분 |
| 3. 내부 링크 교체 | 40분 |
| 4. 리다이렉트 | 20분 |
| 5. SEO 대응 | 10분 |
| 6. 빌드·배포·검증 | 15분 |
| **합계** | **약 2.5시간** |

## 장점
- SEO: 영문 키워드 URL → Google 영문 검색 노출 향상
- GEO: AI 모델이 URL에서 시술명 인식 가능
- 공유: SNS/메신저에서 가독성 높은 URL
- 다국어: /ja/treatments/onda 등 통일된 URL 체계

## 단점/리스크
- 기존 인덱싱 URL 리다이렉트 전환기간 (2~4주)
- slug 충돌 가능성 → 수동 검수로 해결
- 한국어 URL 직관성 상실 → 타겟이 외국인이므로 영문 slug가 적합
