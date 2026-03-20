# 다봄(Dabom) 백엔드 통합 명세서

## 버전 이력

| 버전 | 날짜 | 변경 내용 |
|------|------|-----------|
| v25.0 | 2026-03-20 | 전체 문서 v25.0 Major 버전 동기화 |
| v24.5 | 2026-03-19 | `ERD.md` v24.5 동기화: weekly/monthly family recap 성능 개선용 인덱스 6종(`mission_item`, `mission_request`, `mission_log`, `policy_appeal`)을 반영하고, `notification_log.type`/알림 타입 표기를 현재 ENUM(`CUSTOMER_BLOCKED`, `CUSTOMER_UNBLOCKED`, `EMERGENCY_APPROVED`, `ADMIN_PUSH`) 기준으로 정리 |
| v24.0 | 2026-03-16 | usage-events 처리 흐름과 notification outbox 구조를 통합 반영: `usage-persist`/`usage-realtime` 토픽 및 관련 payload 제거, processor-usage가 Redis/Lua 이후 `usage_record`·`customer_quota`·`family_quota`를 직접 DB 정산하도록 수정. `usage_event_outbox` 신규 테이블 추가, notification은 Outbox 적재 후 배치 서버가 `notification-events`로 후행 발행하며 payload는 평탄화 형태로 정리 |
| v23.2 | 2026-03-15 | API_SPECIFICATION v23.1 동기화: NOTIFICATIONS 엔드포인트 갱신 (커서 무한스크롤, type 필터, 30일 제한, RESTful 경로 개선), PUSH 도메인 추가 (4개 엔드포인트), SSE 경로 `/events/stream` 변경 |
| v23.1 | 2026-03-15 | `family` 월별 상태를 `family_quota`로 분리하고 Family Redis 키(`info`, `remaining`, `alert`)에 `{yyyyMM}` suffix 규칙 반영 |
| v23.0 | 2026-03-15 | Major 버전 동기화 |
| v22.0 | 2026-03-12 | API_SPECIFICATION v22.2 동기화: UPLOADS 도메인 추가 (upload 패키지, `POST /uploads/images`, UploadType enum), 보상 템플릿 상세 조회 엔드포인트 추가 (`GET /admin/rewards/templates/{id}`) |
| v21.0 | 2026-03-12 | API_SPECIFICATION v21.6 동기화: 도메인 리네이밍 (negotiation→appeal, report→recap), 엔티티/Enum 갱신, 엔드포인트 경로 갱신, 알림 타입 APPEAL_* 전환, admin 로그인/리프레시 권한 수정 |
| v11.0 | 2026-03-03 | 2차기획서 Phase 2 기능 반영: 5개 신규 도메인 패키지, 신규 Enum/엔티티, 엔드포인트 확장, 알림 타입 확장, 기능 명세 추가 |

---

## 시스템 개요

다봄(Dabom) 백엔드는 3개의 마이크로서비스로 구성된 이벤트 기반 데이터 사용량 관리 플랫폼입니다.

| 서비스 | 역할 | REST API | Kafka |
|--------|------|----------|-------|
| **api-core** | 핵심 API 서버 (인증, 가족/정책/사용량 관리) | O | Producer + Consumer |
| **processor-usage** | 실시간 사용량 처리 프로세서 | X (순수 이벤트 처리) | Consumer + Producer |
| **api-notification** | 실시간 알림 서비스 (SSE + REST) | O | Consumer |

---

## 공통 기술 스택

| 항목 | 기술 |
|------|------|
| Framework | Spring Boot 3.4 |
| Language | Java 21 |
| ORM | Spring Data JPA + QueryDSL 5.1 |
| Cache | Spring Data Redis |
| Messaging | Spring Kafka |
| Database | MySQL (RDS) |
| Build | Gradle (Groovy DSL), Java 21 Toolchain |
| Container | Docker (Multi-stage: eclipse-temurin:21-jdk → eclipse-temurin:21-jre) |
| Observability | Micrometer + Prometheus + OpenTelemetry, Logstash Logback Encoder |

### 서비스별 추가 스택

| 서비스 | 추가 기술 |
|--------|----------|
| api-core | JWT (jjwt 0.11.2), Springdoc OpenAPI (Swagger), SSE (SseEmitter) |
| processor-usage | Redis Lua Script (원자적 연산) |
| api-notification | JWT (jjwt 0.11.2), Springdoc OpenAPI (Swagger), SSE (SseEmitter) |

---

## 아키텍처

### 전체 시스템 흐름도

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  web-service │   │  web-admin   │   │ simulator-   │
│  (프론트엔드) │   │  (관리자)     │   │ usage (외부) │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                   │
       ▼                  ▼                   │
┌──────────────────────────┐                  │
│        api-core          │                  │
│  (REST API + JWT 인증)   │                  │
│  - 사용자/관리자 인증     │                  │
│  - 가족/정책/사용량 CRUD  │                  │
│  - SSE 실시간 사용량      │                  │
└──────┬───────────────────┘                  │
       │ policy-updated                       │ usage-events
       ▼                                      ▼
┌─────────────────────────────────────────────────┐
│              processor-usage                     │
│  (Kafka Consumer/Producer, REST API 없음)        │
│  - Redis Lua Script 원자적 사용량 처리            │
│  - 차단/경고 판정                                 │
│  - DB 직접 정산 (`usage_record`, `customer_quota`, `family_quota`) │
│  - usage_event_outbox 적재                        │
└──┬──────────────┬──────────────────┬────────────────────┘
   │              │                  │
   │ MySQL        │ usage_event_     │ notification-
   │ direct       │ outbox           │ events
   ▼              ▼                  ▼
