# Codex Workflow Guide

이 문서는 Codex에게 작업을 맡길 때의 순서를 정의한다.

한 번에 전체 서비스를 만들라고 지시하지 않는다.

작업은 반드시 작은 단위로 나눈다.

---

## 전체 작업 순서

1. Bootstrap
2. HTML Fetcher
3. Text Extractor
4. PageSnapshot
5. Snapshot Diff
6. ChangeType Detector
7. ChangeEvent
8. Redis Lock
9. Scheduler
10. Email Notification
11. Admin Dashboard
12. Deployment

---

## 1. Bootstrap

목표:

- 프로젝트 기본 구조 생성
- 공통 응답/예외 처리
- Entity/Repository 뼈대
- Health API
- Frontend 기본 라우팅

프롬프트:

```text
docs/codex-bootstrap-prompt.md를 읽고 초기 세팅만 진행하세요.
```

---

## 2. HTML Fetcher

목표:

- URL로 HTML 가져오기
- timeout
- user-agent
- redirect
- 실패 로그

하지 말 것:

- Diff 계산
- Scheduler
- 알림

---

## 3. Text Extractor

목표:

- HTML에서 본문 텍스트 추출
- script/style/nav/footer 제거
- 너무 짧은 텍스트 실패 처리
- contentHash 생성

---

## 4. PageSnapshot

목표:

- 추출 텍스트를 PageSnapshot으로 저장
- WatchPage 기준 최신 Snapshot 조회
- 동일 contentHash면 변경 없음 처리

---

## 5. Snapshot Diff

목표:

- 이전 Snapshot과 현재 Snapshot 비교
- 변경 문장 후보 추출
- 너무 지저분한 diff는 관리자 로그로 확인 가능하게 처리

---

## 6. ChangeType Detector

목표:

아래 ChangeType을 키워드 기반으로 감지한다.

```text
RESERVATION_OPEN
SOLD_OUT_DETECTED
RESTOCK_DETECTED
OPERATION_TIME_CHANGED
NOTICE_CHANGED
```

---

## 7. ChangeEvent

목표:

- ChangeEvent 생성
- 이전/현재 Snapshot 연결
- matchedKeyword 저장
- beforeText / afterText 저장
- contentHash 저장

---

## 8. Redis Lock

목표:

- 같은 WatchPage 중복 Fetch 방지
- 동일 변경 이벤트 중복 생성 방지

Redis Key:

```text
watch:page:lock:{watchPageId}
notify:sent:{watchPageId}:{changeType}:{contentHash}
```

---

## 9. Scheduler

목표:

- 활성 WatchPage 주기적 Fetch
- 실패 페이지 재시도
- Skip / Success / Failure 로그 저장

---

## 10. Email Notification

목표:

- ChangeEvent 발생 시 이메일 알림 발송
- NotificationLog 저장
- 실패 시 재시도 가능하게 기록

---

## 11. Admin Dashboard

목표:

- SourceHealth 조회
- ParseFailureLog 조회
- ChangeEvent 조회
- NotificationLog 조회
- 관리자 강제 Fetch

---

## 12. Deployment

목표:

- Dockerfile
- docker-compose.yml
- Nginx
- GitHub Actions
- 배포 환경변수 정리

---

## Codex에게 지시할 때의 원칙

- 한 번에 하나의 단계만 요청한다.
- 이전 단계가 동작하는지 확인한 뒤 다음 단계로 간다.
- “전체 구현해줘”라고 하지 않는다.
- “리팩터링까지 알아서 해줘”라고 하지 않는다.
- 새 기능을 추가하기 전에 문서를 먼저 업데이트한다.
- 구현 후 반드시 변경 파일 목록과 테스트 방법을 보고하게 한다.

---

## 작업 완료 보고 형식

모든 Codex 작업은 아래 형식으로 보고해야 한다.

```md
## 작업 완료 보고

### 1. 요약

### 2. 변경 파일

### 3. 주요 구현 내용

### 4. 실행 방법

### 5. 테스트 방법

### 6. 아직 구현하지 않은 것

### 7. 다음 작업 제안
```
