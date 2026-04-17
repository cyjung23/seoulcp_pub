# S4-09: WO-028 일본어 사이트 확장

**최종 갱신:** 2026-04-17
**상태:** 계획 수립 완료, 착수 대기

## 목표

seoulcp.com에 일본어(/ja/) 페이지를 추가하여 일본인 대상 서비스 확장

## 의사결정 완료 사항 (DEC-055)

| 항목 | 결정 | 비고 |
|------|------|------|
| 주소 표시 | 영문 주소 사용 | address_en 공유, address_ja 별도 불필요 |
| 언어 전환 UI | ko/en/ja 전환 추가 | 향후 중국어(zh) 추가를 고려한 확장형 UI |
| 시술명 표시 | 영어(메인) + 카타카나(보조) | 예: Botox（ボトックス） |
| 시술 설명 | 일본어 | 상세 설명은 일본어로 작성 |
| UI/섹션 제목 | 일본어 | 人気の施術、お悩みから探す 등 |
| 버튼/링크 텍스트 | 일본어 | 比較する、もっと見る 등 |
| 메타데이터 | 일본어 | title, description 일본어 |
| 클리닉 주소 | 영문 | 기존 address_en 활용 |
| 도메인 전략 | /ja/ 경로 유지 | 서브도메인 사용 안 함 |

## 표시 언어 정리

| 항목 | 언어 | 예시 |
|------|------|------|
| UI/섹션 제목 | 일본어 | 人気の施術、お悩みから探す |
| 시술명 (메인) | 영어 | Botox, Thread Lifting |
| 시술명 (보조) | 카타카나 | ボトックス、糸リフト |
| 시술 설명 | 일본어 | 精密な微細注射でシワを減らし… |
| 클리닉 주소 | 영문 | 영문 주소 그대로 사용 |
| 메타데이터 | 일본어 | title, description 일본어 |
| 버튼/링크 텍스트 | 일본어 | 比較する、もっと見る |

## Phase 구성

### Phase 1 — i18n 인프라 확장 (30분)

- routing.ts: locales에 ja 추가
- messages/ja.json 생성 (nav, common, home 일본어 번역)
- middleware.ts: matcher에 ja 추가
- layout.tsx: locale 타입에 ja 추가
- next.config.ts: redirects에 /ja/ 경로 추가

### Phase 2 — DB 일본어 컬럼 추가 (1시간)

- standard_treatments: name_ja TEXT 추가 (카타카나 병기용)
- concerns: concern_group_ja TEXT 추가
- devices: category_ja TEXT 추가
- encyclopedia: title_ja TEXT, content_ja TEXT 추가 (향후)
- clinics 주소는 address_en 공유 → 별도 컬럼 불필요

### Phase 3 — 페이지 로케일 로직 변경 (2시간)

현재 isEn ? A : B 패턴을 다국어 헬퍼 함수로 전환:

    function locText(locale: string, ko: string, en: string, ja?: string) {
      if (locale === "ja") return ja || en;
      if (locale === "en") return en;
      return ko;
    }

대상 파일 (약 15개):
- page.tsx (메인)
- treatments/page.tsx, treatments/[slug]/page.tsx
- clinics/page.tsx, clinics/[id]/page.tsx
- concerns/page.tsx, concerns/[slug]/page.tsx
- surgeries/page.tsx
- search/page.tsx
- wiki/page.tsx, wiki/[slug]/page.tsx
- DeviceRelatedTreatments.tsx
- beauty 관련 페이지들
- 공통 컴포넌트 (AuthNav, PickButton 등)

### Phase 4 — 일본어 번역 데이터 입력 (2~3시간)

- 시술명 카타카나 120개 (name_ja)
- 고민 카테고리 일본어 번역
- 메인 페이지 UI 텍스트 (Popular Treatments 설명 등)
- 메타데이터 (title, description)
- 언어 전환 UI 컴포넌트 (ko/en/ja, 향후 zh 확장 고려)

### Phase 5 — SEO 대응 (30분)

- sitemap.ts에 /ja/ URL 추가
- hreflang 태그에 ja 추가
- alternates에 ja 경로 포함
- GSC/Naver에 sitemap 재제출

### Phase 6 — 빌드·배포·검증 (30분)

- npm run build
- /ja/, /ja/treatments/botox, /ja/clinics/1 등 검증
- 언어 전환 UI 동작 확인 (ko - en - ja)
- 모바일 반응형 확인

## 예상 소요

| Phase | 내용 | 소요 |
|-------|------|------|
| 1 | i18n 인프라 확장 | 30분 |
| 2 | DB 일본어 컬럼 추가 | 1시간 |
| 3 | 페이지 로케일 로직 변경 | 2시간 |
| 4 | 일본어 번역 데이터 입력 | 2~3시간 |
| 5 | SEO 대응 | 30분 |
| 6 | 빌드·배포·검증 | 30분 |
| **합계** | | **약 6.5~7.5시간** |

## 선행 조건

- WO-027 slug 전환 완료 → /ja/treatments/botox 형태로 통일
- WO-025 뷰티회원제 완료 → 일본어 UI 적용 대상에 포함
- WO-026 메인 디자인 완료 → 일본어 메인 페이지에 동일 디자인 적용

## 향후 확장

- 중국어(zh) 추가 시 동일 패턴 적용 (routing, messages, DB 컬럼, 헬퍼 함수)
- 언어 전환 UI는 처음부터 확장형으로 설계
