# Codex Bootstrap Prompt

아래 내용을 Codex에게 전달한다.

---

Task: 놓치면 알려줘 프로젝트 초기 세팅

너는 이 저장소의 초기 세팅을 담당한다.

반드시 아래 문서를 먼저 읽고 작업하라.

- README.md
- docs/00-project-rules.md
- docs/01-bootstrap-target.md
- docs/02-domain-sketch.md
- docs/03-poc-roadmap.md

이번 작업의 목표는 정식 기능 구현이 아니라, POC 구현을 위한 백엔드/프론트 기본 구조를 세팅하는 것이다.

---

## 현재 상태

사용자가 이미 아래 프로젝트를 생성해둔다.

- backend: Spring Boot
- frontend: React + TypeScript + Vite

너는 생성된 프로젝트 내부에 필요한 패키지, 설정, 공통 구조, 기본 API, 기본 화면을 추가한다.

---

## Backend 작업 범위

### 1. 패키지 구조 생성

기본 패키지:

```text
com.noh.dontmissit

아래 패키지를 생성한다.

domain
controller
service
repository
dto
scheduler
fetcher
parser
diff
notification
admin
config
global.exception
global.response
global.util
2. 공통 응답 구조 생성
ApiResponse<T>
ErrorResponse
ErrorCode
BaseException
GlobalExceptionHandler
3. 공통 Entity 생성
BaseTimeEntity
createdAt
updatedAt
4. 설정 파일 정리

아래 파일을 정리한다.

application.yml
application-local.yml
application-prod.yml

필요 설정:

datasource
jpa
redis
mail placeholder
actuator
logging
5. Config 생성
WebConfig
RedisConfig
SchedulerConfig
JpaAuditingConfig
MailConfig placeholder
6. Health API 생성
GET /api/v1/health

응답 예시:

{
  "success": true,
  "data": {
    "status": "OK",
    "service": "dont-miss-it"
  }
}
7. POC 도메인 엔티티 뼈대 생성

아직 복잡한 비즈니스 로직은 구현하지 말고, Entity와 Repository 뼈대만 만든다.

WatchPage
WatchKeyword
PageSnapshot
ChangeEvent
ParseFailureLog
SourceHealth
NotificationLog
8. Enum 생성
WatchStatus
ChangeType
HealthStatus
NotificationStatus
9. Repository 생성
WatchPageRepository
WatchKeywordRepository
PageSnapshotRepository
ChangeEventRepository
ParseFailureLogRepository
SourceHealthRepository
NotificationLogRepository
Frontend 작업 범위
1. 기본 폴더 구조 생성
src/
├── api
├── components/common
├── components/layout
├── pages
├── routes
├── hooks
├── types
└── utils
2. Axios Client 생성
src/api/client.ts
VITE_API_BASE_URL 사용
3. Health API 연결
src/api/healthApi.ts
HomePage에서 health check 결과 확인
4. Router 구성

페이지:

HomePage
WatchPageCreatePage
WatchPageListPage
AdminDashboardPage
5. 기본 Layout 생성
Header
PageShell
Footer
6. Placeholder UI 생성

현재는 기능 구현보다 구조가 중요하다.

HomePage에는 서비스 설명과 링크 입력 CTA를 둔다.

문구 예시:

재입고·예약오픈·공지변경 놓치지 않게 알려드릴게요.
보고 있던 공식 페이지 링크만 붙여넣으세요.
이번 작업에서 하지 말 것
로그인 구현하지 말 것
OAuth 구현하지 말 것
실제 HTML Fetch 구현하지 말 것
실제 Diff 로직 구현하지 말 것
실제 이메일 발송 구현하지 말 것
관리자 대시보드를 완성하려 하지 말 것
AI 기능 추가하지 말 것
팝업 목록/추천/랭킹 기능 만들지 말 것
크롤링 대상 사이트 하드코딩하지 말 것

이번 작업은 POC 구현을 위한 기본 세팅까지만 한다.

작업 완료 후 보고할 것

작업 완료 후 아래 형식으로 보고하라.

생성한 백엔드 패키지 목록
생성한 백엔드 클래스 목록
생성한 Entity 목록
생성한 Repository 목록
생성한 API 목록
생성한 프론트 폴더/파일 목록
실행 방법
아직 구현하지 않은 것
다음 단계 제안

---

## 사용 방식

너는 이렇게 진행하면 돼.

1. Spring Boot 프로젝트 생성
2. React + TypeScript 프로젝트 생성
3. 위 `docs/` 파일들 추가
4. Codex에게 `docs/codex-bootstrap-prompt.md` 내용 전달
5. Codex가 기본 세팅 완료
6. 그 다음에 POC 기능을 한 단계씩 구현

지금 Codex에게 한 번에 “서비스 다 만들어줘”라고 하면 망할 가능성이 커.  
처음에는 **초기 세팅만** 시키고, 그 다음에 `HTML Fetch → Snapshot → Diff → ChangeEvent → Redis → Scheduler → Admin` 순서로 쪼개서 가는 게 좋아.
