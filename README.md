# 놓치면 알려줘

> 관심 있는 팝업·굿즈·티켓·행사 공식 페이지를 저장하면,  
> 예약오픈·재입고·품절해제·운영시간 변경·장소 변경·공지 변경처럼  
> **사용자가 행동해야 하는 변화**가 생겼을 때 알려주는 알림 서비스

---

## 프로젝트 소개

**놓치면 알려줘**는 사용자가 이미 관심을 가진 공식 페이지를 등록하면, 해당 페이지를 주기적으로 확인하고 이전 상태와 비교하여 의미 있는 변화가 발생했을 때 알려주는 서비스입니다.

이 프로젝트는 새로운 팝업이나 티켓팅 정보를 추천하는 서비스가 아닙니다.

사용자가 이미 보고 있던 공식 페이지에서 다음과 같은 변화가 생겼을 때 알려주는 것이 핵심입니다.

- 굿즈가 재입고된 경우
- 팝업스토어 사전예약이 열린 경우
- 티켓팅 일정이나 공지가 변경된 경우
- 행사 운영시간이나 장소가 변경된 경우
- 품절 상태가 구매 가능 상태로 바뀐 경우
- 방문 전 확인해야 할 공지 문구가 바뀐 경우

즉, 이 서비스의 정체성은 다음과 같습니다.

> **내가 이미 관심 가진 공식 페이지의 행동 가능한 변화만 감지해주는 서비스**

---

## 개발 목적

이 프로젝트는 단순 CRUD 서비스가 아니라, 외부 페이지를 주기적으로 감시하고 상태 변화를 감지하는 **운영형 백엔드 시스템**을 구현하는 것을 목표로 합니다.

주요 구현 목표는 다음과 같습니다.

- 외부 페이지 HTML 수집
- Scheduler 기반 주기적 감시
- 페이지 Snapshot 저장
- 이전 Snapshot과 현재 Snapshot 비교
- 의미 있는 ChangeEvent 생성
- Redis Lock 기반 중복 실행 방지
- 동일 변경 중복 알림 방지
- Fetch / Parse 실패 관리
- Source Health 관리
- 알림 발송 및 실패 로그 관리
- 관리자용 운영 대시보드 구성
- 알림 클릭률 및 공식 페이지 이동률 수집

---

## 핵심 기능

### 사용자 기능

- 관심 있는 공식 페이지 URL 등록
- 감시할 변화 유형 선택
- 페이지 변경 감지 결과 조회
- 변경 발생 시 이메일 알림 수신
- 알림 클릭 후 공식 페이지 이동

### 운영 기능

- HTML Fetch
- Text Extraction
- PageSnapshot 저장
- Snapshot Diff 계산
- ChangeType 분류
- ChangeEvent 생성
- Redis Lock 기반 중복 감시 방지
- 동일 변경 중복 알림 방지
- Parse Failure 기록
- Source Health 관리
- 관리자용 운영 대시보드

---

## 감지 대상 ChangeType

초기 MVP에서는 아래 변경 유형을 우선 감지합니다.

| ChangeType | 설명 |
|---|---|
| `RESERVATION_OPEN` | 예약 오픈, 사전예약, 예매 오픈 문구 감지 |
| `SOLD_OUT_DETECTED` | 품절, 매진, SOLD OUT 문구 감지 |
| `RESTOCK_DETECTED` | 재입고, 구매 가능, 판매중 문구 감지 |
| `OPERATION_TIME_CHANGED` | 운영시간, 영업시간, 조기마감, 연장 운영 문구 변화 감지 |
| `NOTICE_CHANGED` | 공지, 안내, 유의사항, 현장 안내 변화 감지 |
| `PAGE_UNAVAILABLE` | 페이지 접근 실패 |
| `PARSE_FAILED` | 텍스트 추출 실패 |

---

## 사용자 흐름

1. 사용자가 관심 있는 공식 페이지를 발견합니다.
2. `놓치면 알려줘`에 접속합니다.
3. 공식 페이지 링크를 붙여넣습니다.
4. 서비스가 페이지 텍스트를 추출합니다.
5. 감시 가능한 키워드 또는 변화 유형을 선택합니다.
6. 서버가 주기적으로 페이지를 Fetch합니다.
7. 이전 Snapshot과 현재 Snapshot을 비교합니다.
8. 의미 있는 변화가 감지되면 ChangeEvent를 생성합니다.
9. 중복 알림 여부를 확인합니다.
10. 사용자에게 알림을 발송합니다.
11. 사용자는 알림을 클릭해 공식 페이지로 이동합니다.

