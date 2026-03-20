# 실시간 가족 데이터 통합 관리 시스템 - API 명세서

> **문서 버전**: v25.0
> **작성일**: 2026-03-20
> **작성자**: DABOM 팀
> **변경 이력**:

> - v25.0 - 미션 목록 조회 응답 dto 수정 - `requestId`필드 추가, 전체 문서 v25.0 Major 버전 동기화
>
> - v24.3 - 응답 상태 코드 정책을 `styleguide.md`와 현재 `ApiResponse` 래퍼 관행에 맞춰 재정렬. 생성 API는 `201 Created`, 본문이 없는 성공 처리도 공개 계약에서는 `200 OK + data: null`을 기본으로 정리. 공개 API 63개 + 내부 테스트 SSE 1개 유지
> - v24.2 - 응답 상태 코드를 "현재 구현"이 아니라 "수정 목표" 기준으로 정리. 생성 API는 `201 Created`, 본문 없는 성공 처리 API도 공통 래퍼 유지 원칙에 맞춰 `200 + data: null` 기준으로 재정렬
> - v24.1 - api-core 커밋 `778d64eef4de4e50f2e46a1c7a6b9ef9881ebf69` 반영. 엔드포인트 추가/삭제는 없지만, 정책 수정/이의제기/미션/보상 처리 시 notification outbox를 통해 알림 이벤트가 발행되는 구현을 문서화하고, `OUTBOX_001` 내부 에러 코드를 추가
> - v24.0 - usage-events 처리 흐름과 notification outbox 구조를 통합 반영: `usage-persist`/`usage-realtime` 제거, processor-usage의 직접 DB 정산 구조로 수정, `usage_event_outbox` 기반 배치 서버 후행 발행 흐름 반영, notification payload를 `subType` 없는 평탄화 형태로 정리. 엔드포인트 총 64개 유지
>
> - v23.3 - FAMILIES/APPEALS 동기화: `GET /families/members` 신규 추가(OWNER 전용, MEMBER 역할 구성원 목록 조회), `POST /appeals` 요청/응답에 `policyActive` 반영, `POST /appeals/emergency` 요청에서 `additionalBytes` 제거 및 서버 고정 300MB 지급 로직 반영, 긴급 승인 시 `CUSTOMER_QUOTA.monthly_limit_bytes`와 `MONTHLY_LIMIT` 정책 할당의 `rules.limitBytes` 동기화. 엔드포인트 총 64→65개
>
> - v23.1 - NOTIFICATIONS 도메인 개선: `GET /notifications` 페이징→커서 기반 무한스크롤 + 30일 제한 + `type` 콤마 구분 다중 필터 + `title` 필드 추가, `GET /notifications/alert`·`GET /notifications/block` 제거, `GET /notifications/unread-count` 신규, `DELETE /notifications/{notificationId}` 신규, 읽음 경로 RESTful 개선 (`/{id}/read`, `/read-all`). PUSH 도메인 신설 (Section 3.11): `GET /push/vapid-public-key`·`POST /push/subscribe`·`DELETE /push/subscribe`·`POST /push/send` 4개 엔드포인트, PUSH 에러 코드 3개 추가. SSE 경로 `GET /families/usage/sse` → `GET /events/stream` + Phase 2 알림 이벤트 7개 추가 (총 12개). 엔드포인트 총 60→64개
>
> - v22.3 - GET `/recaps/monthly`의 mission/appeal summary 집계 기준을 월 내부 full week weekly snapshot 합계 + 좌우 partial raw 보강 구조로 정리하고, `appealSummary`/`appealHighlights`의 승인 집계 기준을 `resolved_at` 월 기준으로 명시. `communicationScore`는 carry-in 포함 처리율/이행률 공식으로 갱신. 엔트포인트 총 60개 유지
>
> - v22.2 - REWARDS 도메인: `GET /admin/rewards/templates/{id}` 상세 조회 엔드포인트 추가, 엔드포인트 총 59→60개
>
> - v22.1 - MISSIONS 도메인 Request/Response를 Spring 구현에 맞춰 수정: `GET /missions` 응답에 `requestStatus`·`createdBy`·페이지네이션 반영, `GET /missions/logs` 리스트 필드명 `logs`→`missions` + `assignedTo` 추가, `GET /missions/history`에서 `assignedTo` 제거 + `requestedAt`·`respondedAt` 추가, `POST /missions` 요청에서 `rewardValue` 제거 + 응답에서 `target` 제거
>
> - v22.0 - UPLOADS 도메인 신규 추가: `POST /uploads/images` (R2 이미지 업로드), thumbnailUrl 필드 CDN 전체 URL 형식으로 일괄 변경 (16건), 에러 코드 UPLOAD_xxx 4개 추가, 엔드포인트 총 58→59개
>
> - v21.6 - 스타일가이드 준수 리팩터링: Response→Responses 상태 코드별 그룹 전환 (56건), 권한 형식 통일 (12건), Path Parameters 테이블 추가 (8건), 비-SSE 엔드포인트 Headers 섹션 제거 (3건), SSE 엔드포인트 경로 수정 (`/families/{familyId}/stream`→`/families/usage/sse`), 3.2.5/3.2.6 SSE 표기 제거 및 3.2.5 응답 ApiResponse 래퍼 추가 (일반 엔드포인트), admin 로그인/리프레시 권한 `admin`→`없음` 수정. 엔드포인트 총 58개 유지
>
> - v21.5 - 미션 요청 이력 조회 엔드포인트 경로 변경: `GET /missions/requests` → `GET /missions/history`, 엔드포인트 총 58개 유지
>
> - v21.4 - 미션 API 역할 분리: `GET /missions/logs`를 순수 `MissionLog` 기반 미션 상태 변화 타임라인으로 재정의 (응답 구조 변경), `GET /missions/history` 신규 추가 (`MissionRequest` 기반 요청 이력 조회, 승인/거절 결과 포함). 기존 `GET /missions/logs` 응답이 `MissionRequest` 기반이던 것을 `MissionLog` 기반으로 변경. 엔드포인트 총 57→58개
>
> - v21.3 - ERD v21.3 동기화: `mission_log.action_type` ENUM 재정의 — `MISSION_APPROVED`·`MISSION_REJECTED` 제거 (요청 처리 결과는 `mission_request.status`가 담당), `MISSION_CANCELLED` 추가 (미션 삭제 로그). 미션 삭제(`DELETE /missions/{missionId}`) 서버 처리 흐름 추가, 엔드포인트 총 57개 유지
>
> - v21.2 - ERD v21.2 동기화: 긴급 쿼터 요청 동시성 문제 해결 — 서버 처리 흐름에서 SELECT 기반 중복 체크를 `emergency_grant_month` UNIQUE 제약 기반 INSERT 중복 방지로 변경, UNIQUE 위반 시 `APPEAL_EMERGENCY_MONTHLY_LIMIT` (429) 에러 반환, 엔드포인트 총 57개 유지
>
> - v21.0 - Figma 디자인 반영: reward 객체에서 defaultValue·value·unit·templateDefaultValue 필드 제거,
>   thumbnailUrl 필드 추가 (S3/R2 이미지 경로),
>   category ENUM 간소화 (DATA/GIFTICON만),
>   지급 내역 API에 phoneNumber 검색·sort 정렬·unusedOnly 필터 추가,
>   템플릿 수정 시 category 변경 불가 반영,
>   예시 데이터 Figma 기준 교체
>
> - v20.0 - ERD v20.0 동기화: REWARD_TEMPLATE에 price·isActive 필드 추가, category ENUM ERD 기준 통일 (MONEY/FOOD→삭제, DATA/GIFTICON/TIME/ETC 정규화), unit ENUM ERD 기준 통일 (원/분/회→MB/MINUTE/COUNT/NONE), REWARD 응답에 templateDefaultValue 추가, REWARD_GRANT 신규 테이블에 따른 admin 보상 지급 내역 조회 엔드포인트 추가, 예시 데이터 현실화, 엔드포인트 총 56→57개
>
> - v19.2 - ERD v19.2 동기화: CUSTOMER `profileImageUrl` 필드 삭제 (GET/PUT /customers/profile), GET /appeals 이의제기 목록 조회에 커서 기반 무한스크롤 적용, 엔드포인트 총 56개 유지
>
> - v19.1 - ERD v19.1 동기화: 이의제기 취소 엔드포인트 추가 (`PATCH /appeals/{appealId}/cancel`), 이의제기 목록 조회 status 필터에 `CANCELLED` 추가, 이의제기 상세 조회 응답에 `policyType` 필드 추가, POLICY_APPEAL.status ENUM에 `CANCELLED` 추가, 에러 코드 3개 추가 (`APPEAL_CANCEL_FORBIDDEN`, `APPEAL_NOT_CANCELLABLE`, `APPEAL_EMERGENCY_CANCEL_NOT_ALLOWED`), 엔드포인트 총 55→56개
>
> - v19.0 - ERD v19.0 동기화: REWARD 엔티티 추가에 따른 API 응답 구조 변경, 모든 미션/보상 응답에서 `rewardTemplate` + `rewardValue` 플랫 구조 → `reward` 중첩 객체로 통합 (GET /missions, GET /missions/logs, POST /missions, POST /missions/{missionId}/request, PUT /rewards/requests/{requestId}/respond, GET /rewards/received), POST /missions 요청에서 `rewardCategory` 필드 제거 (category는 템플릿에서 스냅샷), 엔드포인트 총 55개 유지
>
> - v18.3 - ERD v18.3 동기화: GET /recaps/monthly 응답에 `communicationScore` 추가, `NORMAL` 이의제기와 미션 완료 건수를 기반으로 월간 소통 점수 계산 규칙 명시, `NORMAL` 이의제기 0건이면 미션 완료율로 fallback 하고 이의제기/미션 모두 0건일 때만 `null` 반환
>
> - v18.2 - ERD v18.2 동기화: GET /recaps/monthly 응답의 이의제기 하이라이트 구조를 `appealHighlights`로 재정의 (`topSuccessfulRequester`, `topAcceptedApprover`, 최신 이력 최대 3개), `approverSummary` 제거
>
> - v18.1 - 코드-명세서 동기화: 정책 수정 PATCH→PUT (구현 기준 동기화), 관리자 가족 검색/상세 URL `/families`→`/admin/families` (AdminFamilyController 코드 기준 동기화), ERD v18.1 동기화 (REWARD_TEMPLATE에 updated_at/deleted_at 추가), 엔드포인트 총 55개 유지
>
> - v18.0 - ERD v17.0 동기화: MISSION_ITEM에 target_customer_id 추가 (GET/POST /missions, GET /missions/logs 반영), MISSION_REQUEST에 reject_reason 추가 (reason→rejectReason 변경), GET /rewards/received 신규 엔드포인트 추가, REPORTS→RECAPS 도메인 리네이밍 (GET /reports/monthly→GET /recaps/monthly), GET /recaps/monthly 응답 JSON 구조 통합 (usageByWeekday, peakUsage, missionSummary, appealSummary, approverSummary), POLICY_APPEAL.type ENUM APPEAL→NORMAL, 역할 기반 접근 패턴 문서화, 에러코드 MISSION_NOT_ASSIGNED/MISSION_TARGET_INVALID 추가, 엔드포인트 총 54→55개

