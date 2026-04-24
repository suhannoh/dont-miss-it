# Environment Variable Guide

이 프로젝트는 중요한 설정값을 `.env`로 관리합니다.

실제 `.env` 파일은 Git에 커밋하지 않습니다.

반드시 `.env.example`만 커밋합니다.

---

## 파일 구성

추천 구조는 아래와 같습니다.

```text
dont-miss-it/
├── .env.example
├── .env                 # 로컬 Docker Compose용, Git 커밋 금지
├── backend/
└── frontend/
    ├── .env.example
    └── .env.local       # Vite 로컬 실행용, Git 커밋 금지
```

---

## Root `.env`

루트 `.env`는 Docker Compose 실행 시 사용합니다.

예시:

```env
# App
APP_NAME=dont-miss-it
APP_PROFILE=local
APP_TIMEZONE=Asia/Seoul

# Backend
BACKEND_PORT=8080

# Frontend
FRONTEND_PORT=5173

# PostgreSQL
POSTGRES_DB=dontmissit
POSTGRES_USER=dontmissit
POSTGRES_PASSWORD=dontmissit
POSTGRES_PORT=5432

# Spring Datasource
SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/dontmissit
SPRING_DATASOURCE_USERNAME=dontmissit
SPRING_DATASOURCE_PASSWORD=dontmissit

# Redis
SPRING_REDIS_HOST=redis
SPRING_REDIS_PORT=6379

# Mail
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your-email@example.com
MAIL_PASSWORD=your-mail-password
MAIL_FROM=no-reply@example.com

# CORS
CORS_ALLOWED_ORIGINS=http://localhost:5173

# Fetch Policy
FETCH_TIMEOUT_MILLIS=5000
FETCH_USER_AGENT=DontMissItBot/0.1
FETCH_MIN_TEXT_LENGTH=100

# Scheduler
WATCH_FETCH_INTERVAL_MS=1800000
FAILED_RETRY_INTERVAL_MS=3600000

# Redis TTL
REDIS_WATCH_LOCK_TTL_SECONDS=300
REDIS_SNAPSHOT_TTL_SECONDS=1800
REDIS_NOTIFY_DEDUP_TTL_SECONDS=86400
REDIS_SOURCE_HEALTH_TTL_SECONDS=600
```

---

## Root `.env.example`

`.env.example`은 실제 값이 아니라 예시 값만 넣습니다.

```env
APP_NAME=dont-miss-it
APP_PROFILE=local
APP_TIMEZONE=Asia/Seoul

BACKEND_PORT=8080
FRONTEND_PORT=5173

POSTGRES_DB=dontmissit
POSTGRES_USER=dontmissit
POSTGRES_PASSWORD=dontmissit
POSTGRES_PORT=5432

SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/dontmissit
SPRING_DATASOURCE_USERNAME=dontmissit
SPRING_DATASOURCE_PASSWORD=dontmissit

SPRING_REDIS_HOST=redis
SPRING_REDIS_PORT=6379

MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your-email@example.com
MAIL_PASSWORD=your-mail-password
MAIL_FROM=no-reply@example.com

CORS_ALLOWED_ORIGINS=http://localhost:5173

FETCH_TIMEOUT_MILLIS=5000
FETCH_USER_AGENT=DontMissItBot/0.1
FETCH_MIN_TEXT_LENGTH=100

WATCH_FETCH_INTERVAL_MS=1800000
FAILED_RETRY_INTERVAL_MS=3600000

REDIS_WATCH_LOCK_TTL_SECONDS=300
REDIS_SNAPSHOT_TTL_SECONDS=1800
REDIS_NOTIFY_DEDUP_TTL_SECONDS=86400
REDIS_SOURCE_HEALTH_TTL_SECONDS=600
```

---

## Frontend `.env.local`

Vite는 `VITE_` prefix가 붙은 변수만 클라이언트에서 사용할 수 있습니다.

```env
VITE_API_BASE_URL=http://localhost:8080
VITE_APP_NAME=놓치면 알려줘
```

---

## Backend 설정 방식

Spring Boot는 `.env` 파일을 자동으로 읽지 않습니다.

따라서 Backend는 `application.yml`에서 환경변수 placeholder를 사용합니다.

예시:

```yaml
spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL}
    username: ${SPRING_DATASOURCE_USERNAME}
    password: ${SPRING_DATASOURCE_PASSWORD}

  data:
    redis:
      host: ${SPRING_REDIS_HOST}
      port: ${SPRING_REDIS_PORT}

app:
  fetch:
    timeout-millis: ${FETCH_TIMEOUT_MILLIS:5000}
    user-agent: ${FETCH_USER_AGENT:DontMissItBot/0.1}
    min-text-length: ${FETCH_MIN_TEXT_LENGTH:100}
```

로컬 IntelliJ 실행 시에는 Run Configuration에 환경변수를 직접 넣거나, Docker Compose로 실행합니다.

---

## Git 관리 규칙

Git에 커밋할 파일:

```text
.env.example
frontend/.env.example
```

Git에 커밋하지 않을 파일:

```text
.env
.env.local
frontend/.env.local
backend/.env
backend/.env.local
```

`.gitignore`에 아래를 추가합니다.

```gitignore
.env
.env.*
!.env.example

frontend/.env
frontend/.env.*
!frontend/.env.example

backend/.env
backend/.env.*
!backend/.env.example
```
