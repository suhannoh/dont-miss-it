# Docker Guide

이 프로젝트는 Docker Compose 기반으로 로컬 개발 환경과 배포 환경을 구성한다.

초기 목표는 아래 서비스들을 Docker로 실행할 수 있게 만드는 것이다.

- PostgreSQL
- Redis
- Backend
- Frontend
- Nginx

---

## 기본 원칙

- 로컬 개발에서는 PostgreSQL과 Redis를 Docker로 실행한다.
- Backend와 Frontend는 로컬 실행과 Docker 실행을 모두 지원한다.
- 실제 `.env` 파일은 Git에 커밋하지 않는다.
- `.env.example`만 커밋한다.
- Docker Compose는 루트 디렉토리에서 실행한다.
- 초기 POC 단계에서는 복잡한 운영 배포보다 로컬 재현성을 우선한다.

---

## 서비스 구성

```text
Client
  ↓
Nginx
  ↓
Frontend / Backend
  ↓
PostgreSQL / Redis
```

---

## docker-compose.yml

루트 디렉토리에 위치한다.

```yaml
services:
  postgres:
    image: postgres:16
    container_name: dont-miss-it-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      TZ: ${APP_TIMEZONE}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - dont_miss_it_network

  redis:
    image: redis:7
    container_name: dont-miss-it-redis
    restart: unless-stopped
    ports:
      - "${REDIS_PORT}:6379"
    networks:
      - dont_miss_it_network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: dont-miss-it-backend
    restart: unless-stopped
    depends_on:
      - postgres
      - redis
    environment:
      SPRING_PROFILES_ACTIVE: ${APP_PROFILE}
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB}
      SPRING_DATASOURCE_USERNAME: ${POSTGRES_USER}
      SPRING_DATASOURCE_PASSWORD: ${POSTGRES_PASSWORD}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PORT: 6379
      MAIL_HOST: ${MAIL_HOST}
      MAIL_PORT: ${MAIL_PORT}
      MAIL_USERNAME: ${MAIL_USERNAME}
      MAIL_PASSWORD: ${MAIL_PASSWORD}
      MAIL_FROM: ${MAIL_FROM}
      CORS_ALLOWED_ORIGINS: ${CORS_ALLOWED_ORIGINS}
      FETCH_TIMEOUT_MILLIS: ${FETCH_TIMEOUT_MILLIS}
      FETCH_USER_AGENT: ${FETCH_USER_AGENT}
      FETCH_MIN_TEXT_LENGTH: ${FETCH_MIN_TEXT_LENGTH}
    ports:
      - "${BACKEND_PORT}:8080"
    networks:
      - dont_miss_it_network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        VITE_API_BASE_URL: ${VITE_API_BASE_URL}
    container_name: dont-miss-it-frontend
    restart: unless-stopped
    depends_on:
      - backend
    ports:
      - "${FRONTEND_PORT}:80"
    networks:
      - dont_miss_it_network

  nginx:
    image: nginx:1.27
    container_name: dont-miss-it-nginx
    restart: unless-stopped
    depends_on:
      - frontend
      - backend
    ports:
      - "${NGINX_PORT}:80"
    volumes:
      - ./infra/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - dont_miss_it_network

volumes:
  postgres_data:

networks:
  dont_miss_it_network:
    driver: bridge
```

---

## Backend Dockerfile

`backend/Dockerfile`

```dockerfile
FROM gradle:8.10.2-jdk21 AS builder
WORKDIR /app
COPY . .
RUN chmod +x gradlew || true
RUN ./gradlew clean bootJar -x test

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## Frontend Dockerfile

`frontend/Dockerfile`

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app

ARG VITE_API_BASE_URL
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:1.27-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## Nginx 설정

`infra/nginx/default.conf`

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://frontend:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/ {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /actuator/ {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## .env.example

루트 디렉토리에 둔다.

```env
# App
APP_NAME=dont-miss-it
APP_PROFILE=local
APP_TIMEZONE=Asia/Seoul

# Ports
BACKEND_PORT=8080
FRONTEND_PORT=5173
NGINX_PORT=80
POSTGRES_PORT=5432
REDIS_PORT=6379

# PostgreSQL
POSTGRES_DB=dontmissit
POSTGRES_USER=dontmissit
POSTGRES_PASSWORD=dontmissit

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
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://localhost

# Frontend
VITE_API_BASE_URL=http://localhost/api

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

## 실행 방법

### 1. `.env` 생성

```bash
cp .env.example .env
```

### 2. Docker Compose 실행

```bash
docker-compose up -d --build
```

### 3. 로그 확인

```bash
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f nginx
```

### 4. 서비스 접속

```text
Frontend: http://localhost
Backend Health: http://localhost/api/v1/health
Actuator Health: http://localhost/actuator/health
```

### 5. 종료

```bash
docker-compose down
```

### 6. 볼륨까지 삭제

```bash
docker-compose down -v
```

---

## 로컬 개발 방식

초기 개발 중에는 아래 방식도 허용한다.

```text
PostgreSQL / Redis → Docker
Backend → IntelliJ local run
Frontend → npm run dev
```

이 경우 Backend 환경변수는 IntelliJ Run Configuration에 직접 넣는다.

Frontend는 `frontend/.env.local`을 사용한다.

```env
VITE_API_BASE_URL=http://localhost:8080
```

---

## Docker 사용 시 주의사항

- Backend 컨테이너 내부에서 PostgreSQL host는 `localhost`가 아니라 `postgres`다.
- Backend 컨테이너 내부에서 Redis host는 `localhost`가 아니라 `redis`다.
- Frontend Docker build 시 `VITE_API_BASE_URL`은 build arg로 들어간다.
- `.env`는 Git에 커밋하지 않는다.
- 운영 배포에서는 `POSTGRES_PASSWORD`, `MAIL_PASSWORD`를 반드시 변경한다.
- 운영 배포에서는 HTTPS 설정을 추가해야 한다.
- 운영 배포에서는 Nginx reverse proxy와 SSL 인증서를 별도로 구성한다.

---

## 초기 Codex 작업 범위

Codex가 Docker 관련해서 처음 해야 할 일은 아래까지만이다.

- `docker-compose.yml` 생성
- `backend/Dockerfile` 생성
- `frontend/Dockerfile` 생성
- `infra/nginx/default.conf` 생성
- `.env.example` 생성
- `.gitignore`에 `.env` 제외 규칙 추가
- Docker 실행 방법 README 또는 docs에 반영

Codex는 초기 세팅에서 운영 배포용 HTTPS, Certbot, AWS 배포, GitHub Actions 배포까지 한 번에 구현하지 않는다.
