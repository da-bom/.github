# 실시간 가족 데이터 통합 관리 시스템 - 용어집

> **문서 버전**: v25.0
> **작성일**: 2026-03-20
> **작성자**: DABOM 팀
> **변경 이력**: v25.0 - 전체 문서 v25.0 Major 버전 동기화: Outbox Pattern·직접 DB 정산 용어 추가, usage-persist·Self-Consumption Pattern 제거, notification 14종 타입 확정, PUSH·VAPID 용어 추가, Event Envelope subType 참조 제거 | v22.0 - API_SPECIFICATION v22.2 Major 버전 동기화: UPLOADS 도메인 용어 추가, R2 약어 추가 | v21.0 - API_SPECIFICATION v21.6 동기화: 협상→이의제기(Appeal), 리포트→리캡(Recap) 용어 전면 갱신, 소통 점수(Communication Score) 반영 | v11.0 - 2차기획서 Phase 2 기능 반영: 이의제기/미션/보상/리캡 용어 및 API 도메인 추가 | v10.2 - ERD v10.2 동기화: POLICY 테이블 is_activate → is_active 리네이밍 | v10.0 - web-core 서브도메인 분리: web-service (www.dabom.site) + web-admin (admin.dabom.site) 용어 반영 | v9.0 - api-spec 최종 동기화: 도메인 구조 변경 (7→5도메인), 용어 업데이트 | v8.1 - ERD v8.1 동기화: 정책 활성화(Is Activate) 용어 추가 | v8.0 - 전체 문서 버전 통일 (공유 Major + 독립 Minor 체계 도입) | v7.0 - simulator-traffic → simulator-usage 리네이밍 동기화 | v6.0 - ERD v6.0 동기화: FAMILY_GROUP→FAMILY 리네이밍, ALREADY_BLOCKED 제거 | v5.0 - ERD v5.0 동기화: daily→monthly 전환, CUSTOMER/ADMIN 분리 반영, 신규 용어 추가 | v4.0 - API 도메인 그룹핑 반영, REST 알림 API 추가, 도메인 용어 추가

---

## 1. 도메인 용어

### 사용자 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 가족 그룹 | Family Group | 데이터를 공유하는 사용자들의 집합. 최대 10명까지 구성 가능 |
| 가족 구성원 | Family Member | 가족 그룹에 속한 일반 사용자. 데이터 조회만 가능 |
| Owner 계정 | Owner | 가족 그룹 내 정책 수정 권한을 가진 관리 사용자. 복수 Owner 가능 (`family_member.role='OWNER'` 기준) |
| 운영자 | Backoffice Admin | 시스템 전체를 관리하는 내부 관리자 |

### 데이터 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 할당량 | Quota | 사용자 또는 그룹에 할당된 데이터 한도 (바이트 단위) |
| 잔여량 | Remaining | 사용 가능한 남은 데이터량 |
| 사용량 | Usage | 실제로 사용한 데이터량 |
| 월별 한도 | Monthly Limit | 한 달 동안 사용할 수 있는 최대 데이터량 |
| 임계치 | Threshold | 알림을 발송하는 기준점 (50%, 30%, 10%) |
| 가족 할당량 | Family Quota | 가족의 월별 데이터 총량 스냅샷 엔티티. family에서 분리된 family_quota 테이블로 독립 관리 |

### 정책 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 정책 | Policy | 데이터 사용에 적용되는 규칙 (한도, 시간대 차단 등) |
| 시간대 차단 | Time Block | 특정 시간대(예: 22:00~07:00)에 데이터 사용을 차단하는 정책 |
| 즉시 차단 | Manual Block | Owner가 특정 구성원의 데이터 사용을 즉시 차단하는 기능 |
| 앱별 차단 | App Block | 특정 앱/서비스의 데이터 사용을 차단하는 정책 (MVP 제외) |
| 선착순 완전 승인 | First-Come-First-Served | 동시 요청 시 먼저 도착한 요청만 전체 승인, 나머지는 즉시 차단하는 정책 |
| 기본 규칙 | Default Rules | 정책 템플릿에 정의된 기본 규칙 JSON (`default_rules`). 적용 시 POLICY_ASSIGNMENT.rules로 복사 |
| 최소 역할 | Require Role | 정책 적용을 받을 수 있는 최소 역할 (`require_role`). MEMBER 또는 OWNER |
| 정책 활성화 | Is Active | 정책 템플릿의 활성/비활성 상태 (DB: `is_active`, API JSON: `isActive`). FALSE이면 신규 적용 불가 |