> - v17.0 - POLICY_APPEAL type APPEAL→NORMAL 리네이밍, 이의제기 엔드포인트 /policies prefix 제거 (/policies/appeals → /appeals), APPEALS 도메인 독립 분리, admin 보상 템플릿 엔드포인트를 REWARDS 도메인으로 문서 재분류, 엔드포인트 총 54개 유지
> - v16.0 - ERD v16.0 동기화: POLICY_APPEAL_LOG 참조 제거 (긴급 쿼터 처리 흐름에서 삭제), POLICY_ASSIGNMENT reason 필드 제거 (GET /customers/policies, GET /families/policies 응답, PATCH /families/policies 요청에서 삭제), POLICY_APPEAL reason→requestReason/rejectReason 분리 (이의제기 생성/목록/상세/승인거절/긴급 요청 전체 반영), 3.3.6 이의제기 목록 조회 페이징 제거, 3.3.7 이의제기 상세 조회 댓글 커서 기반 무한스크롤 추가, 3.4.1 미션 항목 목록 조회 페이징 제거, 3.4.2 미션 승인/획득 로그 조회 페이징→커서 기반 무한스크롤 변경, 보상 승인 요청을 미션 승인 요청으로 변경하여 MISSIONS 도메인으로 이동 (POST /missions/{missionId}/request), 보상 승인/거절 처리는 REWARDS 도메인 유지 (PUT /rewards/requests/{requestId}/respond), 엔드포인트 총 54개 유지
> - v15.0 - ERD v15.0 동기화: 긴급 쿼터 요청 엔드포인트 추가 (POST /policies/appeals/emergency), GET /policies/appeals에 type 필터 추가, GET /reports/monthly에 emergencyUsedCount 복원, APPEAL_EMERGENCY_xxx 에러 코드 3개 추가, 엔드포인트 총 53→54개
> - v14.0 - ERD v14.0 동기화: POLICY_APPEAL 이의제기 엔드포인트 5개 추가 (목록/상세/생성/승인거절/댓글), desiredRules(nullable) 필드 반영, NEGOTIATION 에러 → APPEAL 에러로 교체, 엔드포인트 총 48→53개; POLICY_ASSIGNMENT reason 필드 추가 (GET /families/policies, GET /customers/policies 응답, PATCH /families/policies 요청), NEGOTIATION 엔드포인트 6개 완전 삭제 (협상 이력/조르기/긴급요청/승인거절/코멘트), negotiationSummary → appealSummary 변경 (emergencyUsedCount 제거), 엔드포인트 총 53개 유지 (본문 negotiation 6개 삭제, 요약 테이블 동기화)
> - v11.0 - 2차기획서 Phase 2 기능 반영: ~30개 신규 엔드포인트 추가 (협상/미션/보상/리포트/알림/프로필), 에러 코드 확장
> - v10.2 - ERD v10.2 동기화: POLICY 테이블 is_activate → is_active 리네이밍, 변경 이력 isActivate → isActive 수정
> - v10.0 - web-core 서브도메인 분리 Major 버전 동기화
> - v9.0 - api-spec 최종 동기화: 도메인 구조 변경 (7→5도메인), 엔드포인트 URL/메서드/응답 구조 변경, isActive 통일, 신규 엔드포인트 추가, 미사용 엔드포인트 제거
> - v8.1 - ERD v8.1 동기화: POLICY 응답에 isActive 필드 추가, 로그인 요청 phoneNumber 숫자만 형식으로 변경
> - v8.0 - 전체 문서 버전 통일 (공유 Major + 독립 Minor 체계 도입), simulator-traffic → simulator-usage 리네이밍 동기화
> - v7.1 - /admin/users 응답 userId→adminId 수정 (ADMIN 테이블 네이밍 통일)
> - v7.0 - api-spec 기반 통일: userId→customerId, members→customers, 응답 구조 개선, POLICY 필드 추가, rules JSON 키 통일
> - v6.0 - ERD v6.0 동기화: /mypage/policies 응답 필드명 통일 (params→rules)
> - v5.0 - ERD v5.0 동기화: daily→monthly 전환, DAILY_LIMIT→MONTHLY_LIMIT, CUSTOMER/ADMIN 분리 반영, POLICY.rules→POLICY_ASSIGNMENT 이동, 관리자 전용 인증 API 추가
> - v4.0 - api-spec.csv 기반 전면 재구성, /api/v1 prefix 제거, 도메인별 그룹핑 (AUTH/FAMILIES/POLICIES/NOTIFICATIONS/MYPAGE/ADMIN), JWT familyId 추론, REST 알림 API 추가

---

## 1. API 개요

### 1.1 Base URL

| 환경      | URL                      |
| --------- | ------------------------ |
| 개발/운영 | `https://api.dabom.site` |

### 1.2 인증 방식

**JWT (JSON Web Token) 자체 구현**

| 토큰 유형     | 용도              | 유효기간 |
| ------------- | ----------------- | -------- |
| Access Token  | API 인증          | 30분     |
| Refresh Token | Access Token 갱신 | 7일      |

**헤더 형식**:

```http
Authorization: Bearer <access_token>
```

**JWT Payload에 포함되는 정보**:

```json
{
  "customerId": 12345,
  "familyId": 100,
  "role": "OWNER",
  "exp": 1705312200
}
```

> **Note (v4.0)**: `familyId`는 JWT 토큰에서 추론합니다. 대부분의 API에서 경로 파라미터로 `familyId`를 전달하지 않습니다.
> **Note**: `role`은 `family_member.role`에서 가져오며, 한 가족 그룹 내에 복수 OWNER가 존재할 수 있습니다.

### 1.3 API 버전 관리

**Accept-Version 헤더 방식**:

```http
Accept-Version: 1.0
```

> **Note (v4.0)**: URL 경로에서 `/api/v1` 프리픽스가 제거되었습니다. API 버전 관리는 Accept-Version 헤더로만 수행합니다.

### 1.4 공통 응답 형식

#### 성공 응답

```json
{
  "success": true,
  "data": { ... },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 에러 응답

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "details": { ... }
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

> **Note**: `dabom-api-notification` 모듈의 에러 응답은 위 래퍼와 다른 형식을 사용한다:
>
> ```json
> {
>   "status": 404,
>   "code": "NOTIFICATION_003",
>   "message": "알림을 찾을 수 없습니다."
> }
> ```

### 1.5 공통 HTTP 상태 코드

| 코드 | 설명                             |
| ---- | -------------------------------- |
| 200  | 성공                             |
| 201  | 생성 성공                        |
| 202  | 요청 수락 (비동기 처리)          |
| 400  | 잘못된 요청                      |
| 401  | 인증 필요                        |
| 403  | 권한 없음                        |
| 404  | 리소스 없음                      |
| 409  | 충돌 (중복 등)                   |
| 429  | 요청 제한 초과                   |
| 500  | 서버 오류                        |
| 503  | 서비스 이용 불가 (Fallback 모드) |

### 1.6 문서 기준 HTTP 상태 코드 원칙

- 리소스 생성: `201 Created`
- 조회/수정/삭제/상태 변경: `200 OK`
- 본문이 실질적으로 없더라도 공통 응답 래퍼를 유지하는 공개 계약은 `200 OK + data: null`을 사용한다.
- `204 No Content`는 문서상 기본 계약으로 채택하지 않는다.
- 업서트 API는 `200 OK`를 기본으로 본다.

### 1.7 권한 모델

| 권한       | 대상                    | 설명                                                                                                                                           |
| ---------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **member** | 모든 인증된 가족 구성원 | 본인 데이터 조회, 알림 수신                                                                                                                    |
| **owner**  | Owner 계정 (복수 가능)  | 가족 내 정책 수정, 구성원 관리. 한 가족에 복수 OWNER 허용. 정책 충돌 시 **Last Write Wins** 적용 (마지막 수정이 유효, `audit_log`에 이력 기록) |
| **admin**  | 백오피스 운영자         | 시스템 전체 관리, 정책 템플릿 CRUD                                                                                                             |

> `familyId`는 JWT 토큰에서 자동으로 추론되므로 대부분의 엔드포인트에서 경로 파라미터로 전달하지 않습니다. 예외: `POST /families/{familyId}/invite`, `GET /families/{familyId}`

---

## 2. Kafka 이벤트 스키마 (simulator-usage → processor-usage)

> **아키텍처 변경 (v2.0)**: HTTP `POST /usage` 엔드포인트가 제거되었습니다.
> simulator-usage가 Kafka로 직접 이벤트를 발행하며, processor-usage가 Consumer로 처리합니다.

### 2.1 데이터 사용 이벤트 (usage-events)

**Kafka Topic**: `usage-events`
**Partition Key**: `familyId` (가족 단위 순서 보장)

#### 이벤트 스키마 (EventEnvelope 패턴)

> 상세 스키마 및 Java Record 구현체는 [Kafka 메시지 스키마](./designs/kafka/MESSAGE_SCHEMA.md) 참조

```json
{
  "eventId": "evt_550e8400-e29b-41d4-a716-446655440000",
  "eventType": "DATA_USAGE",
  "timestamp": "2026-02-06T14:30:00Z",
  "payload": {
    "familyId": 100,
    "customerId": 12345,
    "appId": "com.youtube.app",
    "bytesUsed": 5242880,
    "metadata": {
      "deviceId": "device_pixel_9",
      "networkType": "5G"
    }
  }
}
```

**공통 봉투 (EventEnvelope)**:

| 필드      | 타입   | 필수 | 설명                                                   |
| --------- | ------ | ---- | ------------------------------------------------------ |
| eventId   | string | ✅   | 이벤트 고유 ID (simulator-usage가 UUID v4로 생성)      |
| eventType | string | ✅   | 이벤트 유형 (DATA_USAGE, POLICY_UPDATED, NOTIFICATION) |
| timestamp | string | ✅   | ISO 8601 형식                                          |
| payload   | object | ✅   | eventType에 따라 다른 페이로드                         |

**UsagePayload 필드**:

| 필드       | 타입    | 필수 | 설명                     |
| ---------- | ------- | ---- | ------------------------ |
| familyId   | integer | ✅   | 가족 그룹 ID (파티션 키) |
| customerId | integer | ✅   | 고객(사용자) ID          |
| appId      | string  | ✅   | 앱 식별자                |
| bytesUsed  | integer | ✅   | 사용 바이트 수           |
| metadata   | object  | ❌   | 추가 메타데이터          |

### 2.2 simulator-usage Kafka Producer 설정

```properties
# 버스트 트래픽 처리를 위한 설정
bootstrap.servers=kafka:9092
buffer.memory=67108864        # 64MB 버퍼
batch.size=65536              # 64KB 배치
linger.ms=5                   # 5ms 대기 후 발송
max.block.ms=60000            # 60초 블로킹 허용
acks=1                        # Leader 응답만 대기 (처리량 우선)
compression.type=lz4          # 압축으로 네트워크 효율화
key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer
```

### 2.3 이벤트 발행 예시 (Go)

```go
// simulator-usage에서 Kafka로 직접 발행
event := UsageEvent{
    EventId:   uuid.New().String(),  // simulator-usage가 eventId 생성
    EventType: "DATA_USAGE",
    Timestamp: time.Now().UTC().Format(time.RFC3339),
    FamilyId:  familyId,
    CustomerId: customerId,
    AppId:     appId,
    BytesUsed: bytesUsed,
}