┌──────────┐ ┌──────────────┐ ┌──────────────────┐
│DB 테이블  │ │ 배치 서버     │ │ api-notification │
│(usage_   │ │ (Outbox 조회, │ │ (SSE 실시간 알림,│
│record 등)│ │  후행 발행)   │ │  REST 알림 조회) │
└──────────┘ └──────────────┘ └──────────────────┘
```

### 패키지 구조 (Feature-Based Layered Architecture)

3개 서비스 공통 구조:

```
com.project
├─ Application.java
├─ domain/{feature}/
│   ├─ controller/          # REST 엔드포인트 (api-core, api-notification)
│   ├─ service/             # Interface + Impl 구조
│   │   └─ port/            # 인프라 추상화 인터페이스
│   ├─ repository/          # Spring Data JPA
│   │   └─ impl/            # QueryDSL 구현체
│   ├─ entity/              # JPA 엔티티
│   ├─ enums/               # 도메인 Enum
│   ├─ dto/
│   │   ├─ request/         # 요청 DTO (Java record)
│   │   └─ response/        # 응답 DTO (Java record + static from())
│   └─ infra/               # 인프라 어댑터
│       ├─ cache/           # Redis 캐시 Repository
│       ├─ messaging/       # Kafka Producer/Consumer
│       ├─ event/           # Spring ApplicationEvent (api-core)
│       └─ sse/             # SSE Emitter Registry (api-core)
└─ global/
    ├─ config/              # Redis, Kafka, JPA, QueryDSL, Swagger, CORS, Cache
    ├─ exception/           # ExceptionAdvice, BaseException, ErrorCode
    ├─ auth/                # JWT 인증/인가, AOP (api-core, api-notification)
    ├─ api/response/        # ApiResponse<T> 공통 응답 래퍼
    ├─ event/dto/           # 공유 Kafka 이벤트 Payload DTO
    └─ util/                # BaseEntity, RedisKeyGenerator
```

### 의존성 방향 (단방향만 허용)

```
# api-core
Controller → Service → Repository → Entity

# processor-usage (REST API 없음)
Consumer → Service → Repository → Entity
                  → Redis (Lua Script)
                  → Producer (이벤트 발행)

# api-notification
Controller → Service → Repository → Entity
                    → SseEmitterManager (SSE 푸시)
Consumer → Service → Repository → Entity
                  → SseEmitterManager (SSE 푸시)
```

---

## 도메인 구성

### api-core 도메인

| 도메인 | 패키지 | 역할 |
|--------|--------|------|
| customer | `domain/customer` | 사용자 인증(로그인), 사용량/정책 조회 |
| admin | `domain/admin` | 관리자 인증(로그인), 백오피스 관리 |
| family | `domain/family` | 가족 그룹 관리, 대시보드, 검색, SSE 실시간 사용량 |
| policy | `domain/policy` | 정책 템플릿 CRUD, 가족 구성원 정책 적용/수정 |
| usagerecord | `domain/usagerecord` | 사용량 기록 관리, SSE 실시간 스트림 |
| appeal | `domain/appeal` | 정책 이의신청 요청/승인/거절, 긴급 요청 자동승인 (월 1회 제한, UNIQUE 제약 기반) |
| mission | `domain/mission` | 미션 생성/삭제, 미션 카드 목록, 미션 로그 |
| reward | `domain/reward` | 보상 요청/승인/거절, 보상 템플릿 조회 |
| recap | `domain/recap` | 월간 가족 리포트 조회, 배치 집계 |
| audit | `domain/audit` | 감사 로그 기록/조회 |
| upload | `domain/upload` | 이미지 업로드 (R2 Object Storage), CDN URL 반환 |

### processor-usage 도메인

| 도메인 | 패키지 | 역할 |
|--------|--------|------|
| usage | `domain/usage` | 사용량 이벤트 수신, Redis Lua Script 실시간 처리, DB 직접 정산 + notification Outbox 적재 |
| policy | `domain/policy` | 정책 변경 이벤트 수신, Redis 제약 조건 동기화 |
| family | `domain/family` | 가족 그룹 정보 관리, Redis 캐시 저장/조회 |
| customer | `domain/customer` | 고객 엔티티 및 월별 할당량(CustomerQuota) 관리 |

### api-notification 도메인

| 도메인 | 패키지 | 역할 |
|--------|--------|------|
| notification | `domain/notification` | SSE 실시간 알림 + REST 알림 이력 조회 |
| admin | `domain/admin` | 관리자 엔티티 및 Repository |
| customer | `domain/customer` | 고객 엔티티, 할당량(CustomerQuota) |
| family | `domain/family` | 가족 그룹 엔티티, 구성원 매핑 |
| policy | `domain/policy` | 정책 엔티티, 정책 할당 |

---

## 데이터 모델

### 핵심 엔티티

| 엔티티 | 테이블 | 설명 | 예상 규모 |
|--------|--------|------|-----------|
| Customer | `customer` | 시스템 사용자 (가족 구성원) | ~1,000,000 |
| Admin | `admin` | 백오피스 운영자 | ~100 |
| Family | `family` | 가족 그룹 | ~250,000 |
| FamilyMember | `family_member` | 가족-사용자 매핑 (N:M 해소) | ~1,000,000 |
| CustomerQuota | `customer_quota` | 구성원별 월별 한도/사용량/차단 | ~1,000,000/월 |
| UsageRecord | `usage_record` | 데이터 사용 이력 (직접 정산) | ~432,000,000/일 |
| UsageEventOutbox | `usage_event_outbox` | notification 발행 복구용 Outbox | ~수백만/월 |
| Policy | `policy` | 정책 템플릿 정의 | ~100 |
| PolicyAssignment | `policy_assignment` | 정책 적용 매핑 | ~500,000 |
| NotificationLog | `notification_log` | 알림 발송 이력 | ~수백만/월 |
| AuditLog | `audit_log` | 감사 로그 | ~수십만/월 |
| Invite | `invite` | 가족 초대 | ~수만 |
| PolicyAppeal | `policy_appeal` | 정책 이의신청 요청 (조르기/긴급) | ~수십만/월 |
| AppealComment | `appeal_comment` | 이의신청 코멘트 | ~수십만/월 |
| RewardTemplate | `reward_template` | 보상 템플릿 | ~수백 |
| Reward | `reward` | 보상 인스턴스 (스냅샷) | ~수만 |
| RewardGrant | `reward_grant` | 보상 지급 이력 | ~수만 |
| MissionItem | `mission_item` | 미션 항목 | ~수만 |
| MissionRequest | `mission_request` | 미션 보상 요청 | ~수만/월 |
| FamilyRecapMonthly | `family_recap_monthly` | 월간 가족 리포트 | ~250,000/월 |

### 역할 Enum

```java
public enum RoleType {
    MEMBER,   // 일반 가족 구성원
    OWNER,    // Owner 계정 (복수 가능)
    ADMIN     // 백오피스 운영자
}
```

### 정책 타입 Enum

```java
public enum PolicyType {
    MONTHLY_LIMIT,   // 월별 한도
    TIME_BLOCK,      // 시간대 차단
    MANUAL_BLOCK,    // 즉시 차단
    APP_BLOCK,       // 앱별 차단
    WEBSITE_BLOCK    // 웹사이트 차단
}
```

### 이의신청 타입 Enum

```java
public enum AppealType {
    APPEAL,    // 조르기 요청
    EMERGENCY  // 긴급 요청 (월 1회, 자동승인, emergency_grant_month UNIQUE 제약)
}