### 이의제기 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 이의제기 | PolicyAppeal | 자녀가 부모에게 추가 데이터를 요청하는 과정. NORMAL(조르기)과 EMERGENCY(긴급 요청) 두 유형. |
| 조르기 | Appeal (NORMAL) | 부모 승인이 필요한 일반 이의제기 요청 |
| 긴급 요청 | Emergency Request | 월 1회 제한 (emergency_grant_month UNIQUE 제약), 100~300MB 범위, 자동 승인되는 긴급 데이터 추가 요청 |
| 이의제기 댓글 | Appeal Comment | 이의제기에 대한 부모-자녀 간 대화 메시지 스레드 |

### 미션 및 보상 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 미션 | Mission | 부모가 생성하는 행동 기반 보상 항목. 자녀가 달성 시 보상 요청 가능 |
| 미션 항목 | Mission Item | 미션의 실제 DB 엔티티. 미션 텍스트와 보상(reward) 참조를 포함 |
| 보상 템플릿 | Reward Template | 시스템에서 제공하는 보상 종류 마스터 데이터 (DATA, GIFTICON 카테고리). 상품명, 썸네일, 단가 포함 |
| 보상 인스턴스 | Reward | 미션 생성 시 템플릿에서 스냅샷 복사된 보상 엔티티. name, category, thumbnailUrl 보존 |
| 보상 지급 | Reward Grant | 미션 완료 후 사용자에게 지급된 보상 이력. 쿠폰 코드/URL, 사용 상태 관리 |
| 보상 요청 | Mission Request | 자녀가 미션 달성 후 부모에게 보상을 요청하는 행위 |

### 리캡 및 감사 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 가족 리캡 | Family Recap (FamilyRecapMonthly) | 월말 배치로 생성되는 가족 데이터 사용 리캡 스냅샷 |
| 소통 점수 | Communication Score | 가족의 소통 건강도를 나타내는 종합 지표 (0~100). NORMAL 이의제기와 미션 완료 건수 기반 계산. 이의제기 0건이면 미션 완료율로 fallback, 이의제기/미션 모두 0건일 때만 null 반환 |
| 감사 로그 | Audit Log | 시스템의 모든 주요 상태 변경을 불변 이력으로 기록하는 테이블 |
| 감사 액션 타입 | Audit Action Type | POLICY_CREATED, POLICY_UPDATED, POLICY_ASSIGNED, APPEAL_REQUESTED, APPEAL_APPROVED, APPEAL_REJECTED, EMERGENCY_GRANTED, REWARD_CREATED, REWARD_REQUESTED, REWARD_APPROVED, REWARD_REJECTED, FAMILY_MEMBER_ADDED, FAMILY_MEMBER_REMOVED |

### 알림 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 임계치 알림 | Threshold Alert | 잔여량이 특정 임계치(50/30/10%)에 도달했을 때 발송되는 알림 |
| 차단 알림 | Block Notification | 사용자가 차단되었을 때 발송되는 알림 |
| 정책 변경 알림 | Policy Update Notification | 정책이 변경되었을 때 영향받는 사용자에게 발송되는 알림 |
| 임계치당 1회 | Once Per Threshold | 각 임계치별로 한 번만 알림을 발송하는 정책 |
| 관리자 푸시 | ADMIN_PUSH | 백오피스 관리자가 특정 가족에게 직접 발송하는 푸시 알림 |
| 정책 변경 알림 (이벤트) | POLICY_CHANGED | 정책 변경 시 발행되는 notification-events 이벤트 타입 |

