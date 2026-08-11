# 병원 개별 변경 요청 처리 매뉴얼

병원측 자발적 요청에 대응하는 표준 절차. 방향 C(스키마 미확장, 개별 대응) 원칙.

## 저장소 역할

- `/workspaces/seoulcp` — Next.js 앱 코드
- `/workspaces/seoulcp_pub` — 문서 (docs/)

## 요청 유형별 처리 흐름

### 1) name_en / phone / description 정정
1. Supabase SQL Editor에서 UPDATE clinics SET ... WHERE id = XXXX; 실행
2. name_en 변경 시 name_en_source='clinic_request', name_en_corrected_at=NOW() 함께 갱신
3. curl로 페이지 반영 검증
4. 대장 기록 → 커밋 → 회신

### 2) 다국어 페이지 타이틀 (EN/JA/ZH)
파일: /workspaces/seoulcp/src/app/[locale]/clinics/[id]/page.tsx

파일 상단 상수 CLINIC_TITLE_OVERRIDES에 병원 항목 추가.
형식 예시:
    const CLINIC_TITLE_OVERRIDES: Record<string, Record<string, string>> = {
      "XXXX": {
        en: "...",
        ja: "...",
      },
    };

generateMetadata 내부 로직은 이미 overrideTitle || clinic.description || (기본 템플릿) 순서로 동작하므로 상수 추가만 하면 됨.

### 3) 표준 마스터 미매핑 clinic_treatments
DATA-004 (3)번으로 이관. 개별 처리 불가.

## DB 정정 시 필수 컬럼

clinics 테이블 정정 시 반드시 함께 갱신:
- name_en_source — 'auto' | 'clinic_request' | 'admin'
- name_en_corrected_at — TIMESTAMPTZ (KST 기준)

## 대장 기록 규칙

파일: docs/s2-data/clinic-change-requests.md

한 요청 = 한 행. 컬럼: 처리일 | 병원ID | 요청 내용 | 처리 결과 | 참조(commit SHA 또는 백로그 ID).

보류 항목은 "보류 (사유, 백로그 ID)"로 기록하고, 나중에 처리되면 sed로 갱신.

## 커밋 메시지 컨벤션

- 코드 변경 (seoulcp): feat(clinic-XXXX): 요약 (병원명 요청 대응)
- 문서 변경 (seoulcp_pub): docs(clinic-XXXX): 요약
- 정정 완료 표기: docs(clinic-XXXX): mark 요청 as 적용 완료 (seoulcp SHA)

## 임계값 (인라인 override 방식)

- 1~4개 병원: 현행 인라인 상수 유지
- 5개 이상: data/clinic-i18n-overrides.json 별도 파일로 전환 검토
- 10개 이상: DATA-004 스키마 확장 (title_en, title_ja 등 컬럼) 재검토

## 참고

- 방향 C 채택 배경: s2-06-changelog.md v2.25
- 요약: s4-06-summary-table.md DATA-005
- 첫 사례: clinic 2280 엄나구모 (seoulcp commits 4775941, d6e52d3)
