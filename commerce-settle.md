# commerce-settle 기여 내용

[← 프로필로 돌아가기](./README.md) · [팀 저장소](https://github.com/500-internal-server-error-team/commerce-settle)

**주문·결제·환불과 판매자 정산을 잇는 커머스 MSA 플랫폼** (팀 프로젝트)

주문이 몰리거나 외부 PG 응답이 유실돼도 거래 상태와 금액이 어긋나지 않게 만드는 데 집중한 프로젝트입니다. Gateway · Commerce · Payment · Settlement · Finance Mock 서비스를 Kafka 이벤트로 연결하고, 업무 상태와 Outbox를 함께 저장하며 수신 측은 Inbox로 중복을 거릅니다.

## 내 역할

Commerce 서비스의 **상품 · 가격 · 재고 · 쿠폰 · 선착순 이벤트**를 맡았고, 후반에는 부하 시험에서 드러난 **DB 부하 개선과 결제 부하 측정**을 진행했습니다.

| 항목 | 값 |
| --- | --- |
| 기간 | 2026-10-28 ~ 2026-11-12 (develop 반영 기준) |
| develop 커밋 | 116개 |
| 병합된 PR | 23개 |
| 리뷰 중인 PR | #323 · #339 · #340 · #343 |

## 기술 스택

Java 21 · Spring Boot 3 · Spring Cloud Gateway · PostgreSQL · Kafka · Redis · Docker Compose · AWS ECS · Prometheus · Grafana · Zipkin · k6

## 영역별 기여

### 📦 상품 · 재고

- 상품 카탈로그와 재고 API, 요청 멱등 처리와 공통 오류 응답 ([#64](https://github.com/500-internal-server-error-team/commerce-settle/pull/64))
- 상품·재고 쓰기의 DB 잠금 대기 상한, 재고 행을 먼저 잠그는 순서 정리 ([#73](https://github.com/500-internal-server-error-team/commerce-settle/pull/73))
- 일반 주문의 재고 예약 · 확정 · 해제 · 복원과 수량 규칙 ([#79](https://github.com/500-internal-server-error-team/commerce-settle/pull/79), [#92](https://github.com/500-internal-server-error-team/commerce-settle/pull/92))
- 반품 입고 검수 결과로 재판매할 수 없는 상품을 가용 재고에서 제외 ([#210](https://github.com/500-internal-server-error-team/commerce-settle/pull/210))

### ⏱ 선착순 이벤트

- 판매자의 이벤트 등록 · 공개 · 종료, 판매 진행률 게이지, 참여 예약과 주문 연결 ([#112](https://github.com/500-internal-server-error-team/commerce-settle/pull/112))
- 매진 뒤 요청이 이벤트 행 잠금을 기다리지 않고 바로 거절되도록 고정 ([#185](https://github.com/500-internal-server-error-team/commerce-settle/pull/185))
- 게이지 조회에 1초 인스턴스 캐시 ([#200](https://github.com/500-internal-server-error-team/commerce-settle/pull/200))
- 재시도 · 공개 · 거절 사유 · 시작 처리의 정확성 보완, 유일 제약 경합의 409 분류 ([#210](https://github.com/500-internal-server-error-team/commerce-settle/pull/210))
- 매진 이벤트 주문을 저장 전에 거절, 선착순 집중 부하 도구와 반품 왕복 smoke ([#229](https://github.com/500-internal-server-error-team/commerce-settle/pull/229))
- 신규 이벤트 주문에 이벤트별 동시 실행 상한 ([#273](https://github.com/500-internal-server-error-team/commerce-settle/pull/273))

### 🎟 쿠폰 · 혜택

- 쿠폰 기반과 웰컴 쿠폰, 주문의 쿠폰 예약 · 사용 · 복구, 만료 스케줄러 ([#167](https://github.com/500-internal-server-error-team/commerce-settle/pull/167))
- 판매자 쿠폰 캠페인 등록 · 발급과 예산 ([#175](https://github.com/500-internal-server-error-team/commerce-settle/pull/175))
- 월간 우수고객 쿠폰 대상 선정 · 발급 Job ([#176](https://github.com/500-internal-server-error-team/commerce-settle/pull/176)), 회차 운영 조회 · 교정 API와 잔류 실행 복구 ([#179](https://github.com/500-internal-server-error-team/commerce-settle/pull/179))
- 구매확정 후 사용할 수 있는 구매 마일리지 적립 — Commerce · Payment 연동 ([#276](https://github.com/500-internal-server-error-team/commerce-settle/pull/276))

### ⚡ 성능 개선 — 부하 시험 SQL 줄이기

300 TPS 선착순 부하 시험에서 DB가 버거워한 원인을 `pg_stat_statements`로 분석하고, 트랜잭션마다 반복되던 SQL과 반복 전수 검사를 줄였습니다.

- 트랜잭션 대기 상한 설정을 물리 트랜잭션당 한 번으로 — Finance Mock ([#317](https://github.com/500-internal-server-error-team/commerce-settle/pull/317))
- Settlement 이벤트 반영 경로 · 업무 지표의 SQL 감소, JDBC batch ([#320](https://github.com/500-internal-server-error-team/commerce-settle/pull/320))
- Settlement 정합성 점검의 반복 전체 검사와 이벤트별 SQL 감소 ([#321](https://github.com/500-internal-server-error-team/commerce-settle/pull/321))
- Payment 결제 처리 · Outbox · 관측 지표의 SQL 감소 ([#322](https://github.com/500-internal-server-error-team/commerce-settle/pull/322))
- Commerce 주문 경로 · 결제 접수 Inbox · Outbox의 SQL 감소 ([#323](https://github.com/500-internal-server-error-team/commerce-settle/pull/323), 리뷰 중)

> 5개 PR을 합친 버전을 로컬 PC 한 대에서 1회 비교한 결과, 50 TPS에서 주문 응답 p95가 246ms → 32ms, Commerce SQL 총 실행 시간이 85% 줄었습니다. 같은 환경의 단일 측정이며 #323은 아직 병합 전입니다.

### 📈 부하 측정 · 관측

- 결제 승인 동시성 6 / 12 / 24에서의 처리량과 상한 대비 차이 측정 ([#252](https://github.com/500-internal-server-error-team/commerce-settle/issues/252))
- 장애 중 지연이 결과에서 빠지던 부하 요약을 Kafka 발행 시각 · drain 포함 기준으로 보완 ([#340](https://github.com/500-internal-server-error-team/commerce-settle/pull/340), 리뷰 중)
- Worker 강제 종료 뒤 Kafka 수신이 45초 멈추는 원인(consumer 세션 만료)과 설정별 비교 ([#253](https://github.com/500-internal-server-error-team/commerce-settle/issues/253))
- 결제 재시도 회차의 Commerce · Payment 왕복 검증과 취소 재요청 버그 수정 ([#343](https://github.com/500-internal-server-error-team/commerce-settle/pull/343), 리뷰 중)

### 🔧 기타

- Payment가 commerce 토픽의 미처리 eventType을 격리 없이 ACK ([#256](https://github.com/500-internal-server-error-team/commerce-settle/pull/256))

## 기록

과정과 측정 결과는 [velog](https://velog.io/@sarden030)에 정리했습니다.
