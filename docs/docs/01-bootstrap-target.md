# Bootstrap Target

이 문서는 Codex가 초기 프로젝트 세팅을 할 때 따라야 하는 기준이다.

사용자는 이미 Spring Boot 백엔드와 React + TypeScript 프론트엔드 프로젝트를 생성해둔다.

Codex는 초기 세팅 단계에서 비즈니스 기능을 완성하려 하지 말고, 앞으로 POC를 구현하기 위한 기본 구조와 컨벤션만 잡는다.

---

## Backend 목표

백엔드는 `backend/` 디렉토리에 위치한다.

기본 패키지는 아래를 사용한다.

```text
com.noh.dontmissit
Backend 기술 스택
Java 21
Spring Boot
Spring Web
Spring Data JPA
Spring Validation
Spring Scheduler
PostgreSQL
Redis
Lombok
Jsoup
Spring Boot Mail
Spring Boot Actuator
Backend 기본 패키지 구조
backend/src/main/java/com/noh/dontmissit/
├── DontMissItApplication.java
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
Backend 초기 세팅 항목

Codex는 초기 세팅에서 아래를 구현한다.

공통 설정
application.yml
application-local.yml
application-prod.yml
PostgreSQL datasource 설정
Redis connection 설정
Mail 설정 placeholder
CORS 설정
Scheduler 활성화
Timezone 설정
Actuator 기본 설정
공통 응답
ApiResponse<T>
ErrorResponse
GlobalExceptionHandler
BaseException
ErrorCode
공통 Entity
BaseTimeEntity
createdAt
updatedAt
기본 Health API
GET /api/v1/health

응답 예시:

{
  "success": true,
  "data": {
    "status": "OK",
    "service": "dont-miss-it"
  }
}
Frontend 목표

프론트엔드는 frontend/ 디렉토리에 위치한다.

Frontend 기술 스택
React
TypeScript
Vite
Tailwind CSS
React Router
Axios
Zustand는 필요할 때만 사용한다.
Frontend 기본 구조
frontend/src/
├── api/
│   ├── client.ts
│   └── healthApi.ts
├── components/
│   ├── common/
│   └── layout/
├── pages/
│   ├── HomePage.tsx
│   ├── WatchPageCreatePage.tsx
│   ├── WatchPageListPage.tsx
│   └── AdminDashboardPage.tsx
├── routes/
│   └── AppRouter.tsx
├── hooks/
├── types/
├── utils/
├── App.tsx
└── main.tsx
Frontend 초기 세팅 항목

Codex는 초기 세팅에서 아래를 구현한다.

Axios client 생성
API base URL 환경변수 적용
React Router 설정
기본 Layout
HomePage
WatchPageCreatePage placeholder
WatchPageListPage placeholder
AdminDashboardPage placeholder
Health API 호출 테스트
Tailwind 기본 스타일 정리
Frontend 환경변수
VITE_API_BASE_URL=http://localhost:8080
초기 세팅에서 하지 말 것
로그인 구현 금지
OAuth 구현 금지
실제 알림 발송 구현 금지
복잡한 관리자 UI 구현 금지
도메인 로직 과구현 금지
AI 기능 추가 금지
결제 기능 추가 금지
크롤링 대상 사이트 하드코딩 금지

초기 세팅의 목표는 POC 구현을 위한 뼈대를 잡는 것이다.
