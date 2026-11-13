# INCIDENT-002. 중국 AS 봇 재공격 + Supabase Unhealthy (2026-11)

- **티켓**: OPS-002
- **발생 기간**: 2026-11-10 ~ 2026-11-12
- **상태**: 조치 완료, 1시간 내 완전 회복 확인
- **영향**: Supabase STATUS Unhealthy, API Gateway Success Rate 98.3% → 15.2% 폭락

## 증상
- 2026-11-10: Supabase "Disk IO Budget 소진 임박" 경고 메일 재발 (PERF-001 적용 5일 후)
- 2026-11-12: Supabase 대시보드 STATUS Unhealthy, CPU/Disk/RAM 0% 표시 + "Conns Unavailable"
- Total Requests 7,329건 / 24h (평소 24,724건의 1/3), Success Rate 15.2%
- API Gateway 7,177건 중 ERRORS 6,073건 (85% 실패)
- Postgres 150건 중 ERRORS 131건, Realtime 2건, Edge Functions·Auth·Storage는 0
- OPS-001(10/30)과 동일 패턴: Postgres/Edge Functions 자체는 정상, API Gateway 레벨에서 전부 실패

## 원인
- Vercel Firewall 로그 분석 결과 중국계 클라우드 3사가 공격 주체:
  - **UCLOUD INFORMATION TECHNOLOGY (AS 135377)**: 774건, Denied 1위 IP 152.32.226.189
  - **CHINA UNICOM China169 Backbone (AS 4837)**: 619건, IP 20.92.243.53 등
  - **China Telecom (AS 4134)**: 467건
- 기타 Baiduspider 408건, 추가 IP(178.238.230.225, 23.251.135.142 등) 활동
- Vercel Observability: Vercel Functions **Timeout 9%** 확인 (정상 1~2%)
- Firewall이 Challenge/Deny 3.5k+1.3k 처리했으나 통과한 봇이 여전히 Supabase 과부하 유발
- PERF-001(ISR 60→3600)로 쿼리는 줄였으나, 공격량이 커서 근본 해결 안 됨

## 조치
### Vercel Firewall (AS 단위 Challenge 규칙 3개 신설)
- Rule "Challenge UCLOUD HK traffic": AS 135377 → Challenge
- Rule "Challenge AS 4837 traffic": China Unicom → Challenge
- Rule "Challenge China Telecom traffic": AS 4134 → Challenge
- **Challenge 선택 근거**: 중국어 페이지 유지(외국인 환자 대상) 정책과 양립, 실사용자는 JS 챌린지 통과 가능, 봇만 차단

### 다층 방어 구조 최종 확보
1. 중국 AS 3개 → Challenge (본 조치)
2. Bot Protection → Challenge (OPS-001)
3. AI Bots → Block (OPS-001)
4. DDoS Mitigation → Vercel 자동 관리

### Supabase
- Restart 없이 자체 회복 (Firewall 조치로 공격 트래픽이 끊기면서 Healthy 전환)

## 복구 확인
- 조치 직후 Last 60 minutes: Total Requests 664 / Success Rate **100.0%**, 모든 서비스 ERRORS 0
- 조치 후 1시간 내 Success Rate 15.2% → 100% 완전 회복
- 24시간 지표는 조치 이전 누적치 포함으로 23.7% (실시간은 정상)

## 보류·후속 과제
- 중국 AS 추가 공격 발생 시 Challenge 대상 AS 확장 (현재 3개)
- OPS-003 (Vercel Pro 전환) 유발 — 본 사건의 간접 후속
- FIX-004 (ja/zh name_en 폴백) 미해결 상태 유지

## 관련 변경
- 코드 변경 없음 (Vercel Firewall 콘솔 설정만)
- 관련: OPS-001 (incident-001, 10월 1차 봇 공격), PERF-001 (ISR 상향, 2026-11-05)

## 교훈
- **IP 단위보다 AS 단위 차단이 압도적으로 효과적** (IP는 계속 바뀌지만 AS는 고정)
- Challenge는 Deny보다 유연 — 실사용자 접근성 유지하면서 봇 차단 가능 (다국어 페이지 운영 정책과 양립)
- 봇 공격은 월 1회 이상 재발 패턴 — PERF-001 같은 쿼리 최적화만으로는 부족, Firewall 레벨 차단 필수
- Vercel Observability의 Functions Timeout % 지표가 봇 공격 조기 탐지에 유용 (9% 넘으면 공격 의심)
