# INCIDENT-001. Vercel 리소스 초과 + Supabase Unhealthy + 봇 어뷰즈 (2026-10)

- **티켓**: OPS-001
- **발생 기간**: 2026-10-20 ~ 2026-10-30
- **상태**: 조치 완료, 모니터링 중
- **영향**: seoulcp.com 간헐 접속 불가, Supabase API Gateway 611건 에러 (Success Rate 0.2%)

## 증상
- 10/26: 사이트 접속 불가 ("seoulcp.com에서 응답하는 데 시간이 너무 오래 걸립니다")
- Vercel Hobby Plan Fluid Active CPU 8h 12m / 4h 한도 (205% 초과)
- Fluid Provisioned Memory 454.0 GB-Hrs / 360 GB-Hrs (126% 초과)
- Supabase Dashboard: 612 Total Requests, Success Rate 0.2%, API Gateway Errors 611건
- Postgres/Edge Functions/Auth 자체는 에러 0 (API Gateway 레벨 전부 실패)

## 원인
- 일 20~30명 실제 사용자 대비 **봇 트래픽 수백 배**
- 주 가해 봇:
  - Bytespider (ByteDance, 10/26 20:57~21:06 9분간 초당 1건, 시간당 240건·일 5,700건)
  - 중국 Urumqi 봇 (10/28, 활성 "사용자" 115명 중 98명이 중국 Urumqi, 평균 참여 2초, direct 유입)
  - IP 34.65.175.217 (554건), 35.240.161.218 (277건)
- middleware.ts의 BLOCKED_BOTS에 Bytespider 포함되어 있었으나 대소문자/매칭 오류로 대부분 통과
- ISR revalidate=60 (v2.34, 10/10 적용) 캐시 miss 반복이 비용 폭증 가속

## 조치
### Vercel Firewall
- Bot Protection On, **AI Bots 토글 On** 활성화
- Challenge 모드 설정 (Log 기록 + Verified bots 예외)
- 효과: Allowed 4.7k·Denied 900·Challenged 124건 수준으로 정상화

### Supabase
- Project Restart 실행 (1~2분 소요)
- STATUS Unhealthy → Healthy 복구 확인 (CPU 0%, Disk 5%, RAM 45%, 7/60 conns)

### 모니터링
- Vercel Firewall Overview 1~2시간 주기 확인 권장
- Disk IO Budget 경고(10/26) 소진 시 5 MB/s 저하 리스크 존재

## 보류·후속 과제
- middleware.ts BLOCKED_BOTS 재점검 (대소문자·매칭 패턴) — 별도 티켓 후보
- ISR revalidate 60→3600 상향 검토 (PERF-001 후보)
- Node.js 20 deprecated 경고 (actions/checkout@v4, supabase/setup-cli@v1 → Node 24 forced) — 별도 티켓 후보
- Supabase Advisor Critical 10건 (RLS references user_metadata) — 별건

## 관련 변경
- 코드 변경 없음 (Vercel/Supabase 콘솔 설정만)
- 관련 결정: DEC-056 (seoulclinicpick.com 전환) Phase 1 미착수 상태로 유지 (본 사건 대응으로 밀림)

## 교훈
- Hobby Plan + 봇 어뷰즈 조합은 사용자 트래픽이 적어도 리소스 폭증 가능
- Vercel Bot Protection은 default Off이므로 신규 프로젝트 세팅 시 활성화 필수
- middleware 레벨 봇 차단은 실제 매칭 로그로 검증 필요 (코드 포함만으로는 신뢰 불가)