public enum AppealStatus {
    PENDING,   // 요청 대기
    APPROVED,  // 승인
    REJECTED,  // 거절
    CANCELLED  // 취소
}
```

### 보상 카테고리 Enum

```java
public enum RewardCategory {
    DATA,     // 데이터
    GIFTICON  // 기프티콘
}
```

### 업로드 타입 Enum

```java
public enum UploadType {
    REWARD,   // 보상 썸네일
    PROFILE,  // 사용자 프로필
    MISSION   // 미션 관련
}
```

### 미션 상태 Enum

```java
public enum MissionStatus {
    ACTIVE,    // 활성
    COMPLETED, // 완료
    CANCELLED  // 취소
}

public enum MissionRequestStatus {
    PENDING,  // 요청 대기
    APPROVED, // 승인
    REJECTED  // 거절
}
```

### 감사 액션 Enum

```java
public enum AuditAction {
    POLICY_CREATED,
    POLICY_UPDATED,
    POLICY_ASSIGNED,
    APPEAL_REQUESTED,
    APPEAL_APPROVED,
    APPEAL_REJECTED,
    APPEAL_CANCELLED,
    EMERGENCY_GRANTED,
    REWARD_CREATED,
    REWARD_REQUESTED,
    REWARD_APPROVED,
    REWARD_REJECTED,
    FAMILY_MEMBER_ADDED,
    FAMILY_MEMBER_REMOVED
}
```

### Soft Delete 적용

- 영속 엔티티에 `deleted_at` 컬럼 존재
- `NULL` = 활성, `NOT NULL` = 삭제
- UNIQUE 제약에 `deleted_at` 포함하여 삭제 후 재생성 허용
- 모든 엔티티가 `BaseEntity`를 상속하므로 `created_at`, `updated_at`, `deleted_at` 3개 필드를 공통으로 가짐. 이력성/불변 테이블은 운영상 Soft Delete를 사용하지 않으나, `deleted_at` 컬럼은 존재 (항상 NULL)

### BaseEntity (공통 상속)

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public abstract class BaseEntity {
    @CreatedDate  private LocalDateTime createdAt;
    @LastModifiedDate  private LocalDateTime updatedAt;
    private LocalDateTime deletedAt;

    public void softDelete() { this.deletedAt = LocalDateTime.now(); }
    public boolean isDeleted() { return this.deletedAt != null; }
}
```

### NotificationLog 테이블

```sql
CREATE TABLE notification_log (
    id          BIGINT AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT NOT NULL REFERENCES customer(id),
    family_id   BIGINT NOT NULL REFERENCES family(id),
    type        ENUM(
                    'QUOTA_UPDATED',
                    'THRESHOLD_ALERT',
                    'CUSTOMER_BLOCKED',
                    'CUSTOMER_UNBLOCKED',
                    'POLICY_CHANGED',
                    'MISSION_CREATED',
                    'REWARD_REQUESTED',
                    'REWARD_APPROVED',
                    'REWARD_REJECTED',
                    'APPEAL_CREATED',
                    'APPEAL_APPROVED',
                    'APPEAL_REJECTED',
                    'EMERGENCY_APPROVED',
                    'ADMIN_PUSH'
                ) NOT NULL,
    message     TEXT NOT NULL,
    payload     JSON,
    is_read     BOOLEAN NOT NULL DEFAULT FALSE,
    sent_at     DATETIME DEFAULT CURRENT_TIMESTAMP,
    deleted_at  DATETIME NULL
);

CREATE INDEX idx_notif_customer ON notification_log(customer_id, sent_at DESC);
CREATE INDEX idx_notif_customer_type ON notification_log(customer_id, type, sent_at DESC);
CREATE INDEX idx_notif_family ON notification_log(family_id, sent_at DESC);
```

### 리캡 배치 성능용 인덱스

weekly/monthly family recap 배치의 `created_at`/`resolved_at`/`completed_at` 범위 집계를 줄이기 위해 아래 인덱스를 사용한다.

```sql
CREATE INDEX idx_mission_recap_family_created
    ON mission_item(family_id, created_at, deleted_at);

CREATE INDEX idx_mission_recap_family_completed
    ON mission_item(family_id, status, completed_at, deleted_at);

CREATE INDEX idx_mreq_recap_item_status_resolved
    ON mission_request(mission_item_id, status, resolved_at, deleted_at);

CREATE INDEX idx_mission_log_recap_item_action_created
    ON mission_log(mission_item_id, action_type, created_at, deleted_at);

CREATE INDEX idx_appeal_recap_assignment_type_created
    ON policy_appeal(policy_assignment_id, type, created_at, deleted_at);

CREATE INDEX idx_appeal_recap_assignment_type_status_resolved
    ON policy_appeal(policy_assignment_id, type, status, resolved_at, deleted_at);
```

- `idx_mission_recap_family_created`: weekly/monthly recap 미션 생성 건수 집계
- `idx_mission_recap_family_completed`: weekly/monthly recap 미션 완료 건수 집계
- `idx_mreq_recap_item_status_resolved`: weekly/monthly recap 반려 요청 집계
- `idx_mission_log_recap_item_action_created`: monthly recap carry-in 집계
- `idx_appeal_recap_assignment_type_created`: weekly/monthly recap 이의제기 생성 건수 집계
- `idx_appeal_recap_assignment_type_status_resolved`: weekly/monthly recap 이의제기 처리 건수 집계

