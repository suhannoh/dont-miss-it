# Codex Bootstrap Prompt

## 작업 목적

이 저장소는 `놓치면 알려줘` 프로젝트입니다.

이 프로젝트는 팝업·굿즈·티켓·행사 정보를 추천하거나 모아주는 서비스가 아닙니다.

핵심은 사용자가 이미 관심 가진 공식 페이지를 등록하면, 해당 페이지를 주기적으로 감시하고, 예약오픈·재입고·품절해제·운영시간 변경·장소 변경·공지 변경처럼 **사용자가 행동해야 하는 변화**가 생겼을 때 알려주는 운영형 알림 서비스입니다.

이번 작업의 목표는 정식 기능 구현이 아닙니다.

이번 작업은 POC 구현을 위한 **초기 세팅**만 진행합니다.

---

## 반드시 먼저 읽을 문서

작업 전 아래 문서를 순서대로 읽고, 문서의 방향을 어기지 마세요.

1. `README.md`
2. `docs/00-project-rules.md`
3. `docs/01-bootstrap-target.md`
4. `docs/02-domain-sketch.md`
5. `docs/03-poc-roadmap.md`
6. `docs/04-env-guide.md`
7. `docs/05-external-api-prep.md`
8. `docs/06-codex-workflow.md`

문서를 읽은 뒤, 이번 작업 범위가 **초기 세팅**인지 다시 확인하고 시작하세요.

---

## 현재 전제

사용자는 이미 아래 프로젝트를 생성해둡니다.

```text
dont-miss-it/
├── backend/     # Spring Boot 프로젝트
├── frontend/    # React + TypeScript + Vite 프로젝트
├── docs/
├── README.md
├── docker-compose.yml
└── .env.example
```

Codex는 이미 생성된 프로젝트 내부에 필요한 패키지, 설정, 공통 구조, 기본 API, 기본 화면을 추가합니다.

---

## 이번 작업에서 반드시 할 것

### Backend

기본 패키지는 아래를 사용합니다.

```text
com.noh.dontmissit
```

아래 패키지 구조를 생성합니다.

```text
backend/src/main/java/com/noh/dontmissit/
├── domain/
├── controller/
├── service/
├── repository/
├── dto/
├── scheduler/
├── fetcher/
├── parser/
├── diff/
├── notification/
├── admin/
├── config/
└── global/
    ├── exception/
    ├── response/
    └── util/
```

---

### Backend 공통 구조

아래 공통 클래스를 생성합니다.

```text
global/response/ApiResponse.java
global/response/ErrorResponse.java
global/exception/ErrorCode.java
global/exception/BaseException.java
global/exception/GlobalExceptionHandler.java
domain/BaseTimeEntity.java
```

`BaseTimeEntity`에는 아래 필드를 둡니다.

```text
createdAt
updatedAt
```

JPA Auditing 설정을 추가합니다.

---

### Backend 설정

아래 설정 파일을 정리합니다.

```text
application.yml
application-local.yml
application-prod.yml
```

설정은 `.env` 값을 직접 읽는 방식이 아니라, Spring의 환경변수 placeholder를 사용합니다.

예:

```yaml
spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL}
    username: ${SPRING_DATASOURCE_USERNAME}
    password: ${SPRING_DATASOURCE_PASSWORD}
```

아래 설정을 포함합니다.

- datasource
- jpa
- redis
- mail placeholder
- actuator
- logging
- scheduler
- timezone

---

### Backend Config

아래 Config 클래스를 생성합니다.

```text
config/WebConfig.java
config/RedisConfig.java
config/SchedulerConfig.java
config/JpaAuditingConfig.java
config/MailConfig.java
```

`MailConfig`는 실제 발송 구현까지 하지 말고, 이후 이메일 알림을 붙일 수 있는 placeholder 수준으로 둡니다.

---

### Backend Health API

아래 API를 생성합니다.

```http
GET /api/v1/health
```

응답 예시:

```json
{
  "success": true,
  "data": {
    "status": "OK",
    "service": "dont-miss-it"
  }
}
```

---

### Backend POC 도메인 뼈대

이번 단계에서는 복잡한 비즈니스 로직을 구현하지 않습니다.

Entity와 Repository 뼈대만 생성합니다.

생성할 Entity:

```text
WatchPage
WatchKeyword
PageSnapshot
ChangeEvent
ParseFailureLog
SourceHealth
NotificationLog
```

생성할 Enum:

```text
WatchStatus
ChangeType
HealthStatus
NotificationStatus
```

생성할 Repository:

```text
WatchPageRepository
WatchKeywordRepository
PageSnapshotRepository
ChangeEventRepository
ParseFailureLogRepository
SourceHealthRepository
NotificationLogRepository
```

---

## ChangeType 초기 값

초기 ChangeType은 아래만 사용합니다.

```text
RESERVATION_OPEN
SOLD_OUT_DETECTED
RESTOCK_DETECTED
OPERATION_TIME_CHANGED
NOTICE_CHANGED
PAGE_UNAVAILABLE
PARSE_FAILED
```

---

## Frontend 작업 범위

아래 폴더 구조를 생성합니다.

```text
frontend/src/
├── api/
├── components/
│   ├── common/
│   └── layout/
├── pages/
├── routes/
├── hooks/
├── types/
└── utils/
```

---

### Frontend 기본 파일

아래 파일을 생성합니다.