### API 도메인 관련

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 마이페이지 | MyPage | 로그인한 사용자 본인의 사용량 및 적용 정책을 조회하는 개인 영역 (/customers/usage, /customers/policies) |
| 데이터 차단/허용 | Data Block | Owner(복수 가능)가 특정 구성원의 데이터 사용을 즉시 차단 또는 허용하는 통합 기능 (PATCH /families/policies) |
| 정책 템플릿 | Policy Template | 관리자(admin)가 생성/관리하는 재사용 가능한 정책 정의 (/policies/*) |
| REST 알림 조회 | REST Notification | SSE 실시간 알림과 병행하여 알림 이력을 REST API로 조회하는 기능 (/notifications/*) |
| 고객 | Customer | 시스템의 일반 사용자 (가족 구성원). 기존 USER 테이블에서 CUSTOMER로 분리 |
| 고객 ID | Customer ID | API 응답에서 사용하는 사용자 식별자 (`customerId`). ERD의 `customer.id`에 대응 |
| 운영자 테이블 | Admin | 백오피스 운영자 전용 테이블. CUSTOMER와 독립된 인증 체계 |
| 고객 도메인 | Customers Domain | 고객 인증/개인 조회를 통합한 API 도메인 (/customers/*) |
| 구성원 월별 할당량 | Customer Quota | 구성원별 월별 데이터 한도와 사용량, 차단 상태를 관리하는 엔티티. 기존 MEMBER_QUOTA에서 CUSTOMER_QUOTA로 변경 |
| 미션 도메인 | MISSIONS | 미션 생성/조회/삭제, 미션 상태 로그(/missions/logs) 및 요청 이력(/missions/history)을 처리하는 API 도메인 (/missions/*) |
| 보상 도메인 | REWARDS | 보상 요청/승인/거절, 보상 템플릿 조회를 처리하는 API 도메인 (/rewards/*) |
| 리캡 도메인 | RECAPS | 월간 가족 리캡을 처리하는 API 도메인 (/recaps/*) |
| 업로드 도메인 | UPLOADS | 이미지 업로드 (R2 Object Storage → CDN URL 반환)를 처리하는 API 도메인 (/uploads/*) |
| 푸시 도메인 | PUSH | PWA Web Push 구독 관리 및 발송 API 도메인 (/push/*). VAPID 공개키, 구독/해지, 발송 4개 엔드포인트 |

---

## 2. 기술 용어

### 아키텍처 패턴

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 이벤트 기반 아키텍처 | Event-Driven Architecture (EDA) | 이벤트의 발생, 감지, 처리를 중심으로 설계된 아키텍처 패턴 |
| Write-Behind | Write-Behind Pattern | 먼저 캐시(Redis)에 기록하고, 나중에 비동기로 DB에 반영하는 패턴 |
| Write-Through | Write-Through Pattern | 캐시와 DB에 동시에 기록하는 패턴 |
| Outbox Pattern | Transactional Outbox | 비즈니스 트랜잭션과 동일 TX에서 이벤트를 outbox 테이블에 적재한 뒤, 별도 배치가 메시지 브로커로 발행하는 패턴. processor-usage의 usage_event_outbox에 적용 |
| 직접 DB 정산 | Direct DB Settlement | Redis/Lua 실시간 처리 직후 동일 흐름에서 MySQL(usage_record, customer_quota, family_quota)까지 직접 반영하는 패턴 |
| Soft Delete | Soft Delete | 물리적 삭제 대신 `deleted_at` 컬럼에 삭제 시각을 기록하는 논리 삭제 방식. 데이터 복구와 감사 추적을 보장 |
| 멱등성 | Idempotency | 동일 요청을 여러 번 처리해도 결과가 같은 성질 |
| CQRS | Command Query Responsibility Segregation | 명령(쓰기)과 조회(읽기)를 분리하는 패턴 |

### 데이터 처리

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 원자 연산 | Atomic Operation | 분리할 수 없는 단일 단위로 실행되는 연산. 전부 성공하거나 전부 실패 |
| 파티션 키 | Partition Key | Kafka에서 메시지를 파티션에 분배하는 기준이 되는 키 (familyId) |
| 컨슈머 그룹 | Consumer Group | 동일한 토픽을 구독하는 컨슈머들의 집합 |
| 컨슈머 랙 | Consumer Lag | 메시지 발행 속도와 처리 속도의 차이로 발생하는 지연 |
| DLQ | Dead Letter Queue | 처리에 실패한 메시지를 저장하는 별도의 큐 |

### Redis 데이터 설계

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 런타임 제약 | Runtime Constraints | 다양한 정책이 계산되어 실제 적용될 제약 조건의 모음. Redis Hash (`family:{fid}:customer:{cid}:constraints`)에 `ACTION:TYPE` 형태의 Field로 저장 |
| 이벤트 봉투 | Event Envelope | 모든 Kafka 이벤트를 감싸는 공통 래퍼 구조. `eventId`, `eventType`, `timestamp`, `payload` 필드로 구성. subType 없는 평탄화 envelope이며 `payload.type`으로 알림 유형 구분. Jackson `@JsonTypeInfo`를 활용한 다형성 역직렬화 지원 |
| 정책 키 | Policy Key | 런타임 제약의 Field Name 표준. `ACTION:TYPE:TARGET` 형태 (예: `LIMIT:DATA:DAILY`, `BLOCK:APP:com.youtube`, `THROTTLE:SPEED`). 기존 Enum 기반 `policyType`을 대체 |
| Poly-Policy Engine | Poly-Policy Engine | Redis Lua Script 기반의 다중 정책 평가 엔진. constraints Hash를 동적으로 순회하며 BLOCK/LIMIT/THROTTLE 등 다양한 정책을 코드 수정 없이 평가 |

### 동시성 제어

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 분산 락 | Distributed Lock | 여러 서버에서 동시에 같은 리소스에 접근하는 것을 방지하는 잠금 메커니즘 |
| Lua 스크립트 | Lua Script | Redis에서 여러 명령을 원자적으로 실행하기 위해 사용하는 스크립트 |
| 낙관적 락 | Optimistic Lock | 충돌이 발생하지 않을 것으로 가정하고 처리 후 검증하는 방식 |
| 비관적 락 | Pessimistic Lock | 충돌이 발생할 것으로 가정하고 먼저 잠금을 획득하는 방식 |
| CAS | Compare-And-Set | 현재 값을 비교하고 일치할 때만 새 값으로 설정하는 원자 연산 |

### 장애 대응

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 서킷 브레이커 | Circuit Breaker | 장애가 발생한 서비스에 대한 호출을 일시적으로 차단하는 패턴 |
| 폴백 | Fallback | 주 시스템 장애 시 대체 시스템으로 전환하는 것 (예: Redis 장애 시 DB Fallback) |
| Fail-Open | Fail-Open | 장애 시 제한 없이 허용하는 정책 (사용성 우선) |
| Fail-Closed | Fail-Closed | 장애 시 모든 요청을 차단하는 정책 (안전 우선) |
| 재시도 | Retry | 실패한 작업을 다시 시도하는 것 |
| 지수 백오프 | Exponential Backoff | 재시도 간격을 점진적으로 늘리는 전략 (1초, 5초, 30초 등) |

### 데이터 보관

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| Batch 정산 | Reconciliation | Redis와 RDS 간 데이터 불일치를 주기적으로 보정하는 작업. 매일 새벽 3시 실행 |
| 계층형 보관 | Tiered Storage (Hot/Warm/Cold) | 데이터 접근 빈도에 따라 저장소를 분리하는 전략. Hot(7일: Redis+RDS), Warm(90일: RDS), Cold(90일+: S3) |

### 실시간 통신

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| SSE | Server-Sent Events | 서버에서 클라이언트로 단방향 실시간 스트림을 전송하는 기술 |
| WebSocket | WebSocket | 서버와 클라이언트 간 양방향 실시간 통신을 지원하는 프로토콜 |
| 푸시 알림 | Push Notification | 서버에서 클라이언트로 능동적으로 전송하는 알림 |
| PWA | Progressive Web App | 웹 기술로 구현된 네이티브 앱 같은 웹 애플리케이션 |
| VAPID | Voluntary Application Server Identification | Web Push 프로토콜에서 애플리케이션 서버를 식별하는 키 쌍. PWA Push 구독에 사용 |

---

## 3. 시스템 컴포넌트 용어

| 컴포넌트 | 역할 |
|----------|------|
| **simulator-usage** | 실시간 데이터 소모 이벤트 시뮬레이터. eventId 생성 후 Kafka로 직접 발행 |
| **processor-usage** | 이벤트 검증, 중복 체크, 정책평가/쿼터차감, Redis Atomic, 직접 DB 정산(usage_record·customer_quota·family_quota), usage_event_outbox 적재 → 배치 서버가 notification-events로 후행 발행 |
| **api-core** | 5개 도메인 REST API (CUSTOMERS/FAMILIES/POLICIES/NOTIFICATIONS/ADMIN), JWT familyId 추론, 정책 즉시 반영 트리거 |
| **api-notification** | notification-events consumer, SSE API + REST 알림 조회 API, 실시간 알림 Push |
| **web-core** | Turborepo 기반 프론트엔드 모노레포, web-service + web-admin + shared 패키지 포함 |
| **web-service** | 가족 사용자 PWA (www.dabom.site), 가족 대시보드, Owner 정책 관리, 실시간 알림 UI |
| **web-admin** | 백오피스 관리 UI (admin.dabom.site), 정책 템플릿 관리, 가족/구성원 조회, 감사 로그 |

> **Note (v3.0)**: Traffic Generator → simulator-usage, Usage Engine → processor-usage, api-message → api-notification (API Layer로 이동), web-user + web-admin → web-core 모노레포로 통합 (web-service: www.dabom.site, web-admin: admin.dabom.site 서브도메인 분리)

---

## 4. 이벤트 타입

| 이벤트 | 설명 |
|--------|------|
| `usage-events` | 데이터 사용 이벤트 (simulator-usage → Kafka → processor-usage) |
| `policy-updated` | 정책 변경 이벤트 (api-core → processor-usage) |
| `notification-events` | 통합 알림 이벤트 (usage_event_outbox 배치 발행 + api-core 도메인 이벤트 → api-notification). 14종 type: QUOTA_UPDATED, THRESHOLD_ALERT, CUSTOMER_BLOCKED, CUSTOMER_UNBLOCKED, POLICY_CHANGED, MISSION_CREATED, REWARD_REQUESTED, REWARD_APPROVED, REWARD_REJECTED, APPEAL_CREATED, APPEAL_APPROVED, APPEAL_REJECTED, EMERGENCY_APPROVED, ADMIN_PUSH |

> **Note (v3.0)**: 기존 quota-updated, user-blocked, threshold-alert, user-unblocked 토픽이 notification-events 단일 토픽으로 통합

---

## 5. 약어 정의

| 약어 | 전체 표현 | 의미 |
|------|----------|------|
| TPS | Transactions Per Second | 초당 트랜잭션 수 |
| QoS | Quality of Service | 서비스 품질 (우선순위, 속도 제어 등) |
| SSE | Server-Sent Events | 서버에서 클라이언트로의 단방향 스트림 |
| PWA | Progressive Web App | 설치 가능한 웹 앱 |
| JWT | JSON Web Token | JSON 기반 인증 토큰 |
| RBAC | Role-Based Access Control | 역할 기반 접근 제어 |
| DLQ | Dead Letter Queue | 실패 메시지 저장 큐 |
| TTL | Time To Live | 데이터 유효 기간 |
| AOF | Append Only File | Redis 영속성 방식 |
| RDS | Relational Database Service | 관계형 데이터베이스 서비스 |
| MSK | Managed Streaming for Apache Kafka | AWS 관리형 Kafka |
| ECS | Elastic Container Service | AWS 컨테이너 서비스 |
| ALB | Application Load Balancer | 애플리케이션 로드 밸런서 |
| WAF | Web Application Firewall | 웹 애플리케이션 방화벽 |
| CDN | Content Delivery Network | 콘텐츠 전송 네트워크 |
| R2 | Cloudflare R2 | S3 호환 오브젝트 스토리지 (이그레스 무과금) |
| KST | Korea Standard Time | 한국 표준시 |
| P99 | 99th Percentile | 상위 99% 기준값 |
| ADR | Architecture Decision Record | 아키텍처 결정 기록 |

---

## 6. 다중 Owner 관련 용어

| 용어 (한글) | 용어 (영문) | 정의 |
|------------|------------|------|
| 다중 Owner | Multiple Owners | 한 가족 그룹 내에 복수의 OWNER 역할 구성원이 존재하는 구조. `family_member.role='OWNER'`로 판단 |
| Last Write Wins | LWW (Last Write Wins) | 복수 OWNER가 동일 정책을 동시에 수정할 경우, 마지막으로 수정한 값이 적용되는 충돌 해결 전략. 모든 변경은 `audit_log`에 기록 |
| 그룹 생성자 | Created By | 가족 그룹을 최초로 생성한 사용자. `family.created_by_id`에 기록되며, 이력/감사 전용으로 OWNER 권한 판단에는 사용하지 않음 |

---

## 7. 차단 사유 코드

| 코드 | 설명 |
|------|------|
| `MONTHLY_LIMIT_EXCEEDED` | 월별 한도 초과 |
| `FAMILY_QUOTA_EXCEEDED` | 가족 할당량 소진 |
| `TIME_BLOCK` | 시간대 차단 정책 |
| `MANUAL` | Owner에 의한 수동 차단 |
| `APP_BLOCK` | 앱별 차단 정책 (MVP 제외) |

---

## 관련 문서

- [기획서](./SPECIFICATION.md)
- [아키텍처 설계서](./ARCHITECTURE.md)
- [API 명세서](./API_SPECIFICATION.md)
- [데이터 모델](./DATA_MODEL.md)
- [ERD 설계서](./ERD.md)
- [Kafka 토픽 설계서](./designs/kafka/TOPIC_DESIGN.md)
- [Kafka 메시지 스키마](./designs/kafka/MESSAGE_SCHEMA.md)
- [Redis Key 설계서](./designs/redis/KEY_DESIGN.md)