### CustomerQuota 엔티티

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | Long | PK (auto-increment) |
| `familyId` | Long | 가족 ID |
| `customerId` | Long | 고객 ID |
| `currentMonth` | LocalDate | 기준 월 (yyyy-MM-01) |
| `monthlyUsedBytes` | Long | 월간 누적 사용량 (bytes) |
| `monthlyLimitBytes` | Long | 월간 한도 (bytes) |
| `isBlocked` | Boolean | 차단 여부 |
| `blockReason` | String | 차단 사유 |

**UNIQUE 제약**: `(family_id, customer_id, current_month)` + `deleted_at`

### FamilyQuota 엔티티

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | Long | PK (auto-increment) |
| `familyId` | Long | 가족 ID |
| `currentMonth` | LocalDate | 기준 월 (yyyy-MM-01) |
| `totalQuotaBytes` | Long | 월별 총 할당량 스냅샷 |
| `usedBytes` | Long | 월별 총 사용량 |

**UNIQUE 제약**: `(family_id, current_month)` + `deleted_at`

> 가족 월별 총량 조회의 Source of Truth는 `family`가 아니라 `family_quota`다.

---

## 인증 & 인가

### JWT 인증 (api-core, api-notification)

| 토큰 유형 | 유효기간 | 용도 |
|----------|---------|------|
| Access Token | 설정 가능 (기본 12시간) | API 인증 + SSE 연결 |
| Refresh Token | 설정 가능 (기본 14일) | Access Token 갱신 |

**JWT Payload**:
```json
{
  "sub": "12345",
  "role": "OWNER",
  "exp": 1705312200
}
```

### 인가 AOP

| 어노테이션 | 역할 | 설명 |
|------------|------|------|
| `@AdminOnly` | admin | 관리자 전용 엔드포인트 보호 |
| `@OwnerOnly` | owner | Owner 전용 엔드포인트 보호 |
| `@CustomerId` | member | JWT에서 customerId를 추출하여 파라미터에 주입 |

---

## API 엔드포인트

### api-core 엔드포인트

#### CUSTOMERS (사용자)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| POST | `/customers/login` | - | 사용자 로그인 (전화번호 + 비밀번호) |
| POST | `/customers/refresh` | member | Access Token 갱신 |
| POST | `/customers/logout` | member | 로그아웃 |
| POST | `/customers/signup` | - | 회원가입 |
| GET | `/customers/profile` | member | 내 프로필 조회 |
| PUT | `/customers/profile` | member | 내 프로필 수정 |
| GET | `/customers/usage` | member | 내 데이터 사용량 조회 |
| GET | `/customers/policies` | member | 내 적용 정책 조회 |
| GET | `/terms` | - | 이용약관 조회 |
| POST | `/customers/terms` | member | 이용약관 동의 |

#### FAMILIES (가족)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| GET | `/families/dashboard/usage?year=&month=` | member | 대시보드 사용량 조회 |
| GET | `/families/reports/usage?year=&month=` | member | 사용량 상세 리포트 |
| GET | `/families/usage/current` | member | 실시간 가족 사용량 (SSE) |
| GET | `/families/usage/customers` | member | 실시간 구성원별 사용량 |
| GET | `/families/usage/dashboard` | member | 가족 사용량 대시보드 |
| PUT | `/families` | owner | 가족 정보 수정 |
| POST | `/families/{familyId}/invite` | owner | 가족 초대 (전화번호 기반) |
| POST | `/families` | admin | 가족 목록 검색 (필터/페이징) |
| GET | `/families/{familyId}` | admin | 가족 상세 조회 |
| GET | `/families/policies` | owner | 가족 구성원 정책 조회 |
| PATCH | `/families/policies` | owner | 가족 구성원 정책 수정 |
| GET | `/appeals` | member | 이의신청 이력 목록 조회 |
| POST | `/appeals/appeal` | member | 조르기 이의신청 요청 |
| POST | `/appeals/emergency` | member | 긴급 데이터 요청 (자동승인, 월 1회, 100~300MB, UNIQUE 제약 기반) |
| PUT | `/appeals/{id}/respond` | owner | 이의신청 승인/거절 |
| POST | `/appeals/{id}/messages` | member | 이의신청 코멘트 작성 |
| GET | `/appeals/{id}/messages` | member | 이의신청 코멘트 목록 조회 |
| GET | `/rewards/templates` | member | 보상 템플릿 목록 조회 |

#### MISSIONS (미션)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| GET | `/missions` | member | 미션 카드 목록 조회 |
| POST | `/missions` | owner | 미션 생성 |
| DELETE | `/missions/{id}` | owner | 미션 삭제 (→ CANCELLED, MISSION_LOG에 MISSION_CANCELLED 기록) |
| GET | `/missions/logs` | member | 미션 상태 변화 로그 조회 (MissionLog 기반) |
| GET | `/missions/history` | member | 미션 요청 이력 조회 (MissionRequest 기반) |

#### REWARDS (보상)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| POST | `/missions/{missionId}/request` | member | 보상 요청 (ACTIVE 미션만) |
| PUT | `/rewards/requests/{id}/respond` | owner | 보상 승인/거절 |
| GET | `/rewards/received` | member(MEMBER) | 내가 받은 보상 조회 |
| GET | `/admin/rewards/grants` | admin | 보상 지급 내역 조회 |

#### RECAPS (리포트)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| GET | `/recaps/monthly` | member | 월간 가족 리포트 조회 |

#### UPLOADS (업로드)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| POST | `/uploads/images` | admin | 이미지 업로드 (R2, 5MB 제한, PNG/JPEG/WEBP) |

#### POLICIES (정책 템플릿)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| GET | `/policies` | admin | 정책 목록 조회 (페이징) |
| POST | `/policies` | admin | 정책 생성 |
| GET | `/policies/{policyId}` | admin | 정책 상세 조회 |
| PUT | `/policies/{policyId}` | admin | 정책 수정 |
| DELETE | `/policies/{policyId}` | admin | 정책 삭제 (비활성화된 것만) |