---

## 시스템 흐름

```text
User
  ↓
WatchPage 등록
  ↓
Scheduler
  ↓
HTML Fetcher
  ↓
Text Extractor
  ↓
PageSnapshot 저장
  ↓
Diff Engine
  ↓
ChangeType Detector
  ↓
ChangeEvent 생성
  ↓
Duplicate Guard
  ↓
Notification Sender
  ↓
Event Logging
  ↓
Admin Dashboard
```

---

## 기술 스택

### Backend

- Java 21
- Spring Boot
- Spring Data JPA
- Spring Scheduler
- PostgreSQL
- Redis

### Frontend

- React
- TypeScript
- Tailwind CSS

### Infra

- Docker
- Docker Compose
- Nginx
- GitHub Actions

---

## 주요 도메인

| 도메인 | 설명 |
|---|---|
| `WatchPage` | 사용자가 감시 등록한 페이지 |
| `WatchKeyword` | 감시할 키워드와 ChangeType |
| `PageSnapshot` | 특정 시점에 추출한 페이지 텍스트 |
| `ChangeEvent` | 이전 Snapshot과 현재 Snapshot을 비교해 생성된 변경 이벤트 |
| `NotificationLog` | 알림 발송 결과와 클릭 여부 |
| `ParseFailureLog` | Fetch 또는 Parse 실패 기록 |
| `SourceHealth` | 도메인별 수집 성공률과 실패 상태 |
| `AdminMetric` | 관리자 대시보드용 운영 지표 |

---

## API 개요

### 사용자 API

```http
POST   /api/v1/watch-pages
GET    /api/v1/watch-pages
GET    /api/v1/watch-pages/{id}
DELETE /api/v1/watch-pages/{id}

POST   /api/v1/watch-pages/{id}/keywords
GET    /api/v1/watch-pages/{id}/changes

POST   /api/v1/notifications/subscribe
POST   /api/v1/events/notification-click
POST   /api/v1/events/official-link-click
```

### 관리자 API

```http
POST /api/v1/admin/watch-pages/{id}/fetch
GET  /api/v1/admin/change-events
GET  /api/v1/admin/parse-failures
GET  /api/v1/admin/source-health
GET  /api/v1/admin/notification-logs
POST /api/v1/admin/watch-pages/{id}/refresh
```

---

## Redis 활용

| Key | 목적 |
|---|---|
| `watch:page:lock:{watchPageId}` | 동일 페이지 중복 Fetch 방지 |
| `watch:snapshot:{watchPageId}` | 최근 Snapshot 캐시 |
| `notify:sent:{watchPageId}:{changeType}:{contentHash}` | 동일 변경 중복 알림 방지 |
| `source:rate:{domain}:{yyyyMMddHHmm}` | 도메인별 호출 제한 |
| `source:health:{domain}` | 도메인 상태 캐시 |

---

## Scheduler 작업

| 주기 | 작업 |
|---|---|
| 10~30분 | 활성 WatchPage Fetch |
| 1시간 | 실패 페이지 재시도 |
| 매일 00:30 | 오래된 Snapshot 정리 |
| 매일 01:00 | 일별 운영 지표 집계 |
| 매일 09:00 | 오늘 변경 요약 알림 |
| 수동 | 관리자 강제 새로고침 |

---

## Fetch / Parse 정책

### Fetch 정책

- Timeout 설정
- User-Agent 설정
- 3xx Redirect 허용
- 4xx / 5xx 응답은 실패 로그 저장
- 도메인별 Rate Limit 적용
- 동일 페이지 중복 Fetch 방지

### Parse 정책

- `script`, `style`, `nav`, `footer` 제거
- 본문 텍스트 중심으로 추출
- 너무 짧은 텍스트는 `PARSE_FAILED` 처리
- 동적 렌더링으로 본문이 비어 있으면 `PARSE_FAILED` 처리
- 추출 텍스트는 `contentHash`로 비교

---

## 장애 대응

