# S3-05: WO-008 ~ WO-013 매핑 확장

**최종 갱신:** 2026-04-10

## WO-008 — raw_texts 키워드 추출 (clinic_devices 시도)
- 작업일: 2026-04-03. 상태: 완료 (제한적 성과).
- 177개 미매핑 클리닉 대상, 1건 성공
- 실패: mojibake, 일반 정보 위주, 미매핑 60%가 성형외과

## WO-010 — clinic_concerns 자동 확장 1차
- 기간: 2026-04-03 ~ 2026-04-04. 상태: 완료.
- treatment_concerns 기반 경로, concern_group 대표만, source: auto_treatment
- 결과: 약 16,986건, 1,336개 클리닉

## WO-011 — 미매핑 15개 시술 concern 연결 + 가슴수술 그룹 신설
- 기간: 2026-04-04 ~ 2026-04-05. 상태: 완료.
- 대상 15개 시술 (흉터재건, 기능코수술, 뒤트임, 돌출입수술 등)
- concerns 5건 신규: 가슴확대(182), 가슴축소(183), 가슴재수술(184), 여유증(185), 돌출입(186)
- treatment_concerns 20건, clinic_concerns +434건 (source: auto_treatment_v2)

## WO-012 — standard_treatment_id 경로 clinic_concerns 확장
- 작업일: 2026-04-05. 상태: 완료.
- clinic_treatments 67%(15,658건)가 treatment_id NULL
- standard_treatments.name_ko = treatments.name_ko 매칭 (104/111 성공)
- +4,095건 (source: auto_std_treatment_group), 총 21,515건, 1,915개(70.2%)

## WO-013 — clinic_devices 자동 확장 (3단계)
- 기간: 2026-04-05 ~ 2026-04-06. 상태: 완료.

### Stage 1 (treatment_devices 기반)
경로 A + B 결합. +5,680건. 5,012->10,692건, 1,081->1,544개(+463).

### Stage 2 (name 매칭 확장)
13개 신규 treatment_device. 27->40건. +221건. 10,913건.

### Stage 3 (영문명 장비 매핑)
Thermage->써마지, Ulthera->울쎄라, InMode->인모드, Potenza->포텐자. 40->44건. +1,605건. 12,518건.

최종: 5,012->12,518(+7,506), 1,081->1,544(+463), 39.6%->56.6%.