#### ADMIN (관리자)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| POST | `/admin/login` | 없음 | 관리자 로그인 (이메일 + 비밀번호) |
| POST | `/admin/refresh` | 없음 | 관리자 토큰 갱신 |
| POST | `/admin/logout` | admin | 관리자 로그아웃 |
| GET | `/admin/dashboard` | admin | 관리자 대시보드 |
| GET | `/admin/audit/logs` | admin | 감사 로그 조회 |
| GET | `/admin/rewards/templates` | admin | 보상 템플릿 목록 조회 |
| GET | `/admin/rewards/templates/{id}` | admin | 보상 템플릿 상세 조회 |
| POST | `/admin/rewards/templates` | admin | 보상 템플릿 생성 |
| PUT | `/admin/rewards/templates/{id}` | admin | 보상 템플릿 수정 |
| DELETE | `/admin/rewards/templates/{id}` | admin | 보상 템플릿 삭제 |

### api-notification 엔드포인트

#### NOTIFICATIONS (알림)

| 메서드 | 경로 | 권한 | 설명 |
|--------|------|------|------|
| GET | `/notifications` | member | 알림 목록 조회 (커서 무한스크롤, type·isRead 필터, 30일 제한) |
| GET | `/notifications/unread-count` | member | 읽지 않은 알림 수 조회 (30일 이내) |
| PATCH | `/notifications/{notificationId}/read` | member | 개별 알림 읽음 처리 |
| PATCH | `/notifications/read-all` | member | 전체 알림 읽음 처리 |
| DELETE | `/notifications/{notificationId}` | member | 알림 삭제 (소프트) |
| GET | `/events/stream` | member | SSE 실시간 이벤트 스트림 |
| GET | `/push/vapid-public-key` | 없음 | VAPID 공개키 조회 |
| POST | `/push/subscribe` | member | Push 구독 등록 |
| DELETE | `/push/subscribe` | member | Push 구독 해제 |
| POST | `/push/send` | admin | Push 수동 발송 (운영용) |

### API 공통 응답 형식

```json
// 성공
{ "success": true, "data": { ... }, "timestamp": "..." }

// 실패
{ "success": false, "error": { "code": "...", "message": "..." }, "timestamp": "..." }
```

---

## 이벤트 아키텍처

### Kafka 토픽 구성

| 토픽 | 발행자 | 소비자 | 파티션 키 | 설명 |
|------|--------|--------|-----------|------|
| `usage-events` | simulator-usage | processor-usage | familyId | 원본 데이터 사용량 이벤트 |
| `policy-updated` | api-core | processor-usage | familyId | 정책 변경 이벤트 |
| `notification-events` | 배치 서버 | api-notification | - | 통합 알림 이벤트 (`usage_event_outbox` 조회 후 발행) |

### EventEnvelope 구조 (공통 이벤트 래퍼)

```java
public record EventEnvelope<T>(
    String eventId,           // UUID
    String eventType,         // DATA_USAGE | POLICY_UPDATED | NOTIFICATION
    LocalDateTime timestamp,
    T payload
) { }
```

### eventType → Payload 매핑

| eventType | Payload 클래스 | 설명 |
|-----------|---------------|------|
| `DATA_USAGE` | `UsagePayload` | 원본 데이터 사용량 |
| `POLICY_UPDATED` | `PolicyUpdatedPayload` | 정책 변경 |
| `NOTIFICATION` | `NotificationEventPayload` | 평탄화된 알림 이벤트 |

### 이벤트 페이로드 상세

#### UsagePayload (원본 사용량)
```json
{
  "eventId": "evt_abc123",
  "familyId": 100,
  "customerId": 200,
  "appId": "com.youtube",
  "bytesUsed": 1048576,
  "metadata": {"network": "WIFI"}
}
```

#### PolicyUpdatedPayload (정책 변경)
```json
{
  "familyId": 100,
  "targetCustomerId": 200,
  "policyKey": "LIMIT:DATA:MONTHLY",
  "oldValue": "5368709120",
  "newValue": "3221225472",
  "changedBy": 1
}
```

#### NotificationEventPayload (평탄화된 알림 이벤트)
```json
{
  "familyId": 100,
  "customerId": 1,
  "type": "THRESHOLD_ALERT",
  "title": "데이터 사용량 경고",
  "message": "가족 데이터 잔여량이 10% 미만입니다.",
  "data": {
    "threshold": 10,
    "triggerStatus": "WARNING_10"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| `payload.familyId` | Long | 가족 그룹 ID |
| `payload.customerId` | Long | 대상 고객 ID |
| `payload.type` | String | 알림 타입 |
| `payload.title` | String | 알림 제목 |
| `payload.message` | String | 알림 메시지 |
| `payload.data` | Object | 알림 타입별 최소 필드 집합 (`THRESHOLD_ALERT` 예: `threshold`, `triggerStatus`) |

### 전체 이벤트 흐름도

```
simulator-usage ──usage-events──> [processor-usage] ──직접 DB 정산──> [MySQL]
                                                    ──usage_event_outbox──> [배치 서버] ──notification-events──> [api-notification (SSE + DB)]

api-core ──policy-updated──> [processor-usage (Redis 제약 조건 동기화)]
```

---

## processor-usage 핵심 로직

### 사용량 처리 흐름 (UsageSyncService)

```
UsageEventsConsumer
  1. ConsumerRecord 수신
  2. EventEnvelope<UsagePayload> 역직렬화
  3. UsageEventValidator 검증
  4. UsageSyncService.syncUsage() 위임
       │
       ▼
  Redis Lua Script (usage_update.lua) 실행
  1. 개인 제약 조건(BLOCK:ACCESS) 확인
  2. 개인 월간 한도(LIMIT:DATA:MONTHLY) 초과 확인
  3. 가족 잔여량 부족 확인
  4. 가족 잔여량 차감 (DECRBY)
  5. 개인 월간 사용량 증가 (INCRBY)
  6. 임계치 경고 판정 (WARNING_10/30/50)
  7. 경고 중복 발송 방지 (HEXISTS/HSET)
       │
       ▼
  DB 직접 정산 + notification Outbox 적재
  ├─ usage_record / customer_quota / family_quota 직접 반영
  └─ usage_event_outbox (알림 발행 의도, 조건부)