// familyId를 파티션 키로 사용 (가족 단위 순서 보장)
producer.Produce(&kafka.Message{
    TopicPartition: kafka.TopicPartition{Topic: "usage-events"},
    Key:            []byte(strconv.Itoa(event.FamilyId)),
    Value:          eventJson,
})
```

---

## 3. REST API

### 3.1 CUSTOMERS 도메인

#### 3.1.1 로그인

**POST** `/customers/login` | 권한: 없음

**Request**:

```json
{
  "phoneNumber": "01012345678",
  "password": "password123"
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "role": "OWNER"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.1.2 Access Token 재발급 (🔥 2차)

**POST** `/customers/refresh` | 권한: `없음`

**Request**:

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 1800
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.1.3 로그아웃 (🔥 2차)

**POST** `/customers/logout` | 권한: `없음`

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.1.4 회원가입 (✨UPDATED)

**POST** `/customers/signup` | 권한: `없음`

**Request**:

```json
{
  "phoneNumber": "01012345678",
  "password": "password123",
  "name": "홍길동"
}
```

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "id": 23
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.1.5 내 계정 정보 조회 (🌟 New)

**GET** `/customers/me` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "id": 12345,
    "phoneNumber": "010-****-5678",
    "name": "아빠",
    "email": "dad@example.com",
    "isOnboarded": true,
    "termsAgreedAt": "2024-01-01T00:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.1.6 마이페이지 조회 (🌟 New)

**GET** `/customers/mypage` | 권한: `member`

> `customerId`, `familyId`는 JWT 토큰에서 추론

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                 |
| -------- | ------- | ---- | -------------------- |
| year     | integer | ✅   | 조회 연도 (예: 2024) |
| month    | integer | ✅   | 조회 월 (1-12)       |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "name": "홍길동",
    "familyName": "김씨 가족",
    "isBlocked": false,
    "blockReason": null,
    "monthlyLimitBytes": 2147483648,
    "monthlyUsedBytes": 1073741824,
    "timeBlock": null
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

### 3.2 FAMILIES 도메인

#### 3.2.1 가족 그룹 검색

**POST** `/admin/families` | 권한: `admin`

**Request**:

```json
{
  "filters": {
    "name": "김씨",
    "phone": "010",
    "usageRate": 50
  },
  "sort": "createdAt",
  "page": 0,
  "size": 20
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "familyId": 100,
        "familyName": "김씨 가족",
        "customers": [
          {
            "customerId": 12345,
            "name": "아빠",
            "role": "OWNER"
          }
        ],
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 250000,
    "totalPages": 12500
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.2 실시간 가족 사용량

**GET** `/families/usage/current` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "familyId": 100,
    "totalUsedBytes": 53687091200,
    "totalLimitBytes": 107374182400,
    "remainingBytes": 53687091200
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.3 실시간 구성원별 사용량

**GET** `/families/usage/customers` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "customers": [
      {
        "customerId": 12345,
        "name": "아빠",
        "monthlyUsedBytes": 1073741824,
        "monthlyLimitBytes": 5368709120
      },
      {
        "customerId": 12346,
        "name": "자녀1",
        "monthlyUsedBytes": 2147483648,
        "monthlyLimitBytes": 2147483648
      }
    ]
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.4 가족 정보 상세 조회

**GET** `/admin/families/{familyId}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명         |
| -------- | ------- | ------------ |
| familyId | integer | 가족 그룹 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "familyId": 100,
    "familyName": "김씨 가족",
    "createdById": 12345,
    "customers": [
      {
        "customerId": 12345,
        "name": "아빠",
        "phoneNumber": "010-****-5678",
        "role": "OWNER",
        "monthlyLimitBytes": 5368709120,
        "monthlyUsedBytes": 1073741824,
        "isBlocked": false,
        "joinedAt": "2024-01-01T00:00:00Z"
      },
      {
        "customerId": 12346,
        "name": "자녀1",
        "phoneNumber": "010-****-1234",
        "role": "MEMBER",
        "monthlyLimitBytes": 2147483648,
        "monthlyUsedBytes": 2147483648,
        "isBlocked": true,
        "blockReason": "MONTHLY_LIMIT_EXCEEDED",
        "joinedAt": "2024-01-02T00:00:00Z"
      }
    ],
    "totalQuotaBytes": 107374182400,
    "usedBytes": 53687091200,
    "usedPercent": 50.0,
    "currentMonth": "2024-01",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.5 관리자 가족 구성원 수정 (🌟 New)

**PATCH** `/admin/families/{familyId}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명         |
| -------- | ------- | ------------ |
| familyId | integer | 가족 그룹 ID |

**Request**:

```json
{
  "members": [
    {
      "customerId": 12346,
      "role": "MEMBER",
      "monthlyLimitBytes": 3221225472
    }
  ]
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "familyId": 100,
    "updatedCount": 1
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.6 가족 구성원 정책 조회

**GET** `/families/policies` | 권한: `owner`

> `familyId`는 JWT 토큰에서 추론

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "familyId": 100,
    "customers": [
      {
        "customerId": 12346,
        "name": "자녀1",
        "phoneNumber": "010-****-8888",
        "role": "MEMBER",
        "usedBytes": 53687091200,
        "policies": [
          {
            "assignmentId": 1,
            "policyId": 10,
            "policyName": "야간 차단",
            "type": "TIME_BLOCK",
            "isActive": true,
            "rules": {
              "start": "22:00",
              "end": "07:00",
              "timezone": "Asia/Seoul"
            }
          },
          {
            "assignmentId": 2,
            "policyId": 11,
            "policyName": "자녀1 월별 한도",
            "type": "MONTHLY_LIMIT",
            "isActive": true,
            "rules": {
              "limitBytes": 2147483648
            }
          }
        ]
      }
    ]
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.7 가족 구성원 정책 수정 (✨UPDATED)

**PATCH** `/families/policies` | 권한: `owner`

> `familyId`는 JWT 토큰에서 추론. 부분 업데이트(Partial Update) 방식.

**Request**:

```json
{
  "updateInfo": {
    "customerId": 12346,
    "type": "MONTHLY_LIMIT",
    "value": {
      "limitBytes": 3221225472
    }
  }
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "result": {
      "customerId": 12346,
      "type": "MONTHLY_LIMIT",
      "status": "APPLIED"
    }
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.8 가족 구성원 중 자녀 목록 조회 (✨UPDATED)

**GET** `/families/members` | 권한: `owner`

> `familyId`는 JWT 토큰에서 추론.
> OWNER가 자신의 가족에 속한 구성원 중 `role=MEMBER` 인원만 조회합니다. OWNER 계정은 응답에서 제외됩니다.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": [
    {
      "customerId": 12346,
      "name": "자녀1",
      "role": "MEMBER"
    },
    {
      "customerId": 12347,
      "name": "자녀2",
      "role": "MEMBER"
    }
  ],
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.9 가족 이름 수정 (🔥 2차)

**PUT** `/families` | 권한: `owner`

> `familyId`는 JWT 토큰에서 추론

**Request**:

```json
{
  "name": "김씨 가족(수정)"
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "familyId": 100,
    "name": "김씨 가족(수정)",
    "updatedAt": "2024-01-15T10:30:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.10 사용량 통계 대시보드 (🔥 2차)

**GET** `/families/usage/dashboard` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론. 현재 월 기준 통계를 제공합니다.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "familyId": 100,
    "familyName": "김씨 가족",
    "currentMonth": "2024-01",
    "totalQuotaBytes": 107374182400,
    "totalUsedBytes": 53687091200,
    "totalRemainingBytes": 53687091200,
    "usedPercent": 50.0,
    "memberStats": [
      {
        "customerId": 12345,
        "name": "아빠",
        "role": "OWNER",
        "monthlyUsedBytes": 1073741824,
        "monthlyLimitBytes": 5368709120,
        "usedPercent": 20.0,
        "isBlocked": false
      },
      {
        "customerId": 12346,
        "name": "자녀1",
        "role": "MEMBER",
        "monthlyUsedBytes": 2147483648,
        "monthlyLimitBytes": 2147483648,
        "usedPercent": 100.0,
        "isBlocked": true,
        "blockReason": "MONTHLY_LIMIT_EXCEEDED"
      }
    ]
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.2.11 보상 템플릿 목록 조회 (🌟 New)

**GET** `/rewards/templates` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론. 가족 구성원이 조회 가능한 보상 템플릿 목록

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "메가커피 아메리카노(ICE)",
      "category": "GIFTICON",
      "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
      "price": 3000,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z"
    },
    {
      "id": 3,
      "name": "100MB",
      "category": "DATA",
      "thumbnailUrl": null,
      "price": 3000,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

### 3.3 POLICIES 도메인

#### 3.3.1 정책 조회

**GET** `/policies` | 권한: `admin`

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                                                                |
| -------- | ------- | ---- | ------------------------------------------------------------------- |
| type     | string  | ❌   | 정책 유형 필터 (MONTHLY_LIMIT, TIME_BLOCK, MANUAL_BLOCK, APP_BLOCK) |
| page     | integer | ❌   | 페이지 번호 (기본: 0)                                               |
| size     | integer | ❌   | 페이지 크기 (기본: 10)                                              |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "policies": [
      {
        "policyId": 10,
        "name": "야간 차단 기본",
        "type": "TIME_BLOCK",
        "defaultRules": {
          "start": "22:00",
          "end": "07:00",
          "timezone": "Asia/Seoul"
        },
        "requireRole": "OWNER",
        "isActive": true,
        "isSystem": false,
        "createdAt": "2024-01-01T00:00:00Z",
        "updatedAt": "2024-01-01T00:00:00Z"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 15,
    "totalPages": 2
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.3.2 정책 생성

**POST** `/policies` | 권한: `admin`

**Request**:

```json
{
  "name": "학습 시간대 차단",
  "type": "TIME_BLOCK"
}
```

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "policyId": 16,
    "name": "학습 시간대 차단",
    "type": "TIME_BLOCK",
    "isSystem": false,
    "createdAt": "2024-01-15T10:30:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.3.3 정책 삭제

**DELETE** `/policies/{policyId}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명    |
| -------- | ------- | ------- |
| policyId | integer | 정책 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "policyId": 16,
    "deletedAt": "2024-01-15T10:30:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.3.4 정책 상세 조회

**GET** `/policies/{policyId}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명    |
| -------- | ------- | ------- |
| policyId | integer | 정책 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "policyId": 10,
    "name": "야간 차단 기본",
    "description": "정책 상세 설명",
    "type": "TIME_BLOCK",
    "defaultRules": {
      "start": "22:00",
      "end": "07:00",
      "timezone": "Asia/Seoul"
    },
    "requireRole": "OWNER",
    "isActive": true,
    "isSystem": false,
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.3.5 정책 수정

**PUT** `/policies/{policyId}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명    |
| -------- | ------- | ------- |
| policyId | integer | 정책 ID |

**Request**:

```json
{
  "description": "정책 상세 정보.....",
  "requireRole": "OWNER",
  "defaultRules": {
    "start": "22:00",
    "end": "07:00",
    "timezone": "Asia/Seoul"
  },
  "isActive": true,
  "overWrite": false
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "policyId": 16,
    "updatedAt": "2024-01-15T10:30:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

### 3.4 APPEALS 도메인 (✨UPDATED)

> 자녀의 정책 이의제기 및 긴급 쿼터 요청을 관리합니다.

#### 3.4.1 이의제기 가능 정책 목록 조회 (🌟 New)

**GET** `/appeals/policies` | 권한: `member`

> `customerId`, `familyId`는 JWT 토큰에서 추론. 이의제기를 생성할 수 있는 정책 목록을 조회한다.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": [
    {
      "assignmentId": 2,
      "policyId": 11,
      "policyName": "자녀1 월별 한도",
      "type": "MONTHLY_LIMIT",
      "isActive": true,
      "rules": {
        "limitBytes": 2147483648
      }
    }
  ],
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.4.2 이의제기 목록 조회 (✨UPDATED)

**GET** `/appeals` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론. MEMBER는 본인 이의제기만, OWNER는 가족 전체 이의제기 조회.
> 커서 기반 무한스크롤로 제공.

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                                               |
| -------- | ------- | ---- | -------------------------------------------------- |
| status   | string  | ❌   | 상태 필터 (PENDING, APPROVED, REJECTED, CANCELLED) |
| cursor   | string  | ❌   | 다음 페이지 커서 (이전 응답의 `nextCursor` 값)     |
| size     | integer | ❌   | 조회 크기 (기본: 20)                               |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "appeals": [
      {
        "appealId": 1,
        "type": "NORMAL",
        "policyAssignmentId": 55,
        "requesterId": 12346,
        "requesterName": "자녀1",
        "requestReason": "인강을 들어야 합니다",
        "desiredRules": { "limitBytes": 524288000 },
        "status": "PENDING",
        "createdAt": "2024-01-15T10:30:00Z"
      }
    ],
    "nextCursor": "eyJhcHBlYWxJZCI6MX0=",
    "hasNext": true
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

> **무한스크롤 동작**: `nextCursor`가 null이면 더 이상 로드할 이의제기가 없음. 프론트엔드에서 스크롤 끝에 도달하면 `cursor` 파라미터로 다음 페이지를 요청.

#### 3.4.3 이의제기 상세 조회 (✨UPDATED)

**GET** `/appeals/{appealId}` | 권한: `member`

> 댓글은 커서 기반 무한스크롤로 제공. 최초 조회 시 최신 댓글 20개를 반환하며, `nextCursor`를 사용하여 이전 댓글을 추가 로딩.
> `policyType`: `policyAssignmentId`로 연결된 정책의 타입을 내려줌. EMERGENCY 타입(policy_assignment_id=NULL)이면 `null`.

**Path Parameters**:

| 파라미터 | 타입    | 설명        |
| -------- | ------- | ----------- |
| appealId | integer | 이의제기 ID |

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                                           |
| -------- | ------- | ---- | ---------------------------------------------- |
| cursor   | string  | ❌   | 다음 페이지 커서 (이전 응답의 `nextCursor` 값) |
| size     | integer | ❌   | 댓글 조회 크기 (기본: 20)                      |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "appealId": 1,
    "policyAssignmentId": 55,
    "policyType": "MONTHLY_LIMIT",
    "requesterId": 12346,
    "requesterName": "자녀1",
    "requestReason": "인강을 들어야 합니다",
    "rejectReason": null,
    "desiredRules": { "limitBytes": 524288000 },
    "status": "PENDING",
    "resolvedById": null,
    "resolvedAt": null,
    "createdAt": "2024-01-15T10:30:00Z",
    "comments": {
      "content": [
        {
          "commentId": 10,
          "authorId": 12346,
          "authorName": "자녀1",
          "comment": "공부할 때 필요해요",
          "createdAt": "2024-01-15T10:35:00Z"
        }
      ],
      "nextCursor": "eyJjb21tZW50SWQiOjEwfQ==",
      "hasNext": true
    }
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**`policyType` 설명**:

| policyType      | 설명                            |
| --------------- | ------------------------------- |
| `MONTHLY_LIMIT` | 월간 데이터 제한 정책           |
| `TIME_BLOCK`    | 시간대 차단 정책                |
| `APP_BLOCK`     | 앱 차단 정책                    |
| `MANUAL_BLOCK`  | 수동 차단 정책                  |
| `null`          | EMERGENCY 타입 (정책 연결 없음) |

> **무한스크롤 동작**: `nextCursor`가 null이면 더 이상 로드할 댓글이 없음. 프론트엔드에서 스크롤 끝에 도달하면 `cursor` 파라미터로 다음 페이지를 요청.

#### 3.4.4 이의제기 생성 (✨UPDATED)

**POST** `/appeals` | 권한: `member (MEMBER)`

**Request**:

```json
{
  "policyAssignmentId": 55,
  "requestReason": "인강을 들어야 합니다",
  "policyActive": true,
  "desiredRules": { "limitBytes": 524288000 }
}
```

> `desiredRules`는 optional. NULL이면 부모가 직접 정책 수정, 값이 있으면 APPROVED 시 PolicyAssignment에 자동 반영.
> `desiredRules` 스키마는 해당 policy.type의 rules 스키마와 동일.
> `policyActive`는 필수입니다. 승인 시 정책 활성화 상태를 함께 변경할 수 있습니다.

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "appealId": 1,
    "policyAssignmentId": 55,
    "status": "PENDING",
    "policyActive": true,
    "desiredRules": { "limitBytes": 524288000 },
    "createdAt": "2024-01-15T10:30:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.4.5 이의제기 승인/거절 (✨UPDATED)

**PATCH** `/appeals/{appealId}/respond` | 권한: `owner`

**Path Parameters**:

| 파라미터 | 타입    | 설명        |
| -------- | ------- | ----------- |
| appealId | integer | 이의제기 ID |

**Request (승인)**:

```json
{
  "action": "APPROVED"
}
```

**Request (거절 — 사유 포함)**:

```json
{
  "action": "REJECTED",
  "rejectReason": "이번 달 데이터 사용량이 이미 많아서 추가 완화가 어렵습니다."
}
```

> `action`: `APPROVED` 또는 `REJECTED` > `rejectReason`: 거절 시 사유 (optional). 승인 시에는 생략 가능.
> `desiredRules`가 있는 이의제기를 `APPROVED` 처리 시, PolicyAssignment.rules에 `desiredRules` 값이 자동 반영됨.
> `desiredRules`가 NULL인 경우 부모가 별도로 정책을 직접 수정해야 함.

**Responses**:

**200 OK (승인)**:

```json
{
  "success": true,
  "data": {
    "appealId": 1,
    "status": "APPROVED",
    "resolvedById": 12345,
    "resolvedAt": "2024-01-15T11:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**200 OK (거절)**:

```json
{
  "success": true,
  "data": {
    "appealId": 1,
    "status": "REJECTED",
    "rejectReason": "이번 달 데이터 사용량이 이미 많아서 추가 완화가 어렵습니다.",
    "resolvedById": 12345,
    "resolvedAt": "2024-01-15T11:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.4.6 이의제기 댓글 작성 (✨UPDATED)

**POST** `/appeals/{appealId}/comments` | 권한: `member`

**Path Parameters**:

| 파라미터 | 타입    | 설명        |
| -------- | ------- | ----------- |
| appealId | integer | 이의제기 ID |

**Request**:

```json
{
  "comment": "부모/자녀 모두 작성 가능한 댓글입니다."
}
```

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "commentId": 10,
    "appealId": 1,
    "authorId": 12345,
    "comment": "부모/자녀 모두 작성 가능한 댓글입니다.",
    "createdAt": "2024-01-15T11:05:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.4.7 긴급 쿼터 요청 (✨UPDATED)

**POST** `/appeals/emergency` | 권한: `member (MEMBER)`

> 자녀가 월 1회, 부모 승인 없이 사유만 남기고 데이터를 받을 수 있는 긴급 쿼터 요청입니다.
> 서버가 고정으로 300MB(`314,572,800 bytes`)를 즉시 승인하며, 승인 후 `CUSTOMER_QUOTA.monthly_limit_bytes`와 `MONTHLY_LIMIT` 정책 할당의 `rules.limitBytes`를 함께 동기화합니다.

**제한 사항**:

- 월 1회만 가능 (`emergency_grant_month` UNIQUE 제약으로 DB 레벨 동시성 안전 중복 방지)
- 무제한 쿼터 사용자(`monthly_limit_bytes=NULL`)는 요청 불가

**Request**:

```json
{
  "requestReason": "급하게 과제 제출해야 합니다"
}
```

| 필드          | 타입   | 필수 | 설명           |
| ------------- | ------ | ---- | -------------- |
| requestReason | string | ✅   | 긴급 요청 사유 |

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "appealId": 42,
    "type": "EMERGENCY",
    "status": "APPROVED",
    "additionalBytes": 314572800,
    "newMonthlyLimitBytes": 2357198848,
    "requestReason": "급하게 과제 제출해야 합니다",
    "createdAt": "2026-03-04T14:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**서버 처리 흐름**:

1. JWT에서 `customerId` 추출, MEMBER 역할 검증
2. `CUSTOMER_QUOTA` 조회 → `monthly_limit_bytes`가 NULL(무제한)이면 에러
3. `POLICY_APPEAL` INSERT (`type=EMERGENCY`, `status=APPROVED`, `resolved_by_id=NULL`, `emergency_grant_month=현재 월 1일`, `desired_rules.additionalBytes=314572800`)
   - `uk_appeal_emergency_month` UNIQUE 제약 위반 시 → `APPEAL_EMERGENCY_MONTHLY_LIMIT` (429) 에러 반환 (동시성 안전)
4. `CUSTOMER_QUOTA.monthly_limit_bytes += 314572800`
5. 대상 고객의 `MONTHLY_LIMIT` 정책 할당이 있으면 `rules.limitBytes`를 증가된 월 한도로 동기화
6. NOTIFICATION 발행 → 부모(OWNER)에게 `EMERGENCY_APPROVED` 사후 알림
7. 201 응답 반환

#### 3.4.8 이의제기 취소 (🌟 New)

**PATCH** `/appeals/{appealId}/cancel` | 권한: `member (MEMBER)`

> 요청자 본인이 생성한 일반(NORMAL) 이의제기만 취소 가능.
> `PENDING` 상태에서만 취소 가능. `EMERGENCY` 타입은 취소 불가.
> 이미 `APPROVED`, `REJECTED`, `CANCELLED` 상태인 건 취소 불가.

**Path Parameters**:

| 파라미터 | 타입    | 설명        |
| -------- | ------- | ----------- |
| appealId | integer | 이의제기 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "appealId": 1,
    "status": "CANCELLED",
    "cancelledAt": "2024-01-15T10:45:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**에러 코드**:

| 코드                                  | HTTP | 설명                                               |
| ------------------------------------- | ---- | -------------------------------------------------- |
| `APPEAL_NOT_FOUND`                    | 404  | 이의제기를 찾을 수 없음                            |
| `APPEAL_CANCEL_FORBIDDEN`             | 403  | 본인이 생성한 이의제기가 아님                      |
| `APPEAL_NOT_CANCELLABLE`              | 409  | PENDING 상태가 아니라 취소 불가 (이미 처리/취소됨) |
| `APPEAL_EMERGENCY_CANCEL_NOT_ALLOWED` | 400  | EMERGENCY 타입은 취소 불가                         |

---

### 3.5 MISSIONS 도메인 (🌟 New)

> 미션 및 보상 기능을 제공합니다. OWNER가 미션을 생성하고, MEMBER가 완료 후 보상 요청을 합니다.

#### 3.5.1 미션 항목 목록 조회 (카드형) (✨UPDATED)

**GET** `/missions` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론
> OWNER는 가족 전체 미션, MEMBER는 본인(target) 미션만 조회.

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                                           |
| -------- | ------- | ---- | ---------------------------------------------- |
| cursor   | string  | ❌   | 다음 페이지 커서 (이전 응답의 `nextCursor` 값) |
| size     | integer | ❌   | 조회 크기 (기본: 20)                           |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "missions": [
      {
        "missionItemId": 201,
        "requestId": 55,
        "missionText": "방 청소하기",
        "requestStatus": null,
        "target": {
          "customerId": 12346,
          "name": "자녀1"
        },
        "createdBy": {
          "customerId": 12345,
          "name": "아빠"
        },
        "reward": {
          "rewardId": 1,
          "name": "메가커피 아메리카노(ICE)",
          "category": "GIFTICON",
          "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
          "templateId": 1
        },
        "createdAt": "2024-01-15T09:00:00Z"
      }
    ],
    "nextCursor": "eyJtaXNzaW9uSXRlbUlkIjoyMDF9",
    "hasNext": true
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.5.2 미션 상태 변화 로그 조회 (✨UPDATED v21.4)

**GET** `/missions/logs` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론. 커서 기반 무한스크롤로 제공.
> OWNER는 가족 전체 로그, MEMBER는 본인 target 미션 로그만 조회.
> `MissionLog` 기반 — 미션 자체의 상태 변화 타임라인만 제공. 요청 처리 결과(승인/거절)는 `GET /missions/history`에서 조회.

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                                           |
| -------- | ------- | ---- | ---------------------------------------------- |
| cursor   | string  | ❌   | 다음 페이지 커서 (이전 응답의 `nextCursor` 값) |
| size     | integer | ❌   | 조회 크기 (기본: 20)                           |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "missions": [
      {
        "logId": 501,
        "actionType": "MISSION_COMPLETED",
        "message": "미션이 완료되었습니다",
        "missionItem": {
          "missionItemId": 201,
          "missionText": "방 청소하기",
          "reward": {
            "rewardId": 1,
            "name": "메가커피 아메리카노(ICE)",
            "category": "GIFTICON",
            "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
            "templateId": 1
          }
        },
        "assignedTo": {
          "customerId": 12346,
          "name": "자녀1"
        },
        "actor": {
          "customerId": null,
          "name": null
        },
        "createdAt": "2024-01-15T12:05:00Z"
      },
      {
        "logId": 500,
        "actionType": "MISSION_REQUESTED",
        "message": "보상 요청이 등록되었습니다",
        "missionItem": {
          "missionItemId": 201,
          "missionText": "방 청소하기",
          "reward": {
            "rewardId": 1,
            "name": "메가커피 아메리카노(ICE)",
            "category": "GIFTICON",
            "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
            "templateId": 1
          }
        },
        "assignedTo": {
          "customerId": 12346,
          "name": "자녀1"
        },
        "actor": {
          "customerId": 12346,
          "name": "자녀1"
        },
        "createdAt": "2024-01-15T12:00:00Z"
      }
    ],
    "nextCursor": "eyJsb2dJZCI6NTAwfQ==",
    "hasNext": true
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**`actionType` 값**:

| actionType          | 설명                                |
| ------------------- | ----------------------------------- |
| `MISSION_CREATED`   | 미션 생성 (부모)                    |
| `MISSION_REQUESTED` | 보상 요청 (자녀)                    |
| `MISSION_COMPLETED` | 미션 완료 처리 (시스템, actor=null) |
| `MISSION_CANCELLED` | 미션 삭제/취소 (부모)               |

#### 3.5.3 미션 요청 이력 조회 (🌟 New v21.4)

**GET** `/missions/history` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론. 커서 기반 무한스크롤로 제공.
> OWNER는 가족 전체 요청 이력, MEMBER는 본인이 요청한 이력만 조회.
> `MissionRequest` 기반 — 보상 요청의 처리 결과(승인/거절)를 조회할 때 사용.

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                                           |
| -------- | ------- | ---- | ---------------------------------------------- |
| status   | string  | ❌   | 필터: `PENDING`, `APPROVED`, `REJECTED`        |
| cursor   | string  | ❌   | 다음 페이지 커서 (이전 응답의 `nextCursor` 값) |
| size     | integer | ❌   | 조회 크기 (기본: 20)                           |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "requests": [
      {
        "requestId": 301,
        "status": "APPROVED",
        "missionItem": {
          "missionItemId": 201,
          "missionText": "방 청소하기",
          "reward": {
            "rewardId": 1,
            "name": "메가커피 아메리카노(ICE)",
            "category": "GIFTICON",
            "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
            "templateId": 1
          }
        },
        "requestedBy": {
          "customerId": 12346,
          "name": "자녀1"
        },
        "respondedBy": {
          "customerId": 12345,
          "name": "아빠"
        },
        "rejectReason": null,
        "requestedAt": "2024-01-15T12:00:00Z",
        "respondedAt": "2024-01-15T12:05:00Z",
        "createdAt": "2024-01-15T12:00:00Z",
        "updatedAt": "2024-01-15T12:05:00Z"
      }
    ],
    "nextCursor": "eyJyZXF1ZXN0SWQiOjMwMX0=",
    "hasNext": true
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

> **무한스크롤 동작**: `nextCursor`가 null이면 더 이상 로드할 로그가 없음. 프론트엔드에서 스크롤 끝에 도달하면 `cursor` 파라미터로 다음 페이지를 요청.

#### 3.5.4 미션 + 템플릿 기반 보상 생성

**POST** `/missions` | 권한: `owner`

> `familyId`, `customerId`는 JWT 토큰에서 추론

**Request**:

```json
{
  "targetCustomerId": 12346,
  "missionText": "방 청소하기",
  "rewardTemplateId": 1
}
```

> `targetCustomerId`는 필수. MEMBER 역할인 가족 구성원만 대상 가능 (OWNER 지정 시 `MISSION_TARGET_INVALID` 400).
> 동일 미션을 여러 자녀에게 부여하려면 각 자녀별로 별도 생성.

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "missionItemId": 202,
    "createdAt": "2024-01-15T13:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.5.5 미션 항목 삭제

**DELETE** `/missions/{missionId}` | 권한: `owner`

> `familyId`는 JWT 토큰에서 추론. 미션을 삭제하지 않고 status를 CANCELLED로 변경합니다.
> `MISSION_LOG`에 `action_type=MISSION_CANCELLED`, `actor_id=요청자(부모)` 로그를 기록합니다.

**Path Parameters**:

| 파라미터  | 타입    | 설명         |
| --------- | ------- | ------------ |
| missionId | integer | 미션 항목 ID |

**서버 처리 흐름**:

1. JWT에서 `customerId` 추출, OWNER 역할 검증
2. `MISSION_ITEM` 조회 → `status=ACTIVE` 검증
3. `MISSION_ITEM.status = CANCELLED` 업데이트
4. `MISSION_LOG` INSERT (`action_type=MISSION_CANCELLED`, `actor_id=customerId`, `message=미션이 삭제되었습니다`)
5. 200 응답 반환

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.5.6 미션 승인 요청 (✨UPDATED)

**POST** `/missions/{missionId}/request` | 권한: `member (MEMBER)`

> `customerId`, `familyId`는 JWT 토큰에서 추론
> 조건: MISSION_ITEM.status == ACTIVE
> 비즈니스 규칙: 요청자(JWT customerId) == MISSION_ITEM.target_customer_id 검증. 본인에게 배정된 미션만 요청 가능 (불일치 시 `MISSION_NOT_ASSIGNED` 403).

**Path Parameters**:

| 파라미터  | 타입    | 설명         |
| --------- | ------- | ------------ |
| missionId | integer | 미션 항목 ID |

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "requestId": 302,
    "missionItem": {
      "missionItemId": 201,
      "missionText": "방 청소하기",
      "reward": {
        "rewardId": 1,
        "name": "메가커피 아메리카노(ICE)",
        "category": "GIFTICON",
        "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
        "templateId": 1
      }
    },
    "status": "PENDING",
    "requestedBy": {
      "customerId": 12346,
      "name": "자녀1"
    },
    "createdAt": "2024-01-15T14:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

### 3.6 REWARDS 도메인 (🌟 New)

> 미션 완료 후 보상 승인/거절 처리를 담당합니다.

#### 3.6.1 보상 승인/거절 처리

**PUT** `/rewards/requests/{requestId}/respond` | 권한: `owner`

> `familyId`는 JWT 토큰에서 추론
> 부수효과: APPROVED 시 MISSION_ITEM.status가 COMPLETED로 변경됩니다.
> 비즈니스 규칙: status=APPROVED이면서 rejectReason이 있으면 400 에러 반환.

**Path Parameters**:

| 파라미터  | 타입    | 설명         |
| --------- | ------- | ------------ |
| requestId | integer | 보상 요청 ID |

**Request**:

```json
{
  "status": "APPROVED",
  "rejectReason": null
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "requestId": 302,
    "status": "APPROVED",
    "missionItem": {
      "missionItemId": 201,
      "missionText": "방 청소하기",
      "status": "COMPLETED",
      "reward": {
        "rewardId": 1,
        "name": "메가커피 아메리카노(ICE)",
        "category": "GIFTICON",
        "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
        "templateId": 1
      }
    },
    "respondedBy": {
      "customerId": 12345,
      "name": "아빠"
    },
    "rejectReason": null,
    "updatedAt": "2024-01-15T14:05:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.6.2 내가 받은 보상 조회 (🌟 New)

**GET** `/rewards/received` | 권한: `member (MEMBER)`

> `customerId`는 JWT 토큰에서 추론. 본인이 target인 미션의 APPROVED 보상 내역만 조회.
> OWNER는 사용 불가 (403). OWNER는 `GET /missions/logs`로 전체 보상 현황 확인 가능.

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                 |
| -------- | ------- | ---- | -------------------- |
| cursor   | string  | ❌   | 커서                 |
| size     | integer | ❌   | 조회 크기 (기본: 20) |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "rewards": [
      {
        "requestId": 302,
        "missionItem": {
          "missionItemId": 201,
          "missionText": "방 청소하기",
          "reward": {
            "rewardId": 1,
            "name": "메가커피 아메리카노(ICE)",
            "category": "GIFTICON",
            "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
            "templateId": 1
          }
        },
        "approvedBy": {
          "customerId": 12345,
          "name": "아빠"
        },
        "approvedAt": "2024-01-15T14:05:00Z"
      }
    ],
    "nextCursor": "eyJyZXF1ZXN0SWQiOjMwMn0=",
    "hasNext": true
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

> **내부 쿼리**: MEMBER 전용. `MISSION_REQUEST.status = 'APPROVED'` AND `MISSION_ITEM.target_customer_id = JWT.customerId` 조건으로 조회.

#### 3.6.3 보상 템플릿 목록 조회 (🌟 New)

**GET** `/admin/rewards/templates` | 권한: `admin`

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "메가커피 아메리카노(ICE)",
      "category": "GIFTICON",
      "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
      "price": 3000,
      "isSystem": true,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-01T00:00:00Z"
    }
  ],
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.6.4 보상 템플릿 상세 조회 (🌟 New)

**GET** `/admin/rewards/templates/{id}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명           |
| -------- | ------- | -------------- |
| id       | integer | 보상 템플릿 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "메가커피 아메리카노(ICE)",
    "category": "GIFTICON",
    "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
    "price": 3000,
    "isSystem": true,
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.6.5 보상 템플릿 생성 (🌟 New)

**POST** `/admin/rewards/templates` | 권한: `admin`

**Request**:

```json
{
  "name": "메가커피 아메리카노(ICE)",
  "category": "GIFTICON",
  "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
  "price": 3000
}
```

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "id": 3,
    "name": "메가커피 아메리카노(ICE)",
    "category": "GIFTICON",
    "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg",
    "price": 3000,
    "isSystem": false,
    "isActive": true,
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.6.6 보상 템플릿 수정 (🌟 New)

**PUT** `/admin/rewards/templates/{id}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명           |
| -------- | ------- | -------------- |
| id       | integer | 보상 템플릿 ID |

**Request**:

```json
{
  "name": "메가커피 아메리카노(ICE)(수정)",
  "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee-v2.jpg",
  "price": 3500,
  "isActive": true
}
```

> **Note (v21.0)**: `category`는 템플릿 생성 시 결정되며, 수정 시 변경할 수 없습니다 (Figma: "유형은 변경할 수 없습니다").

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "메가커피 아메리카노(ICE)(수정)",
    "category": "GIFTICON",
    "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee-v2.jpg",
    "price": 3500,
    "isSystem": true,
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-15T10:35:00Z"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.6.7 보상 템플릿 삭제 (🌟 New)

**DELETE** `/admin/rewards/templates/{id}` | 권한: `admin`

**Path Parameters**:

| 파라미터 | 타입    | 설명           |
| -------- | ------- | -------------- |
| id       | integer | 보상 템플릿 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.6.8 보상 지급 내역 조회 (🌟 New)

**GET** `/admin/rewards/grants` | 권한: `admin`

> 보상 지급 이력을 조회합니다. MISSION_REQUEST 승인 시 자동 생성된 지급 기록을 관리합니다.

**Query Parameters**:

| 파라미터    | 타입    | 필수 | 설명                                         |
| ----------- | ------- | ---- | -------------------------------------------- |
| page        | integer | ❌   | 페이지 번호 (기본: 0)                        |
| size        | integer | ❌   | 페이지 크기 (기본: 20)                       |
| status      | string  | ❌   | 상태 필터 (ISSUED, USED, EXPIRED)            |
| sort        | string  | ❌   | 정렬 (LATEST \| EXPIRING_SOON, 기본: LATEST) |
| unusedOnly  | boolean | ❌   | true이면 status=ISSUED만 조회                |
| phoneNumber | string  | ❌   | 전화번호 검색                                |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "grantId": 1,
        "reward": {
          "rewardId": 1,
          "name": "메가커피 아메리카노(ICE)",
          "category": "GIFTICON",
          "thumbnailUrl": "https://cdn.dabom.site/rewards/mega-coffee.jpg"
        },
        "customer": {
          "customerId": 12346,
          "name": "자녀1",
          "phoneNumber": "010-****-1234"
        },
        "mission": {
          "missionItemId": 201,
          "missionText": "방 청소하기"
        },
        "couponCode": "MEGA-1234-5678",
        "couponUrl": "https://coupon.example.com/MEGA-1234-5678",
        "status": "ISSUED",
        "expiredAt": "2024-02-15T23:59:59Z",
        "createdAt": "2024-01-15T14:05:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

### 3.7 RECAPS 도메인 (✨UPDATED)

> 월간 가족 활동 리캡을 제공합니다. ERD의 FAMILY_RECAP_MONTHLY 테이블과 1:1 매핑됩니다.

#### 3.7.1 월간 가족 리캡 조회

**GET** `/recaps/monthly` | 권한: `member`

> `familyId`는 JWT 토큰에서 추론

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                 |
| -------- | ------- | ---- | -------------------- |
| year     | integer | ✅   | 조회 연도 (예: 2024) |
| month    | integer | ✅   | 조회 월 (1-12)       |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "recapId": 401,
    "familyId": 100,
    "familyName": "김씨 가족",
    "reportMonth": "2026-03-01",
    "totalUsedBytes": 53687091200,
    "totalQuotaBytes": 107374182400,
    "usageRatePercent": 50.0,

    "usageByWeekday": {
      "monday": 15.2,
      "tuesday": 18.5,
      "wednesday": 22.1,
      "thursday": 0.0,
      "friday": 0.0,
      "saturday": 20.3,
      "sunday": 9.9
    },

    "peakUsage": {
      "startHour": 21,
      "endHour": 23,
      "mostUsedWeekday": "sunday"
    },

    "missionSummary": {
      "totalMissionCount": 10,
      "completedMissionCount": 5,
      "rejectedRequestCount": 3
    },

    "appealSummary": {
      "totalAppeals": 4,
      "approvedAppeals": 3,
      "rejectedAppeals": 1
    },

    "appealHighlights": {
      "topSuccessfulRequester": {
        "requesterId": 12346,
        "requesterName": "김민지",
        "approvedAppealCount": 3,
        "recentApprovedAppeals": [
          {
            "appealId": 91,
            "approverId": 12345,
            "approverName": "김철수",
            "requestReason": "야간 차단 해제를 요청했어요.",
            "requestedAt": "2026-03-21T14:32:00"
          },
          {
            "appealId": 87,
            "approverId": 12345,
            "approverName": "김철수",
            "requestReason": "주말 사용 제한 완화를 요청했어요.",
            "requestedAt": "2026-03-18T20:10:00"
          },
          {
            "appealId": 83,
            "approverId": 12345,
            "approverName": "김철수",
            "requestReason": "인강 시청 시간 연장을 요청했어요.",
            "requestedAt": "2026-03-12T19:05:00"
          }
        ]
      },
      "topAcceptedApprover": {
        "approverId": 12345,
        "approverName": "김철수",
        "approvedAppealCount": 3,
        "recentAcceptedAppeals": [
          {
            "appealId": 91,
            "requesterId": 12346,
            "requesterName": "김민지",
            "requestReason": "야간 차단 해제를 요청했어요.",
            "resolvedAt": "2026-03-21T14:32:00"
          },
          {
            "appealId": 87,
            "requesterId": 12346,
            "requesterName": "김민지",
            "requestReason": "주말 사용 제한 완화를 요청했어요.",
            "resolvedAt": "2026-03-18T20:10:00"
          },
          {
            "appealId": 83,
            "requesterId": 12347,
            "requesterName": "김민수",
            "requestReason": "인강 시청 시간 연장을 요청했어요.",
            "resolvedAt": "2026-03-12T19:05:00"
          }
        ]
      }
    },

    "communicationScore": 82.5,
    "generatedAt": "2026-03-01T00:00:00"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

> `missionSummary.totalMissionCount`는 월간 생성된 미션 수입니다. `completedMissionCount`와 `rejectedRequestCount`는 월간 완료 처리 또는 거절 처리된 이벤트 수입니다.
> `appealSummary.totalAppeals`는 월간 생성된 `type='NORMAL'` 이의제기 수입니다. `approvedAppeals`와 `rejectedAppeals`는 월간 승인 처리 또는 거절 처리된 `type='NORMAL'` 이의제기 수입니다.
> `appealHighlights`는 `type='NORMAL' AND status='APPROVED'`이면서 `resolved_at`이 월 구간에 포함된 정책 이의제기만 집계합니다. `EMERGENCY`(긴급 요청)는 포함하지 않습니다.
> `topSuccessfulRequester`는 월간 승인 처리된 정책 이의제기를 가장 많이 성공한 구성원과 최신 승인 이력 최대 3개를 반환합니다. `recentApprovedAppeals` 배열은 승인 처리 시각 기준 내림차순(`resolved_at DESC, id DESC`)이며, 각 항목에는 요청 시각 `requestedAt`을 포함합니다.
> `topAcceptedApprover`는 월간 승인 처리된 정책 이의제기를 가장 많이 수락한 구성원과 최신 수락 이력 최대 3개를 반환합니다. `recentAcceptedAppeals`는 `resolvedAt` 내림차순(`resolved_at DESC, id DESC`)입니다.
> 데이터가 없으면 각 대표 인물의 ID/이름은 `null`, 건수는 `0`, 이력 배열은 `[]`를 반환합니다.
> `communicationScore`는 `type='NORMAL'`인 정책 이의제기와 mission 이벤트를 기반으로 계산한 월간 소통 점수입니다. `EMERGENCY`는 계산에서 제외합니다.
> `appealCarryInCount`는 월 시작 이전 생성됐고 월 시작 이전에 해결 또는 취소되지 않은 `type='NORMAL'` 이의제기 수입니다. `missionCarryInCount`는 월 시작 이전 생성됐고 월 시작 이전 완료 또는 취소 로그가 없는 미션 수입니다.
> `appealBase = appealCarryInCount + totalAppeals`, `missionBase = missionCarryInCount + totalMissionCount`로 계산합니다.
> 두 base가 모두 0이면 `communicationScore`는 `null`입니다.
> 한 base만 0이면 나머지 축 비율만 사용해 `round(rate * 100, 2)`를 반환합니다.
> 두 base가 모두 양수이면 `appealResponseRate = (approvedAppeals + rejectedAppeals) / appealBase`, `missionCompletionRate = completedMissionCount / missionBase`, `communicationScore = round(((appealResponseRate * 0.55) + (missionCompletionRate * 0.45)) * 100, 2)`를 반환합니다.

---

### 3.8 NOTIFICATIONS 도메인 _(api-notification)_

> SSE 실시간 스트림(`GET /events/stream`)과 병행하여, REST API로 알림 이력을 조회합니다.
> SSE는 실시간 푸시용이며, REST는 알림 이력 조회 및 읽음 처리용입니다.

#### 3.8.1 알림 목록 조회

**GET** `/notifications` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론.
> 서버에서 `sent_at >= NOW() - INTERVAL 30 DAY AND deleted_at IS NULL` 조건으로 필터링합니다.
> `hasNext: false`이면 프론트에서 "30일이 지난 알림은 표시되지 않습니다" 문구를 렌더링합니다.

**Query Parameters**:

| 파라미터 | 타입    | 필수 | 설명                                                                                                      |
| -------- | ------- | ---- | --------------------------------------------------------------------------------------------------------- |
| cursor   | string  | ❌   | 다음 페이지 커서 (`nextCursor` 값, 미전달 시 첫 페이지)                                                   |
| size     | integer | ❌   | 조회 크기 (기본: 20)                                                                                      |
| isRead   | boolean | ❌   | 읽음 여부 필터                                                                                            |
| type     | string  | ❌   | 콤마 구분 타입 필터 (예: `CUSTOMER_BLOCKED,CUSTOMER_UNBLOCKED`). 자세한 값은 아래 `type` Enum 값 표 참고. |

**`type` Enum 값**:

| 값                   | 설명                    |
| -------------------- | ----------------------- |
| `QUOTA_UPDATED`      | 할당량 변경 알림        |
| `CUSTOMER_BLOCKED`   | 사용자 차단 알림        |
| `CUSTOMER_UNBLOCKED` | 사용자 차단 해제 알림   |
| `THRESHOLD_ALERT`    | 잔여량 임계치 도달 알림 |
| `POLICY_CHANGED`     | 정책 변경 알림          |
| `MISSION_CREATED`    | 새 미션 생성 알림       |
| `REWARD_REQUESTED`   | 보상 요청 알림          |
| `REWARD_APPROVED`    | 보상 승인 알림          |
| `REWARD_REJECTED`    | 보상 거절 알림          |
| `APPEAL_CREATED`     | 이의제기 생성 알림      |
| `APPEAL_APPROVED`    | 이의제기 승인 알림      |
| `APPEAL_REJECTED`    | 이의제기 거절 알림      |
| `EMERGENCY_APPROVED` | 긴급 요청 승인 알림     |
| `ADMIN_PUSH`         | 관리자 수동 Push 알림   |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "notificationId": 1001,
        "type": "THRESHOLD_ALERT",
        "title": "데이터 경고",
        "message": "가족 데이터가 50% 남았습니다",
        "payload": {
          "threshold": 50,
          "remainingPercent": 48.5
        },
        "isRead": false,
        "sentAt": "2024-01-15T10:00:00Z"
      },
      {
        "notificationId": 1002,
        "type": "CUSTOMER_BLOCKED",
        "title": "데이터 차단",
        "message": "데이터 한도 초과로 차단되었습니다",
        "payload": {
          "reason": "MONTHLY_LIMIT_EXCEEDED"
        },
        "isRead": true,
        "sentAt": "2024-01-15T09:30:00Z"
      }
    ],
    "nextCursor": "eyJub3RpZmljYXRpb25JZCI6MTAwMn0=",
    "hasNext": true,
    "unreadCount": 5
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.8.2 읽지 않은 알림 수 조회

**GET** `/notifications/unread-count` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론.
> 30일 이내(`sent_at >= NOW() - INTERVAL 30 DAY`) + 소프트 삭제 미적용(`deleted_at IS NULL`) 조건으로 집계합니다.
> 앱 배지 표시 등 경량 카운트 조회에 사용합니다.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "unreadCount": 5
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.8.3 개별 알림 읽음 처리

**PATCH** `/notifications/{notificationId}/read` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론

**Path Parameters**:

| 파라미터       | 타입    | 설명    |
| -------------- | ------- | ------- |
| notificationId | integer | 알림 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.8.4 전체 알림 읽음 처리

**PATCH** `/notifications/read-all` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론. 해당 사용자의 모든 미읽음 알림을 읽음으로 처리합니다.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.8.5 알림 삭제

**DELETE** `/notifications/{notificationId}` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론. 소프트 삭제(`deleted_at` 설정)로 처리합니다. 본인 알림만 삭제 가능합니다.

**Path Parameters**:

| 파라미터       | 타입    | 설명    |
| -------------- | ------- | ------- |
| notificationId | integer | 알림 ID |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

### 3.9 ADMIN 도메인

> 관리자 전용 인증 및 백오피스 관리 엔드포인트. CUSTOMER 테이블과 분리된 ADMIN 테이블에 대해 인증을 수행합니다.

#### 3.9.1 관리자 회원가입 (🌟 New)

**POST** `/admin/signup` | 권한: `없음`

**Request**:

```json
{
  "email": "admin001@dabom.site",
  "password": "password123"
}
```

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "id": 1
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.9.2 관리자 로그인

**POST** `/admin/login` | 권한: `없음`

**Request**:

```json
{
  "email": "admin001@dabom.site",
  "password": "password123"
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "role": "ADMIN"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.9.3 관리자 토큰 갱신 (🔥 2차)

**POST** `/admin/refresh` | 권한: `없음`

**Request**:

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 1800
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.9.4 관리자 내 정보 조회 (🌟 New)

**GET** `/admin/me` | 권한: `admin`

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "adminId": 1,
    "email": "admin001@dabom.site",
    "name": "관리자"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.9.5 관리자 로그아웃 (🔥 2차)

**POST** `/admin/logout` | 권한: `admin`

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.9.6 관리자 대시보드 (🔥 2차)

**GET** `/admin/dashboard` | 권한: `admin`

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "totalFamilies": 250000,
    "activeFamilies": 248500,
    "totalUsers": 1000000,
    "blockedUsers": 1523,
    "todayEvents": 432000000,
    "currentTps": 5000,
    "systemHealth": {
      "redis": "UP",
      "kafka": "UP",
      "mysql": "UP"
    },
    "recentBlocks": [
      {
        "familyId": 100,
        "customerId": 12346,
        "reason": "MONTHLY_LIMIT_EXCEEDED",
        "blockedAt": "2024-01-15T10:30:00Z"
      }
    ]
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.9.7 감사 로그 조회 (🔥 2차)

**GET** `/admin/audit/logs` | 권한: `admin`

**Query Parameters**:

| 파라미터   | 타입    | 필수 | 설명                                           |
| ---------- | ------- | ---- | ---------------------------------------------- |
| action     | string  | ❌   | 액션 타입 필터                                 |
| entityType | string  | ❌   | 엔티티 유형 필터 (FAMILY, CUSTOMER, POLICY 등) |
| actorId    | integer | ❌   | 행위자 ID 필터                                 |
| page       | integer | ❌   | 페이지 번호 (기본: 0)                          |
| size       | integer | ❌   | 페이지 크기 (기본: 20)                         |

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "logId": 10001,
        "actorId": 99999,
        "actorName": "관리자",
        "action": "POLICY_CHANGED",
        "entityType": "FAMILY",
        "entityId": 100,
        "oldValue": { "totalQuotaBytes": 107374182400 },
        "newValue": { "totalQuotaBytes": 214748364800 },
        "ipAddress": "192.168.1.100",
        "createdAt": "2024-01-15T10:30:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1523,
    "totalPages": 77
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### 3.10 UPLOADS 도메인 (🌟 New)

> 클라이언트에서 업로드한 이미지를 R2에 저장하고 CDN URL을 반환합니다.

#### 3.10.1 이미지 업로드

**POST** `/uploads/images` | 권한: `admin`

> 백오피스에서 보상 썸네일, 미션 이미지 등을 업로드합니다.
> 서버가 UUID 기반 파일명을 생성하고, type에 따라 R2 저장 경로를 결정합니다.
> 클라이언트는 반환된 URL을 이후 도메인 API 요청(예: `POST /admin/rewards/templates`)에 사용합니다.

**Headers**:

```http
Content-Type: multipart/form-data
```

**Request Body**:

| 필드 | 타입   | 필수 | 설명                                   |
| ---- | ------ | ---- | -------------------------------------- |
| file | file   | ✅   | 업로드할 이미지 파일                   |
| type | string | ✅   | 이미지 용도 (REWARD, PROFILE, MISSION) |

**파일 검증 정책**:

| 항목        | 값                                 |
| ----------- | ---------------------------------- |
| 허용 MIME   | image/png, image/jpeg, image/webp  |
| 최대 크기   | 5MB                                |
| 파일명 생성 | UUID v4 + Content-Type 기반 확장자 |

**type ENUM → R2 경로 매핑**:

| type    | R2 경로 prefix | 설명             |
| ------- | -------------- | ---------------- |
| REWARD  | rewards/       | 보상 썸네일      |
| PROFILE | profiles/      | 사용자 프로필    |
| MISSION | missions/      | 미션 관련 이미지 |

**서버 처리 흐름**:

1. multipart 요청 수신
2. 파일 존재 여부 검증
3. 파일 크기 검증 (5MB 초과 시 거부)
4. MIME 타입 검증 (허용 목록 외 거부)
5. UUID v4 파일명 생성
6. Content-Type 기반 확장자 결정 (image/png→.png, image/jpeg→.jpg, image/webp→.webp)
7. type 기반 R2 경로 생성 (`{type}/{uuid}.{ext}`)
8. R2 업로드 (S3 호환 API)
9. CDN URL 생성 (`https://cdn.dabom.site/{type}/{uuid}.{ext}`)
10. URL 반환

**Responses**:

**201 Created**:

```json
{
  "success": true,
  "data": {
    "url": "https://cdn.dabom.site/rewards/550e8400-e29b-41d4-a716-446655440000.png"
  },
  "timestamp": "2026-03-12T10:30:00Z"
}
```

### 3.11 PUSH 도메인 _(api-notification)_

> PWA Web Push 구독 관리 및 수동 발송 엔드포인트입니다.
> 구현 코드(`dabom-api-notification`의 `WebPushController`) 기반입니다.

#### 3.11.1 VAPID 공개키 조회

**GET** `/push/vapid-public-key` | 권한: `없음`

> 인증 불필요. 프론트가 `PushManager.subscribe()` 호출 전에 VAPID 공개키를 조회합니다.

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": {
    "publicKey": "BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkrxZJjSgSnfckjBJuBkr3qBUYIHBQFLXYp5Nksh8U"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.11.2 Push 구독 등록

**POST** `/push/subscribe` | 권한: `member`

> 같은 endpoint + 같은 customer → 키 갱신, 같은 endpoint + 다른 customer → 재할당, 새 endpoint → 신규 생성.
> endpoint URL 검증: HTTPS 필수, 내부 IP 차단 (SSRF 방어).

**Request**:

```json
{
  "endpoint": "https://fcm.googleapis.com/fcm/send/...",
  "keys": {
    "p256dh": "BNcRdreALRFXTkOOUHK1EtK2wtaz5Ry4YfYCA_0QTpQtUbVlUls0VJXg7A8u-Ts1XbjhazAkj7I99e8p8REfWRk=",
    "auth": "tBHItJI5svbpC7htDNae8w=="
  }
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.11.3 Push 구독 해제

**DELETE** `/push/subscribe` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론. 구독 정보를 삭제합니다 (hard delete).

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 3.11.4 Push 수동 발송 (운영용)

**POST** `/push/send` | 권한: `admin`

> 운영자가 특정 사용자에게 수동으로 Push 알림을 발송합니다 (시스템 공지, 긴급 안내 등).

**Request**:

```json
{
  "customerId": 12345,
  "title": "시스템 공지",
  "message": "서비스 점검이 예정되어 있습니다."
}
```

**Responses**:

**200 OK**:

```json
{
  "success": true,
  "data": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## 4. processor-usage (정책 엔진)

> **아키텍처 변경 (v3.0)**: processor-usage는 Kafka Consumer로 동작하며, 직접적인 HTTP API는 제공하지 않습니다.
> 내부적으로 Redis Lua Script를 실행하여 정책 평가를 수행합니다.
> 결과 notification은 `usage_event_outbox`에 적재되며, 배치 서버가 `notification-events`로 후행 발행합니다.

### 4.1 정책 평가 결과 이벤트

배치 서버가 `usage_event_outbox`를 조회해 `notification-events` 토픽으로 발행하는 Kafka 이벤트 (EventEnvelope 패턴):

> **Note (v24.0)**: `eventType`은 `NOTIFICATION`으로 고정하며, `subType` 없이 `payload.type`으로 구분합니다.

**THRESHOLD_ALERT** 예시:

```json
{
  "eventId": "evt_123",
  "eventType": "NOTIFICATION",
  "timestamp": "2026-03-16T10:15:30Z",
  "payload": {
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
}
```

---

## 5. SSE 명세 _(api-notification)_

### 5.1 SSE 연결

**GET** `/events/stream` | 권한: `member`

> `customerId`는 JWT 토큰에서 추론합니다 (경로 파라미터 없음). SSE 실시간 스트림과 REST `/notifications/*` 엔드포인트가 공존합니다.
> SSE는 실시간 푸시용이며, REST는 알림 이력 조회용입니다. SSE 스트림은 api-notification(`noti.dabom.site`)에서 제공합니다. `text/event-stream` 응답.

**Headers**:

```http
Accept: text/event-stream
Authorization: Bearer <access_token>
```

**실제 전송 이벤트 이름**:

- 연결: `connected`
- 하트비트: `heartbeat`
- 총 사용량: `usage-updated`
- 구성원별 사용량: `usage-updated-by-member`
- 알림 이벤트: `NotificationType.name()` 그대로 사용

**실시간 사용량 payload (`usage-updated`)**:

```json
{
  "familyId": 10,
  "totalUsedBytes": 123456789,
  "totalQuotaBytes": 214748364800,
  "remainingBytes": 214624908011
}
```

**구성원별 사용량 payload (`usage-updated-by-member`)**:

```json
{
  "familyId": 10,
  "customerId": 23,
  "monthlyUsedBytes": 734003200
}
```

### 5.2 이벤트 타입

#### NOTIFICATION 예시

```text
event: notification
data: {"eventType":"NOTIFICATION","timestamp":"2026-03-16T10:15:30Z","payload":{"familyId":100,"customerId":1,"type":"THRESHOLD_ALERT","title":"데이터 사용량 경고","message":"가족 데이터 잔여량이 10% 미만입니다.","data":{"threshold":10,"triggerStatus":"WARNING_10"}}}
```

#### CUSTOMER_UNBLOCKED

```text
event: notification
data: {"eventType":"CUSTOMER_UNBLOCKED","familyId":100,"payload":{"customerId":12346,"reason":"TIME_BLOCK_RELEASED","unblockedAt":"2024-01-15T18:00:00Z"}}
```

#### policy-updated

```text
event: policy-updated
data: {"policyKey":"LIMIT:DATA:MONTHLY","targetCustomerId":12346,"oldValue":"2147483648","newValue":"536870912"}
```

#### POLICY_CHANGED

```text
event: notification
data: {"eventType":"POLICY_CHANGED","familyId":100,"payload":{"customerId":12346,"policyType":"LIMIT:DATA:MONTHLY","changedAt":"2024-01-15T10:30:00Z"}}
```

#### MISSION_CREATED

```text
event: notification
data: {"eventType":"MISSION_CREATED","familyId":100,"payload":{"missionId":501,"title":"독서 미션","assignedTo":12346,"createdAt":"2024-01-15T10:30:00Z"}}
```

#### REWARD_REQUESTED

```text
event: notification
data: {"eventType":"REWARD_REQUESTED","familyId":100,"payload":{"missionId":501,"requesterId":12346,"requesterName":"김민수","requestedAt":"2024-01-15T11:00:00Z"}}
```

#### REWARD_APPROVED

```text
event: notification
data: {"eventType":"REWARD_APPROVED","familyId":100,"payload":{"missionId":501,"rewardId":201,"approvedAt":"2024-01-15T12:00:00Z"}}
```

#### REWARD_REJECTED

```text
event: notification
data: {"eventType":"REWARD_REJECTED","familyId":100,"payload":{"missionId":501,"rejectReason":"미션 조건 미충족","rejectedAt":"2024-01-15T12:00:00Z"}}
```

#### APPEAL_CREATED

```text
event: notification
data: {"eventType":"APPEAL_CREATED","familyId":100,"payload":{"appealId":301,"requesterId":12346,"requesterName":"김민수","createdAt":"2024-01-15T10:00:00Z"}}
```

#### APPEAL_APPROVED

```text
event: notification
data: {"eventType":"APPEAL_APPROVED","familyId":100,"payload":{"appealId":301,"resolvedAt":"2024-01-15T13:00:00Z"}}
```

#### APPEAL_REJECTED

```text
event: notification
data: {"eventType":"APPEAL_REJECTED","familyId":100,"payload":{"appealId":301,"rejectReason":"이의제기 사유 불충분","resolvedAt":"2024-01-15T13:00:00Z"}}
```

#### EMERGENCY_APPROVED

```text
event: notification
data: {"eventType":"EMERGENCY_APPROVED","familyId":100,"payload":{"appealId":302,"grantedBytes":104857600,"approvedAt":"2024-01-15T14:00:00Z"}}
```

### 5.3 연결 유지

- **Heartbeat**: 30초마다 `:heartbeat` 전송
- **재연결**: 연결 끊김 시 클라이언트가 자동 재연결 (Last-Event-ID 헤더 활용)

### 5.4 내부 테스트 SSE

**GET** `/events/stream/test/{customerId}` | 내부 테스트 전용

> 인증 없이 특정 고객 기준으로 SSE를 구독하는 테스트 엔드포인트. 프로덕션에서는 사용하지 않는다.

---

## 6. 알림 연동 메모

api-core 커밋 `778d64e` 기준으로 아래 동작은 DB 커밋 이후 `usage_event_outbox`를 통해 notification 이벤트를 발행한다.

- `PATCH /families/policies`
  - `MONTHLY_LIMIT` 수정 시 `QUOTA_UPDATED`
  - `MANUAL_BLOCK` 활성화 시 `CUSTOMER_BLOCKED`
  - `MANUAL_BLOCK` 비활성화 시 `CUSTOMER_UNBLOCKED`
  - `TIME_BLOCK`, `APP_BLOCK` 수정 시 `POLICY_CHANGED`
- `POST /appeals` → 각 OWNER에게 `APPEAL_CREATED`
- `PATCH /appeals/{appealId}/respond` → 요청자에게 `APPEAL_APPROVED` 또는 `APPEAL_REJECTED`
- `POST /appeals/emergency` → 각 OWNER에게 `EMERGENCY_APPROVED`
- `POST /missions` → 대상 구성원에게 `MISSION_CREATED`
- `POST /missions/{missionId}/request` → 각 OWNER에게 `REWARD_REQUESTED`
- `PUT /rewards/requests/{requestId}/respond` → 요청자에게 `REWARD_APPROVED` 또는 `REWARD_REJECTED`

---

## 7. 에러 코드 정의

### 7.1 인증 에러 (AUTH_xxx)

| 코드                         | HTTP | 설명               |
| ---------------------------- | ---- | ------------------ |
| AUTH_INVALID_CREDENTIALS     | 401  | 잘못된 인증 정보   |
| AUTH_TOKEN_EXPIRED           | 401  | 토큰 만료          |
| AUTH_TOKEN_INVALID           | 401  | 유효하지 않은 토큰 |
| AUTH_INSUFFICIENT_PERMISSION | 403  | 권한 부족          |

### 7.2 데이터 에러 (DATA_xxx)

| 코드                       | HTTP | 설명                       |
| -------------------------- | ---- | -------------------------- |
| DATA_FAMILY_NOT_FOUND      | 404  | 가족 그룹 없음             |
| DATA_USER_NOT_FOUND        | 404  | 사용자 없음                |
| DATA_MEMBER_LIMIT_EXCEEDED | 400  | 최대 구성원 수 초과 (10명) |

### 7.3 정책 에러 (POLICY_xxx)

| 코드                      | HTTP | 설명                            |
| ------------------------- | ---- | ------------------------------- |
| POLICY_USER_BLOCKED       | 403  | 사용자 차단됨                   |
| POLICY_QUOTA_EXCEEDED     | 403  | 할당량 초과                     |
| POLICY_TIME_BLOCKED       | 403  | 시간대 차단 중                  |
| POLICY_ALREADY_BLOCKED    | 409  | 이미 차단됨                     |
| POLICY_TEMPLATE_NOT_FOUND | 404  | 정책 템플릿 없음                |
| POLICY_TEMPLATE_IN_USE    | 409  | 사용 중인 정책 템플릿 삭제 시도 |

### 7.4 이의제기 에러 (APPEAL_xxx)

| 코드                                | HTTP | 설명                                              |
| ----------------------------------- | ---- | ------------------------------------------------- |
| APPEAL_NOT_FOUND                    | 404  | 이의제기를 찾을 수 없음                           |
| APPEAL_ALREADY_RESOLVED             | 409  | 이미 처리된 이의제기                              |
| APPEAL_FORBIDDEN                    | 403  | 이의제기에 접근 권한 없음                         |
| APPEAL_INVALID_DESIRED_RULES        | 400  | desiredRules 스키마가 policy.type과 맞지 않음     |
| APPEAL_EMERGENCY_MONTHLY_LIMIT      | 429  | 이번 달 긴급 요청을 이미 사용함                   |
| APPEAL_EMERGENCY_INVALID_BYTES      | 400  | 긴급 요청 바이트 입력 검증 실패 (현재 API 미사용) |
| APPEAL_EMERGENCY_UNLIMITED          | 400  | 무제한 쿼터 사용자는 긴급 요청 불가               |
| APPEAL_CANCEL_FORBIDDEN             | 403  | 본인이 생성한 이의제기가 아님                     |
| APPEAL_NOT_CANCELLABLE              | 409  | PENDING 상태가 아니라 취소 불가                   |
| APPEAL_EMERGENCY_CANCEL_NOT_ALLOWED | 400  | EMERGENCY 타입은 취소 불가                        |

### 7.5 미션 에러 (MISSION_xxx)

| 코드                             | HTTP | 설명                                                |
| -------------------------------- | ---- | --------------------------------------------------- |
| MISSION_NOT_FOUND                | 404  | 미션을 찾을 수 없음                                 |
| MISSION_NOT_ACTIVE               | 409  | 미션이 활성 상태가 아님                             |
| MISSION_ALREADY_COMPLETED        | 409  | 이미 완료된 미션                                    |
| REWARD_TEMPLATE_NOT_FOUND        | 404  | 보상 템플릿을 찾을 수 없음                          |
| MISSION_REQUEST_NOT_FOUND        | 404  | 보상 요청을 찾을 수 없음                            |
| MISSION_REQUEST_ALREADY_RESOLVED | 409  | 이미 처리된 보상 요청                               |
| MISSION_TARGET_INVALID           | 400  | 미션 대상이 유효하지 않음 (MEMBER 역할만 대상 가능) |
| MISSION_NOT_ASSIGNED             | 403  | 본인에게 배정되지 않은 미션에 대한 요청             |
| REWARD_GRANT_NOT_FOUND           | 404  | 보상 지급 이력을 찾을 수 없음                       |

### 7.6 리캡 에러 (RECAP_xxx)

| 코드            | HTTP | 설명                |
| --------------- | ---- | ------------------- |
| RECAP_NOT_FOUND | 404  | 리캡을 찾을 수 없음 |

### 7.7 알림 에러 (NOTIFICATION_xxx)

| 코드                   | HTTP | 설명                       |
| ---------------------- | ---- | -------------------------- |
| NOTIFICATION_NOT_FOUND | 404  | 알림 없음                  |
| NOTIFICATION_FORBIDDEN | 403  | 해당 알림에 대한 권한 없음 |

### 7.8 업로드 에러 (UPLOAD_xxx)

| 코드                  | HTTP | 설명               |
| --------------------- | ---- | ------------------ |
| UPLOAD_FILE_REQUIRED  | 400  | 파일이 없음        |
| UPLOAD_INVALID_TYPE   | 400  | 지원하지 않는 MIME |
| UPLOAD_FILE_TOO_LARGE | 400  | 파일 크기 초과     |
| UPLOAD_FAILED         | 500  | R2 업로드 실패     |

### 7.9 Push 에러 (PUSH_xxx / SUBSCRIPTION_xxx)

| 코드                   | HTTP | 설명                              |
| ---------------------- | ---- | --------------------------------- |
| SUBSCRIPTION_NOT_FOUND | 404  | 구독 정보 없음                    |
| PUSH_SEND_FAILED       | 500  | Push 발송 실패                    |
| INVALID_ENDPOINT_URL   | 400  | 유효하지 않은 구독 엔드포인트 URL |

### 7.10 시스템 에러 (SYS_xxx)

| 코드                    | HTTP | 설명                          |
| ----------------------- | ---- | ----------------------------- |
| SYS_INTERNAL_ERROR      | 500  | 내부 서버 오류                |
| SYS_REDIS_UNAVAILABLE   | 503  | Redis 장애 (DB Fallback 모드) |
| SYS_RATE_LIMIT_EXCEEDED | 429  | API 호출 제한 초과            |
| SYS_SERVICE_UNAVAILABLE | 503  | 서비스 이용 불가              |

### 7.11 Outbox 에러 (OUTBOX_xxx)

| 코드       | HTTP | 설명                    |
| ---------- | ---- | ----------------------- |
| OUTBOX_001 | 500  | Outbox 이벤트 발행 실패 |

---

## 8. 엔드포인트 요약

> 총 **63개** 공개 엔드포인트 + 내부 테스트 SSE 1개 (v24.3 기준)

### 8.1 CUSTOMERS 도메인

| 메서드 | 경로                 | 설명                         | 권한   |
| ------ | -------------------- | ---------------------------- | ------ |
| POST   | `/customers/login`   | 로그인                       | 없음   |
| POST   | `/customers/refresh` | Access Token 재발급          | 없음   |
| POST   | `/customers/logout`  | 로그아웃                     | 없음   |
| POST   | `/customers/signup`  | 회원가입 (✨UPDATED)         | 없음   |
| GET    | `/customers/me`      | 내 계정 정보 조회            | member |
| GET    | `/customers/mypage`  | 마이페이지 조회 (🌟 New)     | member |

### 8.2 FAMILIES 도메인

| 메서드 | 경로                              | 설명                                      | 권한   |
| ------ | --------------------------------- | ----------------------------------------- | ------ |
| POST   | `/admin/families`                 | 가족 그룹 검색                            | admin  |
| GET    | `/families/usage/current`         | 실시간 가족 사용량                        | member |
| GET    | `/families/usage/customers`       | 실시간 구성원별 사용량                    | member |
| GET    | `/admin/families/{familyId}`      | 가족 정보 상세 조회                       | admin  |
| PATCH  | `/admin/families/{familyId}`      | 관리자 가족 구성원 수정 (🌟 New)          | admin  |
| GET    | `/families/policies`              | 가족 구성원 정책 조회                     | member |
| PATCH  | `/families/policies`              | 가족 구성원 정책 수정                     | owner  |
| GET    | `/families/members`               | 가족 구성원 중 자녀 목록 조회             | owner  |
| PUT    | `/families`                       | 가족 이름 수정                            | owner  |
| GET    | `/families/usage/dashboard`       | 사용량 통계 대시보드                      | member |
| GET    | `/rewards/templates`              | 보상 템플릿 목록 조회                     | owner  |

### 8.3 POLICIES 도메인

| 메서드 | 경로                   | 설명           | 권한  |
| ------ | ---------------------- | -------------- | ----- |
| GET    | `/policies`            | 정책 조회      | admin |
| POST   | `/policies`            | 정책 생성      | admin |
| DELETE | `/policies/{policyId}` | 정책 삭제      | admin |
| GET    | `/policies/{policyId}` | 정책 상세 조회 | admin |
| PUT    | `/policies/{policyId}` | 정책 수정      | admin |

### 8.4 APPEALS 도메인

| 메서드 | 경로                           | 설명                                  | 권한            |
| ------ | ------------------------------ | ------------------------------------- | --------------- |
| GET    | `/appeals/policies`            | 이의제기 가능 정책 목록 조회 (🌟 New) | member          |
| GET    | `/appeals`                     | 이의제기 목록 조회                    | member          |
| GET    | `/appeals/{appealId}`          | 이의제기 상세 조회                    | member          |
| POST   | `/appeals`                     | 이의제기 생성                         | member (MEMBER) |
| PATCH  | `/appeals/{appealId}/respond`  | 이의제기 승인/거절                    | owner           |
| POST   | `/appeals/{appealId}/comments` | 이의제기 댓글 작성                    | member          |
| POST   | `/appeals/emergency`           | 긴급 쿼터 요청                        | member (MEMBER) |
| PATCH  | `/appeals/{appealId}/cancel`   | 이의제기 취소                         | member (MEMBER) |

### 8.5 MISSIONS 도메인

| 메서드 | 경로                            | 설명               | 권한            |
| ------ | ------------------------------- | ------------------ | --------------- |
| GET    | `/missions`                     | 미션 항목 목록 조회 | member          |
| GET    | `/missions/logs`                | 미션 상태 변화 로그 | member          |
| GET    | `/missions/history`             | 미션 요청 이력 조회 | member          |
| POST   | `/missions`                     | 미션 + 보상 생성    | owner           |
| DELETE | `/missions/{missionId}`         | 미션 삭제           | owner           |
| POST   | `/missions/{missionId}/request` | 미션 승인 요청      | member (MEMBER) |

### 8.6 REWARDS 도메인

| 메서드 | 경로                                    | 설명                 | 권한            |
| ------ | --------------------------------------- | -------------------- | --------------- |
| PUT    | `/rewards/requests/{requestId}/respond` | 보상 승인/거절 처리   | owner           |
| GET    | `/rewards/received`                     | 내가 받은 보상 조회   | member (MEMBER) |
| GET    | `/admin/rewards/templates`              | 보상 템플릿 목록 조회 | admin           |
| GET    | `/admin/rewards/templates/{id}`         | 보상 템플릿 상세 조회 | admin           |
| POST   | `/admin/rewards/templates`              | 보상 템플릿 생성      | admin           |
| PUT    | `/admin/rewards/templates/{id}`         | 보상 템플릿 수정      | admin           |
| DELETE | `/admin/rewards/templates/{id}`         | 보상 템플릿 삭제      | admin           |
| GET    | `/admin/rewards/grants`                 | 보상 지급 내역 조회   | admin           |

### 8.7 RECAPS 도메인

| 메서드 | 경로              | 설명                | 권한   |
| ------ | ----------------- | ------------------- | ------ |
| GET    | `/recaps/monthly` | 월간 가족 리캡 조회 | member |

### 8.8 NOTIFICATIONS 도메인 _(api-notification)_

| 메서드 | 경로                                   | 설명                                           | 권한   |
| ------ | -------------------------------------- | ---------------------------------------------- | ------ |
| GET    | `/notifications`                       | 알림 목록 (커서 무한스크롤, type·isRead, 30일) | member |
| GET    | `/notifications/unread-count`          | 읽지 않은 알림 수 (30일 이내)                  | member |
| PATCH  | `/notifications/{notificationId}/read` | 개별 읽음 처리                                 | member |
| PATCH  | `/notifications/read-all`              | 전체 읽음 처리                                 | member |
| DELETE | `/notifications/{notificationId}`      | 알림 삭제                                      | member |

### 8.9 ADMIN 도메인

| 메서드 | 경로                | 설명                       | 권한  |
| ------ | ------------------- | -------------------------- | ----- |
| POST   | `/admin/signup`     | 관리자 회원가입 (🌟 New)   | 없음  |
| POST   | `/admin/login`      | 관리자 로그인              | 없음  |
| POST   | `/admin/refresh`    | 관리자 토큰 갱신           | 없음  |
| GET    | `/admin/me`         | 관리자 내 정보 조회 (🌟 New) | admin |
| POST   | `/admin/logout`     | 관리자 로그아웃            | 없음  |
| GET    | `/admin/dashboard`  | 관리자 대시보드            | admin |
| GET    | `/admin/audit/logs` | 감사 로그 조회             | admin |

### 8.10 UPLOADS 도메인

| 메서드 | 경로              | 설명          | 권한  |
| ------ | ----------------- | ------------- | ----- |
| POST   | `/uploads/images` | 이미지 업로드 | admin |

### 8.11 PUSH 도메인 _(api-notification)_

| 메서드 | 경로                     | 설명                    | 권한   |
| ------ | ------------------------ | ----------------------- | ------ |
| GET    | `/push/vapid-public-key` | VAPID 공개키 조회       | 없음   |
| POST   | `/push/subscribe`        | Push 구독 등록          | member |
| DELETE | `/push/subscribe`        | Push 구독 해제          | member |
| POST   | `/push/send`             | Push 수동 발송 (운영용) | admin  |

### SSE

| 메서드 | 경로                               | 설명                                    | 권한        |
| ------ | ---------------------------------- | --------------------------------------- | ----------- |
| GET    | `/events/stream`                   | SSE 실시간 이벤트 스트림                | member      |
| GET    | `/events/stream/test/{customerId}` | 내부 테스트 SSE (인증 없음)             | 내부 테스트 |

---

## 관련 문서

- [기획서](./SPECIFICATION.md)
- [아키텍처 설계서](./ARCHITECTURE.md)
- [데이터 모델](./DATA_MODEL.md)
- [용어집](./GLOSSARY.md)
- [ERD 설계서](./ERD.md)
- [Kafka 메시지 스키마](./designs/kafka/MESSAGE_SCHEMA.md)
- [Kafka 토픽 설계서](./designs/kafka/TOPIC_DESIGN.md)
- [Redis Key 설계서](./designs/redis/KEY_DESIGN.md)
