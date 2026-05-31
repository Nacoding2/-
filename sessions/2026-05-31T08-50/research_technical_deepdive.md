# 📚 Usage-Based Billing 기술 심층 분석 보고서 (2026.05.31)

## 🎯 분석 목표
Usage-Based Billing 모델의 기술적 아키텍처를 검증하고, MVP 구축을 위한 핵심 API 엔드포인트 및 데이터 스키마의 기술적 근거를 확보함.

## 📈 핵심 기술 트렌드 요약
1. **실시간 이벤트 기반 측정 (Event-Driven Metering):** 사용량 측정을 주기적 배치 작업이 아닌, 사용자 액션 발생 즉시 처리하는 스트리밍 아키텍처가 필수적임. (Kafka/Kinesis 기반)
2. **BaaS 활용 (Billing-as-a-Service):** 복잡한 결제 로직(세금, 환율, 구독)은 Stripe 등 전문 솔루션에 위임하고, 우리 서비스는 '사용량 측정 로직'에 집중해야 함.
3. **소유권 중심 결제:** 단순 돈 지불을 넘어, 토큰/크레딧을 통해 접근 권한을 증명하는 방식(Token Gating)이 시장 표준화 추세임.

## 🛠️ 필수 API 엔드포인트 및 스키마 요구사항 (Task Input)

| 엔드포인트 | HTTP Method | 목적 | 요구되는 데이터 (Schema Key) | 기술적 난이도 |
| :--- | :--- | :--- | :--- | :--- |
| `/api/v1/usage/record` | `POST` | **[핵심] 사용량 측정 기록 (Event Logging)**: 사용자가 기능을 사용할 때마다 이 엔드포인트로 사용 이벤트(Usage Event)를 비동기적으로 전송해야 함. | `user_id`, `usage_type` (ex: image_gen), `unit_cost` (단가), `quantity` (개수), `timestamp` | Medium (스트리밍 처리 필요) |
| `/api/v1/user/token_balance` | `GET` | **[핵심] 잔여 토큰 확인:** 사용자가 결제 전 자신의 크레딧/토큰 잔액을 실시간으로 조회하는 기능. | `user_id`, `current_balance` (int), `currency` | Low (DB 조회) |
| `/api/v1/billing/finalize` | `POST` | **[결제 확정] 청구서 발행 요청:** 특정 기간 또는 사용량 기록을 모아 최종 청구서 발행을 요청. (BaaS 연동 지점) | `user_id`, `billing_cycle_start`, `billing_cycle_end`, `usage_summary` (총 사용량 맵) | High (트랜잭션 관리, 외부 API 호출) |

## 🚀 결론
MVP 아키텍처는 **'사용량 이벤트 수집 및 전송' $\rightarrow$ '사용량 측정 및 토큰 차감' $\rightarrow$ 'BaaS를 통한 최종 청구서 발행'**의 3단계 파이프라인으로 설계되어야 하며, 개발 에이전트는 이 3단계의 시스템 간 연동(Integration)에 초점을 맞춰야 함.