```

### 사용량 상태 판정

| 상태 | 의미 | 트리거 조건 | 후속 동작 |
|------|------|------------|----------|
| `NORMAL` | 정상 | 차단/경고 해당 없음 | Persist + Realtime 발행 |
| `WARNING_10` | 잔여 10% 미만 | 잔여/총량 < 0.1 (최초 1회) | + ThresholdAlert 발행 |
| `WARNING_30` | 잔여 30% 미만 | 잔여/총량 < 0.3 (최초 1회) | + ThresholdAlert 발행 |
| `WARNING_50` | 잔여 50% 미만 | 잔여/총량 < 0.5 (최초 1회) | + ThresholdAlert 발행 |
| `BLOCKED_ACCESS` | 완전 차단 | `constraints[BLOCK:ACCESS] == "1"` | + CustomerBlocked 발행 |
| `BLOCKED_LIMIT_MONTHLY` | 월간 한도 초과 | 월간 사용량 + bytesUsed > limit | + CustomerBlocked 발행 |
| `BLOCKED_FAMILY_QUOTA` | 가족 데이터 소진 | 가족 잔여량 < bytesUsed | + CustomerBlocked 발행 |

### 정책 제약 조건 동기화

```
PolicyKafkaConsumer
  1. policy-updated 토픽 수신
  2. POLICY_UPDATED 타입만 처리
  3. PolicyConstraintSyncService.sync() 위임
       │
       ▼
  Redis Lua Script (policy_constraint_update.lua) 실행
  1. 이벤트 중복 방지 (SET NX EX)
  2. 버전 비교 (stale event 방어)
  3. HSET 또는 HDEL 수행
  결과: APPLIED / DUPLICATE / STALE / INVALID_REQUEST
```

### 허용된 정책 키 (Whitelist)

| 정책 키 | 값 형식 | 설명 |
|---------|---------|------|
| `BLOCK:ACCESS` | `"0"` / `"1"` | 완전 접근 차단 |
| `BLOCK:TIME:START` | `"HHmm"` | 시간대 차단 시작 |
| `BLOCK:TIME:END` | `"HHmm"` | 시간대 차단 종료 |
| `THROTTLE:SPEED` | 양의 정수 (bytes/s) | 속도 제한 |
| `BLOCK:APP:{appId}` | `"0"` / `"1"` | 특정 앱 차단 |
| `LIMIT:DATA:{period}` | 양의 정수 (bytes) | 데이터 한도 |

### DB 영속화 (직접 정산)

```
usage-events 수신 → Redis warmup → Lua 실행 → 상태 해석 → `usage_record` / `customer_quota` / `family_quota` 직접 정산 → Outbox 상태 확정
```

---

## Redis 연동

### Redis Key 구조

| 키 패턴 | 타입 | 설명 |
|---------|------|------|

| `family:{familyId}:info:{yyyyMM}` | Hash | 가족 월별 메타 정보 (name, total_quota 등) |
| `family:{familyId}:remaining:{yyyyMM}` | String | 가족 월별 잔여 데이터량 (bytes) |
| `family:{familyId}:customer:{customerId}:usage:monthly:{yyyyMM}` | String | 고객 월간 사용량 (bytes) |
| `family:{familyId}:customer:{customerId}:constraints` | Hash | 고객별 정책 제약 조건 |
| `family:{familyId}:alert:THRESHOLD:{threshold}:{yyyyMM}` | String | 월별 임계치 경고 발송 이력 |
| `event:dedup:policy:{eventId}:{customerId}` | String | 정책 이벤트 중복 방지 (TTL: 1시간) |
| `event:dedup:usage:{eventId}` | String | 사용량 이벤트 중복 감지 (TTL 기반) |

### RedisTemplate 구성

| Bean | 용도 | Value Serializer |
|------|------|------------------|
| `familyCacheRedisTemplate` | 가족 캐시 | Jackson2JsonRedisSerializer (FamilyCacheDto) |
| `exampleCacheRedisTemplate` | 예제 캐시 | GenericJackson2JsonRedisSerializer |
| `familyStringRedisTemplate` | 문자열/Lua Script | StringRedisSerializer |

### Redis 캐시 전략

**Cache Look-Aside 패턴** (api-core 기본):
1. 캐시 확인 → 2. 캐시 미스 시 DB 조회 → 3. 캐시 저장 (TTL 설정)

---

## SSE (Server-Sent Events)

### api-core SSE

- `GET /families/usage/current` — 가족 총 사용량 실시간 Push
- `SseEmitter` 기반 (`UsageSseEmitterRegistry`)
- Spring `ApplicationEvent` → `@EventListener` → SSE Push

### api-notification SSE

- `GET /events/stream` — 개인별 알림 실시간 Push (12개 이벤트 타입)
- JWT 기반 사용자 식별
- Heartbeat: 30초 간격, 재연결: `Last-Event-ID` 헤더 활용

### SSE Event Types (api-notification)

| Event Type | 설명 |
|------------|------|
| `QUOTA_UPDATED` | 할당량 변경 |
| `CUSTOMER_BLOCKED` | 고객 차단 |
| `THRESHOLD_ALERT` | 임계치 알림 |
| `CUSTOMER_UNBLOCKED` | 차단 해제 |

---

## 알림 체계

### 알림 타입

| type | 설명 | 트리거 조건 | 알림 대상 |
|------|------|------------|----------|
| `THRESHOLD_ALERT` | 잔여량 임계치 도달 | 가족 잔여량 50%, 30%, 10% 미만 | 전체 가족 구성원 |
| `QUOTA_UPDATED` | 할당량 변경 | Owner/Admin이 월별 한도 또는 가족 총량을 수정 | 영향받는 구성원 |
| `CUSTOMER_BLOCKED` | 고객 차단 | 한도 초과, 수동 차단, 시간대 차단 | 해당 구성원 + Owner |
| `CUSTOMER_UNBLOCKED` | 차단 해제 | 시간대 차단 종료, Owner 수동 해제 | 해당 구성원 |
| `POLICY_CHANGED` | 정책 변경 | Owner/Admin이 정책 수정 | 영향받는 구성원 |
| `APPEAL_CREATED` | 이의신청 요청 | Member가 조르기/긴급 요청 | Owner |
| `APPEAL_APPROVED` | 이의신청 승인 | Owner가 이의신청 승인 | 요청한 Member |
| `APPEAL_REJECTED` | 이의신청 거절 | Owner가 이의신청 거절 | 요청한 Member |
| `EMERGENCY_APPROVED` | 긴급 요청 자동승인 | 긴급 요청 조건 충족 (월 1회 이내) | 요청한 Member + Owner |
| `MISSION_CREATED` | 미션 생성 | Owner가 미션 생성 | 전체 가족 구성원 |
| `REWARD_REQUESTED` | 보상 요청 | Member가 보상 요청 | Owner |
| `REWARD_APPROVED` | 보상 승인 | Owner가 보상 승인 | 요청한 Member |
| `REWARD_REJECTED` | 보상 거절 | Owner가 보상 거절 | 요청한 Member |
| `ADMIN_PUSH` | 관리자 수동 Push | 운영자가 특정 사용자/가족에게 공지 발송 | 지정된 수신자 |

### 차단 사유 코드

| blockReason | 설명 |
|-------------|------|
| `BLOCKED_ACCESS` | 완전 접근 차단 (Owner 수동 차단) |
| `BLOCKED_LIMIT_MONTHLY` | 월간 개인 한도 초과 |
| `BLOCKED_FAMILY_QUOTA` | 가족 공유 데이터 소진 |
| `BLOCKED_TIME` | 시간대 차단 정책 적용 |

### 중복 발송 방지

- **임계치 알림**: 50%, 30%, 10% 각각 한 번만 발송 (Redis `family:{familyId}:alert:THRESHOLD:{threshold}:{yyyyMM}` String key 존재 여부로 중복 방지)
- **정책 이벤트**: `event:dedup:policy:{eventId}:{customerId}` (TTL 1시간)
- **사용량 이벤트**: `event:dedup:usage:{eventId}` (TTL 기반)

---

## 코드 품질 & 스타일 가이드 (공통)

### 포매팅

- **Google Java Format** (AOSP 스타일, 4-space 인덴트)
- `./gradlew spotlessApply`로 자동 적용

### Import 순서

```
java → javax → jakarta → org → net → com → 기타 → lombok
```

### Checkstyle

- **Naver Java Convention** 적용
- `./gradlew checkstyleMain`으로 검증
- `maxWarnings = 0` (경고 0개 정책)

### 아키텍처 규칙

| 규칙 | 설명 |
|------|------|
| Entity | Setter 금지, `@Builder`/static factory 생성, BaseEntity 상속 |
| DTO | Response는 `static from()` + record, Request에 `toEntity()` 금지 |
| Service | Interface + Impl 구조, `@Transactional`은 Service에만, 100줄 초과 시 분리 |
| 예외 처리 | `ApplicationException` 사용, 도메인별 `ErrorCode` enum, RuntimeException 직접 throw 금지 |
| API 응답 | 모든 응답 `ApiResponse<T>`로 통일 |

### 테스트

- **JUnit 5** + **Mockito** + **Spring Boot Test**
- **EmbeddedKafka** (Kafka 통합 테스트)
- **H2** 인메모리 DB (테스트용)
- **JaCoCo** 커버리지 리포트 (목표 70%)

---

## Observability (공통)

### Actuator 엔드포인트

| 경로 | 설명 |
|------|------|
| `/actuator/health` | 헬스 체크 (liveness/readiness probe) |
| `/actuator/prometheus` | Prometheus 메트릭 수집 |
| `/actuator/metrics` | 시스템 메트릭 |
| `/actuator/info` | 애플리케이션 정보 |
| `/actuator/loggers` | 런타임 로그 레벨 변경 |

### 로깅

- **Logstash Logback Encoder**: 구조화된 JSON 로그 출력
- **로그 보안**: 외부 입력값 `sanitizeForLog()` 처리 (CR/LF/TAB 치환, 128자 제한)

### 분산 트레이싱

- **Micrometer Tracing** + **OpenTelemetry Exporter**
- OTLP 엔드포인트로 trace 데이터 전송
- 환경변수: `OTEL_TRACING_ENABLED`, `OTEL_SAMPLING_PROBABILITY`

---

## 환경 변수 (공통)

| 변수 | 기본값 | 사용 서비스 |
|------|--------|------------|
| `DATABASE_URL` | `jdbc:mysql://localhost:3310/app_db?...` | 전체 |
| `DATABASE_USER` | `app_user` | 전체 |
| `DATABASE_PASSWORD` | `app_password` | 전체 |
| `REDIS_HOST` | `localhost` | 전체 |
| `REDIS_PORT` | `6379` | 전체 |
| `KAFKA_BOOTSTRAP_SERVERS` | `localhost:9092` | 전체 |
| `SERVER_PORT` | `8080` | 전체 |
| `FRONTEND_URL` | `http://localhost:3000` | 전체 |
| `OTEL_TRACING_ENABLED` | `false` | 전체 |

