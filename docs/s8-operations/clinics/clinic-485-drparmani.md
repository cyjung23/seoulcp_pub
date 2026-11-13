# clinic 485 — 닥터파마니의원 (Dr.PARMANI CLINIC)

- **등록 상태**: 운영 중 (2023년 메이린청담에서 명칭 변경, 2026-11-03 scp 반영)
- **홈페이지**: https://drparmani.com (영문 https://drparmani.com/en/)
- **최초 등록**: 메이린청담 시절부터 등록 (일자 미상)
- **최근 변경**: 2026-11-03

---

## 이력

### 2026-11-03 — 메이린청담 → 닥터파마니의원 전환 (DATA-011 / CASE-004)
- 요청자: GeoWise (marketing@geowise.co.kr)
- 접근 방식: 접근 B (clinic_id 485 유지, 전면 교체)
- 작업: name_ko/en 변경, website drparmani.com, operating_hours·doctors·description·specialties 전면 교체. maylincheongdam.com 리다이렉트 없음, "메이린" 흔적 전부 제거
- 네트워크: 메이린 네트워크 탈퇴, 타 지점(1170·1995·875)과 연결 없음 확인
- 상세: [v2.36 changelog](../../s2-data/s2-06-changelog.md)

### 2026-11-03 — clinic_treatments 전면 교체 3→72건 (DATA-012)
- 작업: 기존 3건(18355·18356·18357) DELETE → 72건 INSERT (id 39123~39194). 3개 프로그램(Revitalization/Skin Revitalization/Signature Membership) 반영
- standard_treatments: 신규 카테고리 "피부재생" 신설 (결정 J)

### 2026-11-03 — clinic_devices 56건 등록 (DATA-013)
- 작업: 기존 devices 재사용 17건(Genesis id 164→169 중복 정리) + 신규 INSERT 39건
- devices 신규 카테고리 3개: 항노화 진단·항노화 치료·피부재생 (결정 U-OK)
- manufacturer 가능 범위 내 채움 (결정 S-b, T-a)
