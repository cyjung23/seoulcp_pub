# S4-04: 보류 작업

**최종 갱신:** 2026-04-11

## WO-019 — Naver body 인덱싱 해결
- 보류 사유: Next.js App Router RSC 구조적 한계. ISR 테스트 실패 확인.
- 대안: (A) prerender.io, (B) Pages Router 부분 전환, (C) 수동 제출 결과 후 재판단
- 다음 액션: MON-001 결과 (4/20 이후) 확인 후 결정

## WO-017-v2 — 아이콘 디자인 교체 재진행
- 보류 사유: WO-017 revert 후 대안 미정. 전체 일괄 교체 필요 (DEC-037).
- 선택지: Lucide 재시도, 커스텀 SVG, 현행 이모티콘 유지

## WO-018 — unmatched 시술 재매칭
- 보류 사유: 511건 (리프팅 415, 콧대 69, 비중격 27). standard_treatments 확장 필요.
- 효과: 커버리지 71.0% -> 75%
- 참조: match_results_v3.csv

## WO-020 — raw_texts mojibake 수정
- 보류 사유: 우선순위 낮음. 672개 파일 EUC-KR/CP949 -> UTF-8 변환 오류.
- 효과: clinic_devices 1~2%p 향상 가능

## CONTENT-001 — 백과사전 본문 품질 강화
- 보류 사유: 원장님 판단에 따라 간단 설명 수준 유지. 사용자 동선 개선에 집중.