### 서비스별 전용 환경 변수

| 변수 | 서비스 | 설명 |
|------|--------|------|
| `JWT_SECRET_KEY` | api-core, api-notification | JWT 서명 키 |
| `JWT_ACCESS_TOKEN_EXPIRES_IN` | api-core, api-notification | Access Token 유효기간 (ms) |
| `JWT_REFRESH_TOKEN_EXPIRES_IN` | api-core, api-notification | Refresh Token 유효기간 (ms) |
| `KAFKA_CONSUMER_GROUP_ID` | 전체 (값 다름) | api-core: `dabom-api-core-dev`, processor-usage: `dabom-processor-usage-dev`, api-notification: `dabom-notification-dev` |
| `KAFKA_POLICY_DEDUP_TTL_SECONDS` | processor-usage | 정책 이벤트 중복 방지 TTL (기본 3600초) |
| `KAFKA_USAGE_PERSIST_DEDUP_TTL_SECONDS` | processor-usage | 사용량 DB 저장 중복 방지 TTL (기본 600초) |

---

## 빌드 & 실행

### 주요 Gradle 태스크 (공통)

| 태스크 | 설명 |
|--------|------|
| `./gradlew bootRun` | 애플리케이션 실행 (`.env` 자동 로딩) |
| `./gradlew bootJar` | 실행 가능한 JAR 빌드 |
| `./gradlew test` | 단위 테스트 + JaCoCo 커버리지 |
| `./gradlew spotlessApply` | Google Java Format 자동 포매팅 |
| `./gradlew checkstyleMain` | Naver Java Convention 검증 |
| `./gradlew sonar` | SonarQube 정적 분석 |

### Docker