```text
src/api/client.ts
src/api/healthApi.ts

src/routes/AppRouter.tsx

src/components/layout/Header.tsx
src/components/layout/Footer.tsx
src/components/layout/PageShell.tsx

src/pages/HomePage.tsx
src/pages/WatchPageCreatePage.tsx
src/pages/WatchPageListPage.tsx
src/pages/AdminDashboardPage.tsx
```

---

### Frontend 기본 요구사항

- Axios client를 생성합니다.
- `VITE_API_BASE_URL` 환경변수를 사용합니다.
- React Router를 설정합니다.
- HomePage에서 Health API 호출 결과를 확인할 수 있게 합니다.
- 기본 Layout을 구성합니다.
- Tailwind CSS 기본 스타일을 정리합니다.

HomePage의 핵심 문구는 아래 방향을 사용합니다.

```text
재입고·예약오픈·공지변경 놓치지 않게 알려드릴게요.
보고 있던 공식 페이지 링크만 붙여넣으세요.
```

---

## .env 사용 원칙

중요 변수는 `.env`로 관리합니다.

단, 실제 `.env` 파일은 Git에 커밋하지 않습니다.

반드시 `.env.example`만 커밋합니다.

Backend는 Spring 환경변수 placeholder를 사용합니다.

Frontend는 Vite 규칙에 따라 `VITE_` prefix가 붙은 환경변수만 사용합니다.

---

## 이번 작업에서 절대 하지 말 것

이번 작업은 초기 세팅만 합니다.

아래 작업은 하지 마세요.

- 로그인 구현 금지
- OAuth 구현 금지
- 실제 HTML Fetch 구현 금지
- 실제 Text Extraction 구현 금지
- 실제 Diff 로직 구현 금지
- 실제 Scheduler 비즈니스 로직 구현 금지
- 실제 이메일 발송 구현 금지
- 관리자 대시보드 완성 금지
- AI 기능 추가 금지
- 결제 기능 추가 금지
- 팝업 목록 기능 구현 금지
- 티켓팅 일정 통합 기능 구현 금지
- 굿즈 검색 기능 구현 금지
- 인기 랭킹 구현 금지
- 커뮤니티 구현 금지
- 특정 크롤링 대상 사이트 하드코딩 금지
- 인스타그램, X, 네이버 예약 등 폐쇄/동적 플랫폼 크롤링 구현 금지

---

## 이번 작업 완료 기준

아래가 되면 초기 세팅은 완료입니다.

1. Backend 패키지 구조가 생성되어 있다.
2. 공통 응답 구조가 생성되어 있다.
3. 전역 예외 처리 구조가 생성되어 있다.
4. BaseTimeEntity와 JPA Auditing이 설정되어 있다.
5. application 설정이 환경변수 기반으로 정리되어 있다.
6. Redis, Mail, Scheduler, CORS, JPA Config가 준비되어 있다.
7. Health API가 동작한다.
8. POC Entity와 Repository 뼈대가 생성되어 있다.
9. Frontend 기본 라우팅이 구성되어 있다.
10. HomePage에서 Health API를 호출할 수 있다.
11. `.env.example` 기준으로 환경변수 목록이 정리되어 있다.

---

## 작업 완료 후 보고 형식

작업 완료 후 아래 형식으로 보고하세요.

```md
## 작업 완료 보고

### 1. 생성/수정한 파일

### 2. 생성한 백엔드 패키지

### 3. 생성한 공통 클래스

### 4. 생성한 Entity

### 5. 생성한 Enum

### 6. 생성한 Repository

### 7. 생성한 API

### 8. 생성한 Frontend 파일

### 9. 환경변수 사용 방식

### 10. 실행 방법

### 11. 아직 구현하지 않은 것

### 12. 다음 단계 제안
```

---

## 다음 단계

초기 세팅 이후에는 아래 순서로 작업합니다.

1. HTML Fetcher 구현
2. Text Extractor 구현
3. PageSnapshot 저장
4. Snapshot Diff 계산
5. ChangeType 감지
6. ChangeEvent 생성
7. Redis Lock 적용
8. Scheduler 적용
9. Email Notification 적용
10. Admin Dashboard 구현

## Docker 작업 범위

이 프로젝트는 Docker를 사용한다.

초기 세팅 단계에서 Codex는 로컬 개발과 POC 실행을 위한 Docker 기본 파일을 생성한다.

생성할 파일:

```text
docker-compose.yml
backend/Dockerfile
frontend/Dockerfile
infra/nginx/default.conf
.env.example
```

Docker Compose에는 아래 서비스를 포함한다.

- postgres
- redis
- backend
- frontend
- nginx

단, 이번 단계에서는 운영 배포용 HTTPS, Certbot, AWS 배포, GitHub Actions 배포는 구현하지 않는다.

---

## Docker 관련 주의사항

- 실제 `.env` 파일은 생성하지 않는다.
- `.env.example`만 생성한다.
- `.env`는 `.gitignore`에 포함한다.
- Backend 컨테이너에서 PostgreSQL host는 `postgres`를 사용한다.
- Backend 컨테이너에서 Redis host는 `redis`를 사용한다.
- Frontend 컨테이너는 Nginx로 정적 파일을 서빙한다.
- 루트 Nginx는 `/api/` 요청을 backend로 프록시한다.
- 루트 Nginx는 `/` 요청을 frontend로 프록시한다.

---

## Docker 완료 기준

아래 명령으로 전체 서비스가 실행 가능해야 한다.

```bash
docker-compose up -d --build
```

아래 주소가 동작해야 한다.

```text
http://localhost
http://localhost/api/v1/health
http://localhost/actuator/health
```
