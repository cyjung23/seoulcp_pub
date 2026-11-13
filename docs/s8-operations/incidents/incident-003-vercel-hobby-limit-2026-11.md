# INCIDENT-003. Vercel Hobby Fluid Memory 한도 초과 → Deployments Paused (2026-11)

- **티켓**: OPS-003
- **발생일**: 2026-11-13
- **상태**: Pro 업그레이드로 즉시 복구, Spend Management 하드캡 적용
- **영향**: seoulcp.com 전면 접속 불가 ("This deployment is temporarily paused")

## 증상
- 2026-11-13 오전: seoulcp.com 접속 시 "This deployment is temporarily paused" 페이지 노출
- Vercel 대시보드 좌측 하단 경고:
  > "Deployments Paused since Nov 13, 2026 for Fluid Provisioned Memory. Your team exceeded the Hobby fair use limits. This pause ends on Dec 13, 2026."
- Supabase는 Healthy, 실시간 요청도 정상 (문제는 Vercel 단독)

## 원인
- **OPS-002 봇 공격의 간접 후속**
- 2026-11-10~12 중국 AS 봇 재공격 기간 Vercel Functions 호출량 폭증 (Supabase API Gateway 24시간 58,518건, 평소 7,329건의 8배)
- 봇 차단 조치(Firewall AS Challenge)로 공격은 멈췄으나 이미 소진된 Fluid Provisioned Memory가 Hobby 월간 한도 초과를 트리거
- Vercel 정책상 한도 초과 시 다음 결제 주기(12-13)까지 자동 Pause

## 조치
### Vercel Pro 업그레이드 (즉시 복구)
- Hobby → Pro ($20/월, 팀당)
- 결제 주기: 2026-11-12 ~ 2026-12-12
- 업그레이드 즉시 Deployments Paused 해제, seoulcp.com 2~3분 내 복구
- Included Credit $20 제공, 초과분은 종량제

### Spend Management (과금 안전장치)
- Budget: **$20** (Included Credit 한도와 동일)
- **Pause: On** (한도 도달 시 자동 정지, 추가 과금 100% 차단)
- Webhook: Off
- Alert thresholds: 자동 설정 50%/75%/100% ($10/$15/$20)
- 수신 이메일: Billing Email

### 다운그레이드 계획
- 2026-12-10경 Pro → Hobby 다운그레이드 예정 (Vercel Settings → Billing → Downgrade)
- 사유: 평시 트래픽 20~30명/일 수준은 Hobby 한도로 충분
- 미리 다운그레이드해도 결제 주기 끝까지 Pro 유지, 다음 주기부터 Hobby 전환 (일할 계산 없음)

## 보류·후속 과제
- **봇 공격 재발 시 $20 하드캡 소진 → 재Paused 가능성** (현재 Firewall 다층 방어로 리스크 저감)
- 다운그레이드 후 Hobby 한도 재초과 패턴이 반복되면 Pro 유지 구조적 필요성 재평가
- Observability Plus, Speed Insights, Web Analytics 활성화는 당분간 보류 (사용량 과금 가능성)

## 관련 변경
- 코드 변경 없음 (Vercel 플랜 전환 + Billing 설정만)
- 관련: OPS-002 (incident-002, 본 사건의 직접 원인), OPS-001 (incident-001, 1차 봇 공격)

## 교훈
- Hobby Plan Fluid Memory 한도는 **일시적 공격 한 번으로도 월 전체를 날릴 수 있음** (한도 리셋이 다음 결제 주기)
- Pro 업그레이드 시 **Spend Management 하드캡 + Pause On은 필수** — 안 하면 종량제 과금 폭탄 가능
- 봇 공격 사후 조치만으로는 Vercel 한도 문제 해결 안 됨 → 사전 Firewall 차단이 본질적 해결책 (OPS-002 교훈과 연계)
- 다운그레이드는 결제 주기 종료 직전(D-3 정도)에 미리 설정해두는 것이 안전