```dockerfile
FROM eclipse-temurin:21-jdk AS builder
# ...빌드 단계 (bootJar)...
FROM eclipse-temurin:21-jre
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 기능 명세서

### 사용자 (Customer) — api-core

| 기능명 | 동작 내용 |
|--------|-----------|
| 로그인 | 전화번호 + 비밀번호 → JWT(Access + Refresh) 발급 |
| 토큰 갱신 | Refresh Token → 새 Access Token 발급 |
| 로그아웃 | 토큰 무효화 |
| 회원가입 | 전화번호 기반 사용자 등록 |
| 내 사용량 조회 | 월별 사용량/한도/차단 상태 반환 |
| 내 정책 조회 | 현재 적용된 정책 목록 반환 |

### 가족 (Family) — api-core

| 기능명 | 동작 내용 |
|--------|-----------|
| 대시보드 사용량 | 가족 전체 잔여 데이터, 구성원별 사용 비중, 최근 알림 |
| 사용량 리포트 | 구성원별 상세 사용량 리포트 (과거 월 데이터) |
| 실시간 사용량 (SSE) | `text/event-stream`으로 가족 총 사용량 실시간 Push |
| 구성원별 사용량 | 각 구성원의 실시간 월별 사용량/한도 |
| 가족 초대 | Owner가 전화번호로 구성원 초대 |
| 가족 목록 검색 | Admin 전용, 조건 필터 + 페이징 |
| 가족 상세 조회 | Admin 전용, 구성원/사용량/차단 상태 |
| 구성원 정책 조회/수정 | Owner가 구성원별 정책 조회/변경 → RDS + Redis + Kafka 동시 갱신 |

### 정책 (Policy) — api-core

| 기능명 | 동작 내용 |
|--------|-----------|
| 정책 CRUD | Admin이 정책 템플릿 생성/조회/수정/삭제 |
| 정책 삭제 제약 | 사용 중인 정책은 삭제 불가 (`POLICY_TEMPLATE_IN_USE`) |

### 알림 (Notification) — api-notification

| 기능명 | 동작 내용 |
|--------|-----------|
| 누적 알림 조회 | 전체 알림 이력 (페이징, 읽음 필터) |
| 잔여량 경고 알림 | `THRESHOLD_ALERT` 타입 필터 |
| 차단 알림 | `BLOCKED`, `UNBLOCKED` 타입 필터 |
| SSE 실시간 알림 | Kafka 수신 → SseEmitter 즉시 Push |

### 관리자 (Admin) — api-core

| 기능명 | 동작 내용 |
|--------|-----------|
| 관리자 인증 | 이메일 + 비밀번호 → Admin 전용 JWT 발급 |
| 감사 로그 조회 | 정책 변경, 차단/해제, 권한 변경 이력 |
| 관리자 대시보드 | 전체 가족 수, 활성 사용자, 차단 현황, TPS, 시스템 헬스 |
| 보상 템플릿 관리 | 보상 템플릿 CRUD (목록 조회/상세 조회/생성/수정/삭제) |
| 이미지 업로드 | R2 이미지 업로드 (보상 썸네일/프로필/미션), CDN URL 반환 (5MB 제한, PNG/JPEG/WEBP) |

### 이의신청 (Appeal) — api-core

| 기능명 | 역할 | 엔드포인트 | 상세 |
|--------|------|-----------|------|
| 조르기 요청 | MEMBER | POST `/appeals/appeal` | 한도 증량 이의신청 요청 생성 |
| 긴급 요청 | MEMBER | POST `/appeals/emergency` | 월 1회 제한, 100~300MB, 자동승인 (emergency_grant_month UNIQUE 제약) |
| 이의신청 응답 | OWNER | PUT `/appeals/{id}/respond` | 승인(APPROVED) 또는 거절(REJECTED) |
| 이의신청 코멘트 작성 | ALL | POST `/appeals/{id}/messages` | 이의신청 관련 메시지 작성 |
| 이의신청 코멘트 조회 | ALL | GET `/appeals/{id}/messages` | 이의신청 코멘트 목록 조회 |
| 이의신청 이력 조회 | ALL | GET `/appeals` | 가족 이의신청 이력 목록 조회 |

### 미션/보상 (Mission/Reward) — api-core

| 기능명 | 역할 | 엔드포인트 | 상세 |
|--------|------|-----------|------|
| 미션 생성 | OWNER | POST `/missions` | 미션 카드 생성 (상태: ACTIVE) |
| 미션 삭제 | OWNER | DELETE `/missions/{id}` | 미션 상태 CANCELLED 처리 + MISSION_LOG에 MISSION_CANCELLED 기록 |
| 미션 목록 | ALL | GET `/missions` | 가족 미션 카드 목록 조회 |
| 미션 상태 로그 | ALL | GET `/missions/logs` | 미션 상태 변화 타임라인 (생성/요청/완료/취소) |
| 미션 요청 이력 | ALL | GET `/missions/history` | 보상 요청 처리 결과 이력 (승인/거절) |
| 보상 요청 | MEMBER | POST `/missions/{missionId}/request` | ACTIVE 상태 미션에 대한 보상 요청 |
| 보상 승인/거절 | OWNER | PUT `/rewards/requests/{id}/respond` | 승인 시 MissionItem 상태 COMPLETED 처리 |

### 리포트 (Recap) — api-core

| 기능명 | 역할 | 엔드포인트 | 상세 |
|--------|------|-----------|------|
| 월간 리포트 조회 | ALL | GET `/recaps/monthly` | 월간 가족 데이터 사용 리포트 조회 |
| 배치 집계 | 시스템 | 매월 말 자동 실행 | FAMILY_RECAP_MONTHLY UPSERT, 이의신청지수 계산 포함 |

---

## 관련 문서

- [기획서](./SPECIFICATION.md)
- [아키텍처 설계서](./ARCHITECTURE.md)
- [API 명세서](./API_SPECIFICATION.md)
- [데이터 모델](./DATA_MODEL.md)
- [ERD 설계서](./ERD.md)
- [용어집](./GLOSSARY.md)
- [프론트엔드 명세서](./FRONT_SPECIFICATION.md)
- [api-core 명세서](./designs/api-core/SPECIFICATION.md)
- [processor-usage 명세서](./designs/processor-usage/SPECIFICATION.md)
- [api-notification 명세서](./designs/api-notification/SPECIFICATION.md)