| 상황 | 대응 |
|---|---|
| 외부 페이지 Fetch 실패 | 실패 로그 저장, 실패 횟수 증가, 재시도 |
| Timeout 발생 | 도메인 timeoutCount 증가, 감시 주기 완화 |
| Parse 실패 | ParseFailureLog 저장, 동적 페이지 의심 여부 기록 |
| 중복 알림 발생 | Redis Key와 DB 제약으로 중복 차단 |
| Scheduler 중복 실행 | Redis Lock 사용 |
| 특정 도메인 실패율 증가 | SourceHealth 경고 표시, 감시 주기 완화 |

---

## 관리자 대시보드

관리자 대시보드는 단순 CRUD 관리자가 아니라, 서비스 운영 상태를 확인하기 위한 화면입니다.

### Source Health

- 도메인
- 최근 성공 시각
- 최근 실패 시각
- Fetch 성공률
- Parse 실패율
- Timeout 횟수
- Health 상태

### Change Event

- 페이지명
- 감지 유형
- 이전 문장
- 변경 문장
- 감지 시각
- 알림 발송 여부

### Notification Log

- 발송 성공률
- 실패 사유
- 재시도 횟수
- 클릭 여부
- 공식 페이지 이동 여부

### 운영 지표

- URL 등록 수
- 활성 감시 페이지 수
- 변경 감지 수
- 알림 클릭률
- 공식 페이지 이동률
- 중복 알림 차단 수
- Scheduler 실패 수
- Redis Lock 충돌 수

---

## 실행 방법

### Backend

```bash
cd backend
./gradlew bootRun
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Docker Compose

```bash
docker-compose up -d
```

---

## 환경 변수 예시

```env
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/dontmissit
SPRING_DATASOURCE_USERNAME=dontmissit
SPRING_DATASOURCE_PASSWORD=dontmissit

SPRING_REDIS_HOST=localhost
SPRING_REDIS_PORT=6379

MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your-email@example.com
MAIL_PASSWORD=your-password
```

---

## 프로젝트 구조

```text
dont-miss-it/
├── backend/
│   └── src/main/java/com/noh/dontmissit/
│       ├── domain/
│       ├── controller/
│       ├── service/
│       ├── repository/
│       ├── scheduler/
│       ├── fetcher/
│       ├── parser/
│       ├── diff/
│       ├── notification/
│       ├── admin/
│       └── global/
│
├── frontend/
│   └── src/
│       ├── api/
│       ├── components/
│       ├── pages/
│       ├── hooks/
│       ├── types/
│       └── utils/
│
├── docs/
├── docker-compose.yml
└── README.md
```

---

## 상세 문서

상세 설계는 `docs/` 디렉토리에 분리합니다.

```text
docs/
├── architecture.md
├── domain-model.md
├── api-spec.md
├── redis-scheduler.md
├── operation-policy.md
├── metrics.md
├── roadmap.md
└── poc-result.md
```

---

## 개발 단계

### 1단계. POC

- URL 등록
- HTML Fetch
- Text Extraction
- PageSnapshot 저장
- Diff 계산
- ChangeEvent 생성
- Fetch / Parse 실패 로그 저장

### 2단계. MVP

- Scheduler 적용
- Redis Lock 적용
- 이메일 알림
- 중복 알림 방지
- SourceHealth 관리
- 관리자 대시보드

### 3단계. 운영 개선

- 알림 클릭률 분석
- 공식 페이지 이동률 수집
- ChangeType별 반응률 분석
- 도메인별 실패율 기반 감시 주기 조정
- 카카오 / 문자 알림 실험

---

## 포트폴리오 포인트

이 프로젝트를 통해 다음과 같은 백엔드 운영 경험을 보여줄 수 있습니다.

- 외부 페이지 수집 실패와 Timeout 대응
- Scheduler 기반 주기적 작업 처리
- Snapshot Diff 기반 상태 변경 감지
- Redis Lock 기반 중복 실행 방지
- 동일 변경 중복 알림 방지
- SourceHealth 기반 도메인별 상태 관리
- ParseFailureLog 기반 실패 케이스 분리
- 알림 발송 성공률과 클릭률 수집
- 관리자 대시보드를 통한 운영 상태 확인

---

## 프로젝트 방향성

이 프로젝트가 흔들리지 말아야 할 기준은 하나입니다.

> 추천 / 목록 / 랭킹으로 가지 않고,  
> **감시 / 변경 / 알림 / 운영**에 집중합니다.
