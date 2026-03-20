# 다봄(Dabom) 프로젝트

## 버전 이력 (Changelog)

| 버전 | 날짜 | 내용 |
| --- | --- | --- |
| **v25.0** | 2026-03-20 | 전체 문서 v25.0 Major 버전 동기화 |
| **v23.1** | 2026-03-15 | API_SPECIFICATION v23.1 동기화: 알림 읽음 처리 경로 RESTful 개선 (`/notifications/{id}/read`, `/notifications/read-all`) |
| **v23.0** | 2026-03-15 | Major 버전 동기화 |
| **v22.0** | 2026-03-12 | API_SPECIFICATION v22.2 동기화: 백오피스 이미지 업로드 기능 명세 추가 |
| **v21.0** | 2026-03-12 | v21.0 - API_SPECIFICATION v21.6 동기화: 엔드포인트 경로 갱신 (/appeals/*, /recaps/*), 용어 갱신 (협상→이의제기, 리포트→리캡) |
| **v11.0** | 2026-03-03 | v11.0 - 2차기획서 Phase 2 기능 반영: 미션/보상, 이의제기/긴급요청, 리캡, 프로필/약관, 알림 관리, 관리자 보상 템플릿 화면 추가 |

## 기술 스택 (Tech Stack)

- Next.js (React)
- TypeScript
- Tailwind CSS
- TanStack Query(React Query)
- SSE (Server-Sent Events)
- PWA (Progressive Web App)

## 기술 스택 선택 배경

### **Next.js & React**
React의 생산성과 Next.js의 강력한 기능(SSR, SSG, ISR, 파일 기반 라우팅 등)을 결합하여 성능과 검색 엔진 최적화(SEO)가 우수한 모던 웹 애플리케이션을 빠르게 개발하기 위해 선택했습니다.

### **TypeScript**
대규모 프로젝트에서 코드의 안정성과 유지보수성을 높이기 위해 도입했습니다. 정적 타이핑을 통해 컴파일 단계에서 오류를 사전에 발견하고, 자동 완성과 같은 IDE 지원을 강화하여 개발 생산성을 향상시킵니다.

### **Tailwind CSS**
Utility-First 접근법을 통해 미리 정의된 클래스를 조합하여 빠르고 일관된 UI를 구축할 수 있습니다. 커스터마이징이 용이하며, CSS 코드베이스가 불필요하게 커지는 것을 방지합니다.

### **Turborepo & pnpm (모노레포)**
`admin`과 `service`라는 두 개의 애플리케이션과 공통 `shared` 패키지를 효율적으로 관리하기 위해 모노레포 구조를 채택했습니다.
- **Turborepo**: 모노레포를 위한 고성능 빌드 시스템으로, 캐싱을 통해 빌드 및 테스트 시간을 단축하고 병렬 실행을 지원하여 개발 워크플로우를 최적화합니다.
- **pnpm**: 디스크 공간을 효율적으로 사용하고 의존성 설치 속도가 빠른 패키지 매니저로, 모노레포 환경(workspaces)을 효과적으로 지원합니다.

### **TanStack Query (React Query)**
서버 상태 관리를 위한 강력한 라이브러리입니다. 클라이언트 측에서 서버 데이터를 가져오고, 캐싱하며, 동기화하는 복잡한 로직을 단순화하여 개발자가 비즈니스 로직에 더 집중할 수 있도록 돕습니다.

### **React Hook Form & Zod**
- **React Hook Form**: 비제어 컴포넌트(uncontrolled components) 기반으로 불필요한 리렌더링을 최소화하여 폼 성능을 최적화합니다. Hooks 기반 API는 사용이 간편하고 직관적입니다.
- **Zod**: TypeScript와 함께 사용할 때 강력한 스키마 기반 유효성 검사를 제공합니다. 런타임에서도 타입 안정성을 보장할 수 있어 안정적인 폼 처리가 가능합니다.

## 디자인 시스템 (Design System)

### Color System
다봄 서비스의 일관된 UI를 위해 명명된 컬러 토큰(Token)을 사용합니다.

#### 1. Primary (브랜드 컬러)
| 토큰명 (Token) | Hex Code | 컬러 |
| :--- | :--- | :---: |
| `primary-100` | `#ffedf5` | ![#ffedf5](https://placehold.co/15x15/ffedf5/ffedf5.png) |
| `primary-200` | `#ffe0ef` | ![#ffe0ef](https://placehold.co/15x15/ffe0ef/ffe0ef.png) |
| `primary-300` | `#ffb3d9` | ![#ffb3d9](https://placehold.co/15x15/ffb3d9/ffb3d9.png) |
| `primary-400` | `#fd3e97` | ![#fd3e97](https://placehold.co/15x15/fd3e97/fd3e97.png) |
| `primary-500` | `#e42068` | ![#e42068](https://placehold.co/15x15/e42068/e42068.png) |
| `primary-600` | `#ca2579` | ![#ca2579](https://placehold.co/15x15/ca2579/ca2579.png) |
| `primary-700` | `#be2371` | ![#be2371](https://placehold.co/15x15/be2371/be2371.png) |
| `primary-800` | `#591035` | ![#591035](https://placehold.co/15x15/591035/591035.png) |

#### 2. Grayscale (무채색 스케일)
| 토큰명 (Token) | Hex Code | 컬러 |
| :--- | :--- | :---: |
| `gray-100` | `#eaeaea` | ![#eaeaea](https://placehold.co/15x15/eaeaea/eaeaea.png) |
| `gray-200` | `#dedede` | ![#dedede](https://placehold.co/15x15/dedede/dedede.png) |
| `gray-300` | `#c8c8c8` | ![#c8c8c8](https://placehold.co/15x15/c8c8c8/c8c8c8.png) |
| `gray-400` | `#bebebe` | ![#bebebe](https://placehold.co/15x15/bebebe/bebebe.png) |
| `gray-500` | `#a5a5a5` | ![#a5a5a5](https://placehold.co/15x15/a5a5a5/a5a5a5.png) |
| `gray-600` | `#949494` | ![#949494](https://placehold.co/15x15/949494/949494.png) |
| `gray-700` | `#6f6f6f` | ![#6f6f6f](https://placehold.co/15x15/6f6f6f/6f6f6f.png) |
| `gray-800` | `#565656` | ![#565656](https://placehold.co/15x15/565656/565656.png) |

#### 3. Brand & Background (대비 및 배경 컬러)
| 토큰명 (Token) | Hex Code | 컬러 |
| :--- | :--- | :---: |
| `brand-white` | `#fdfdfe` | ![#fdfdfe](https://placehold.co/15x15/fdfdfe/fdfdfe.png) |
| `brand-dark`  | `#2d2d2d` | ![#2d2d2d](https://placehold.co/15x15/2d2d2d/2d2d2d.png) |
| `brand-black` | `#101010` | ![#101010](https://placehold.co/15x15/101010/101010.png) |
| `bg-sub`      | `#f7f7f7` | ![#f7f7f7](https://placehold.co/15x15/f7f7f7/f7f7f7.png) |
| `bg-base`     | `#f0f0f3` | ![#f0f0f3](https://placehold.co/15x15/f0f0f3/f0f0f3.png) |

### Typography

기본 폰트는 **Pretendard**를 사용하며, `Mobile/Desktop`에 따라 아래의 스케일을 적용합니다.

**Mobile (모바일)**

| 스타일명 | Weight | Style | Size | Line Height |
| --- | --- | --- | --- | --- |
| **main-m** | 700 | bold | 2rem | - |
| **display-m** | 700 | bold | 2rem | - |
| **h1-m** | 600 | semibold | 1.5rem | - |
| **h2-m** | 600 | semibold | 1.25rem | - |
| **body1-m** | 500 | medium | 1rem | 150% |
| **body2-m** | 500 | medium | 0.875rem | 150% |
| **caption-m** | 500 | medium | 0.75rem | 140% |

**Desktop (데스크탑)**

| 스타일명 | Weight | Style | Size | Line Height |
| --- | --- | --- | --- | --- |
| **display-d** | 700 | bold | 3rem | - |
| **h1-d** | 700 | bold | 2rem | - |
| **h2-d** | 600 | semibold | 1.5rem | - |
| **body1-d** | 600 | semibold | 1.125rem | 150% |
| **body2-d** | 500 | medium | 1rem | 150% |
| **caption-d** | 500 | medium | 0.8125rem | 140% |

---

### 프로젝트 구조도

```
.
├── apps
│   ├── admin
│   │   └── src
│   └── service
│       └── src
├── packages
│   └── shared
│       └── src
├── .gitignore
├── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── tsconfig.json
└── turbo.json
```

### 서브도메인 매핑

| 워크스페이스 | 서브도메인 | 설명 |
|------------|-----------|------|
| `apps/service` | `www.dabom.site` | 가족 사용자 서비스 (web-service) |
| `apps/admin` | `admin.dabom.site` | 백오피스 관리 (web-admin) |

각 애플리케이션은 독립적으로 빌드 및 배포되며, CloudFront + S3를 통해 각 서브도메인에 연결됩니다.

## 2. 워크스페이스 (Workspaces)

### `apps/admin`

관리자용 웹사이트를 위한 Next.js 애플리케이션입니다. (`admin.dabom.site`)

- **주요 역할**: 관리자 기능 제공 (가족 관리, 정책 관리 등)
- **기술 스택**: Next.js, React, TypeScript, Tailwind CSS
- **주요 의존성**:
  - `@repo/shared`: 공용 컴포넌트 및 로직 사용
  - `@tanstack/react-query`: 서버 상태 관리 및 데이터 페칭
  - `dayjs`: 날짜 및 시간 처리

### `apps/service`

사용자에게 실제 서비스를 제공하는 Next.js 애플리케이션입니다. (`www.dabom.site`)

- **주요 역할**: 사용자 서비스 제공 (데이터 사용량 대시보드, 마이페이지, 알림 등)
- **기술 스택**: Next.js, React, TypeScript, Tailwind CSS
- **주요 의존성**:
  - `@repo/shared`: 공용 컴포넌트 및 로직 사용
  - `@tanstack/react-query`: 서버 상태 관리 및 데이터 페칭
  - `chart.js`, `recharts`: 데이터 시각화를 위한 차트 라이브러리
  - `react-hot-toast`: 사용자 알림(토스트 메시지) 제공

### `packages/shared`

여러 애플리케이션에서 재사용되는 코드를 모아놓은 핵심 공유 패키지입니다.

- **주요 역할**: 공통 UI 컴포넌트, 유틸리티 함수, 타입 정의, 아이콘 등을 제공하여 코드 중복을 최소화하고 일관성을 유지합니다.
- **주요 내용**:
  - **UI Components**: `Button`, `Badge`, `InputField` 등 공용으로 사용되는 React 컴포넌트
  - **Utils**: `cn` (for Tailwind CSS), `http` (axios wrapper) 등 유용한 헬퍼 함수
  - **Types**: `familyType`, `policyType` 등 프로젝트 전반에서 사용되는 TypeScript 타입
  - **Assets**: SVG 아이콘을 React 컴포넌트로 변환하여 제공 (`svgr` 사용)
- **빌드**: `tsup`을 사용하여 패키지를 CommonJS와 ESM 형식으로 번들링합니다.
- **주요 의존성**:
  - `react-hook-form`, `zod`: 강력한 폼 핸들링 및 스키마 기반 유효성 검사
  - `clsx`, `tailwind-merge`: 조건부 Tailwind CSS 클래스를 쉽게 관리
---
## 공유 로직 (`packages/shared`)

`@repo/shared` 패키지는 이 프로젝트 아키텍처의 핵심입니다. 애플리케이션 간 코드 공유는 다음과 같이 이루어집니다.

1.  **컴포넌트 공유**: `Button`이나 `Modal` 같은 UI 컴포넌트를 `shared` 패키지에서 만들어 각 앱(`admin`, `service`)에서 `import { Button } from '@shared'`와 같이 가져와 사용합니다.
2.  **타입 공유**: API 응답이나 데이터 모델에 대한 타입을 `shared` 패키지에 정의하고, 각 앱에서는 이를 가져와 사용하여 타입 안전성을 보장합니다.
3.  **유틸리티 공유**: 날짜 포맷팅, 숫자 계산 등 공통 로직을 `shared` 패키지의 유틸리티 함수로 만들어 여러 곳에서 재사용합니다.

이러한 구조는 코드의 재사용성을 극대화하고, 애플리케이션 간의 UI/UX 일관성을 높이며, 새로운 기능을 더 빠르고 안정적으로 개발할 수 있도록 돕습니다.



---
## 백오피스 기능 명세서

### 유저 (User)

| 기능명 | 동작 내용 |
| --- | --- |
| **로그인** | 주어진 계정으로 로그인하여 백오피스에 접근할 수 있다. |

### 정책 (Policy)

| 기능명 | 동작 내용 |
| --- | --- |
| **정책 조회** | - 활성화 정책 (사용 중)<br>- 비활성화 정책 (만료)<br>두 가지 상태를 조회할 수 있다. |
| **정책 생성** | 정책을 새로 생성할 수 있다. |
| **정책 수정** | - 활성화 정책의 세부 사항을 수정한다.<br>- 활성화 정책을 비활성화 상태로 변경한다.<br>- 비활성화 정책을 활성화 상태로 변경한다. |
| **정책 삭제** | 비활성화된 정책을 영구 삭제한다. |

### 가족 (Family)

| 기능명 | 동작 내용 |
| --- | --- |
| **가족 리스트 조회** | - 전체 가족을 리스트로 조회한다.<br>된다. |
| **가족 정보 상세 조회** | - 가족 총 데이터 사용량 (매월 1일~말일 기준)<br>- 각 구성원별 상세 정보 {이름, 권한, 데이터 사용량, 데이터 한도}를 조회한다. |
| **구성원 권한 수정** | - 관리자가 특정 구성원의 owner 권한을 부여/해제(owner/member) 처리한다.<br>- 만 19세 이상인 member(자녀)의 권한 승격이 필요한 경우 관리자가 수동으로 수정한다. |
| **구성원 한도 수정** | 관리자가 CS(고객 지원) 및 시스템 관리 목적으로 각 구성원별 데이터 한도를 강제로 수정할 수 있다. |
| **가족 구성원 관리** | 관리자 권한으로 특정 가족 그룹에 구성원을 임의로 추가 / 수정 / 삭제한다. |
| **가족 검색** | 키워드를 사용하여 특정 가족을 검색한다. |
| **가족 필터링** | 최대 2가지 분류 기준을 조합하여 필터링한다.<br>(ex- 데이터를 50% 이상 사용한 가족) |

| 기능명 | 동작 내용 |
| --- | --- |
| **가족 리스트 조회** | - 전체 가족을 리스트로 조회할 수 있다.<br>
- 리스트에는 가족 중 대표자 1명만 조회 |
| **가족 정보 상세 조회** | - 가족 총 데이터 사용량
- 각 구성원별 { 이름, 권한, 데이터 사용량, 데이터 한도 } |
| **가족 정보 수정** (admin) | - owner 권한 수정 (true/false 두가지 상태로 수정할 수 있다.)
- member 권한 수정 (만 19세 이상인 자녀는 백오피스를 통해 권한을 가질 수 있다.) |
| **가족 정보 수정** (한도) | 각 구성원별 데이터 한도를 수정할 수 있다. |
| **가족 정보 수정** (구성원) | 가족 구성원 추가 / 수정 / 삭제 |
| **가족 검색** | 필터링 / 키워드를 사용하여 검색할 수 있다. |
| **가족 필터링** | 최대 2가지 분류에 대해 필터링을 할 수 있다.<br>(ex- 데이터를 50% 이상 사용한 가족) |
| **가족 키워드** | 가족ID, 전화번호, 이름을 통해 검색할 수 있다. |

---

## 유저 서비스 기능 명세서

### 유저

| 기능명 | 동작 내용 |
| --- | --- |
| **로그인** | 전화번호와 비밀번호를 입력 후 로그인 |
| **로그아웃** | 저장된 토큰 삭제 및 로그인 화면으로 이동 |
| **회원 탈퇴** | 특정 회원의 유저 정보를 영구 삭제한다. |

### 대시보드 (홈)

| 기능명 | 동작 내용 |
| --- | --- |
| **구성원별 데이터 사용량 조회** | - 현재 달에서는 SSE 수신 (실시간 사용량 조회)
- 과거 달에서는 http 통신으로 분기 처리 |

### 알림

| 기능명 | 동작 내용 |
| --- | --- |
| **누적 알림 조회** | 지금까지 사용자에게 누적된 알림을 확인한다. |
| **사용 경고** | 사용자에게 설정된 임계값에 따라 잔여량 경고를 한다. |
| **차단 알림** | - 데이터 잔여량이 없는 경우<br>- 잔여량이 있지만 차단된 경우 알림이 발송된다. |

### 마이페이지

| 기능명 | 동작 내용 |
| --- | --- |
| **이용 약관** | 서비스 이용 약관 및 개인정보 처리방침 텍스트 노출 |
| **내 데이터 사용량 조회** | 사용자의 데이터 사용량을 가로 막대그래프로 표시 |
| **내 정책 조회** | 현재 사용자에게 적용된 정책을 확인할 수 있음 |

### 정책 관리

| 기능명 | 동작 내용 |
| --- | --- |
| **가족 구성원 정책 조회** | - 각 구성원별로 현재 적용되어있는 정책을 확인할 수 있다. |
| **가족 구성원 정책 수정** | 각 구성원별로 현재 적용되어있는 정책을 변경할 수 있다. |
| **시간대별 데이터 사용 차단** | - 휠 피커를 통해 차단 시간 설정 |
| **데이터 한도 수정(슬라이더)** | - 슬라이더 조작 후 디바운싱을 이용하여 데이터 저장<br>- API 요청 실패 시 이전 상태로 롤백 처리 |
| **데이터 차단/해제(토글)** | 토글 버튼을 이용하여 데이터 사용을 차단하거나 해제한다. |

### 차단

| 기능명 | 동작 내용 |
| --- | --- |
| **글로벌 차단** | SSE로 차단 이벤트를 수신받은 즉시 toast창 띄움 |

---

### 미션/보상 (Mission & Reward)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **미션 카드 목록** | `GET /missions` | - 카드형 레이아웃으로 미션 목록 표시<br>- status별 필터 탭: ACTIVE / COMPLETED / CANCELLED<br>- 각 카드에 미션 텍스트 및 보상 정보 표시 |
| **미션 생성** | `POST /missions` | - 미션 텍스트 자유 입력<br>- 보상 템플릿 선택 (`GET /rewards/templates`)<br>- 템플릿의 썸네일/상품명/단가 표시<br>- OWNER만 접근 가능 |
| **보상 요청** | `POST /missions/{missionId}/request` | - ACTIVE 상태 미션에서 요청 버튼 노출<br>- MEMBER만 접근 가능 |
| **보상 승인/거절** | `PUT /rewards/requests/{id}/respond` | - OWNER가 MEMBER의 보상 요청에 승인 또는 거절 처리<br>- 거절 시 사유 입력 필드 표시 |
| **미션 삭제** | `DELETE /missions/{id}` | - OWNER만 삭제 가능<br>- 삭제 전 확인 다이얼로그 표시 |
| **미션 상태 로그** | `GET /missions/logs` | - 리스트형 타임라인 표시<br>- 미션 상태 변화(생성 / 요청 / 완료 / 취소)를 시간순으로 표시 |
| **미션 요청 이력** | `GET /missions/history` | - 리스트형 이력 표시<br>- 보상 요청의 처리 결과(PENDING / APPROVED / REJECTED)를 표시<br>- status 필터 제공 |

---

### 이의제기/긴급요청 (Appeal & Emergency)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **이의제기 목록** | `GET /appeals` | - type / status 필터 제공<br>- 페이지네이션 적용 |
| **조르기 요청** | `POST /appeals` | - 요청 데이터량 입력 (바이트 → MB 변환 UI 제공)<br>- 사유 입력<br>- MEMBER만 접근 가능 |
| **긴급 요청** | `POST /appeals/emergency` | - 100~300MB 범위 슬라이더 또는 직접 입력<br>- 사유 입력<br>- 월 1회 사용 여부 UI 표시<br>- MEMBER만 접근 가능 |
| **이의제기 상세** | - | - 요청 정보 상세 표시<br>- 코멘트 스레드 함께 표시 |
| **이의제기 코멘트** | `POST/GET /appeals/{id}/messages` | - 채팅형 코멘트 UI<br>- 메시지 입력 및 스크롤 목록 표시 |
| **이의제기 응답** | `PUT /appeals/{id}/respond` | - OWNER가 승인 또는 거절 처리<br>- 거절 시 사유 입력 필드 표시 |

---

### 월간 리캡 대시보드 (Monthly Recap)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **리캡 조회** | `GET /recaps/monthly` | - 연/월 선택 UI 제공<br>- 선택한 월의 전체 리캡 데이터 조회 |
| **사용량 섹션** | - | - 총 사용량 및 사용률 차트 표시<br>- 요일별 사용 패턴 막대 그래프<br>- 피크 시간대 표시 |
| **이의제기 현황 섹션** | - | - 총 이의제기 요청 건수<br>- 승인 건수<br>- 승인율 표시 |
| **긴급 요청 섹션** | - | - 총 긴급 승인 횟수<br>- 사용 자녀 수 표시 |
| **미션 보상 섹션** | - | - 완료 미션 수 표시<br>- `completed_mission_json` 기반 미션 보상 카드 목록 표시 |
| **소통 지수** | - | - 0~100 게이지 차트 표시<br>- 승인율(50%) / 이의제기활용도(30%) / 행동실행력(20%) 분해 표시 |
| **최다 승인 부모** | - | - 아이콘 및 이름 표시 |

---

### 프로필/약관 (Profile & Terms)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **프로필 조회** | `GET /customers/profile` | - 이름, 이메일, 전화번호, 프로필 이미지 표시 |
| **프로필 수정** | `PUT /customers/profile` | - 이름, 이메일, 프로필 이미지 URL 수정 가능 |
| **약관 조회** | `GET /terms` | - 약관 목록 텍스트 표시 |
| **약관 동의** | `POST /customers/terms` | - 체크박스 기반 동의 UI 제공 |

---

### 알림 관리 (Notification)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **알림 목록** | `GET /notifications` | - 알림 카드 리스트 표시<br>- 읽음 / 안읽음 상태 시각적 구분<br>- 타입별 아이콘 표시 |
| **개별 읽음 처리** | `PATCH /notifications/{notificationId}/read` | - 알림 카드 클릭 시 읽음 처리 |
| **전체 읽음 처리** | `PATCH /notifications/read-all` | - 버튼 클릭으로 전체 알림 읽음 처리 |

---

## 백오피스 기능 명세서 확장 (v11.0)

### 보상 템플릿 관리 (Reward Template)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **보상 템플릿 목록** | `GET /admin/rewards/templates` | - 테이블형 목록 표시<br>- ID, 유형(category), 썸네일(thumbnailUrl), 상품(name), 단가(price) 컬럼 표시. 수정 버튼 |
| **보상 템플릿 생성** | `POST /admin/rewards/templates` | - 생성 모달: 유형(기프티콘/데이터 라디오), 썸네일(이미지 업로드), 상품(name), 단가(price) 입력 |
| **보상 템플릿 수정** | `PUT /admin/rewards/templates/{id}` | - 수정 모달: 유형은 변경 불가, 썸네일/상품/단가 수정 가능. 삭제 버튼 포함 |
| **보상 템플릿 삭제** | `DELETE /admin/rewards/templates/{id}` | - 삭제 확인 다이얼로그 후 처리 |
| **지급 내역 목록** | `GET /admin/rewards/grants` | - 테이블: 전화번호, 사용여부, 상품, 지급미션, 쿠폰번호, 발급일, 만료일. 정렬: 최신지급순/만료임박순. 필터: 미사용내역만보기. 검색: 전화번호 |

### 감사 로그 (Audit Log)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **감사 로그 조회** | `GET /admin/audit/logs` | - 필터: action, entityType, actorId<br>- 테이블형 목록 표시<br>- 페이지네이션 적용 |

### 관리자 대시보드 (Admin Dashboard)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **전체 통계 요약** | `GET /admin/dashboard` | - 전체 가족 수, 사용자 수, 정책 수 등 통계 요약 카드 표시 |

### 가족 이름 수정 (Admin)

| 기능명 | API | 동작 내용 |
| --- | --- | --- |
| **가족 이름 수정** | `PUT /families` | - 관리자가 가족 이름을 수정할 수 있다. |
