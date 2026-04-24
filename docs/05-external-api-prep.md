# External API Preparation

이 문서는 프로젝트 진행 전 준비해야 할 외부 API와 계정 정보를 정리한다.

---

## 결론

POC 단계에서는 별도의 외부 API Key가 거의 필요 없다.

POC는 아래 기능만 검증한다.

- URL Fetch
- Text Extraction
- Snapshot 저장
- Diff 계산
- ChangeEvent 생성
- 실패 로그 저장
- SourceHealth 관리

즉, POC는 SMTP나 카카오 알림톡 없이도 진행 가능하다.

---

## 단계별 필요한 외부 준비물

| 단계 | 필요한 외부 준비물 | 필수 여부 |
|---|---|---|
| POC | 없음 | 필수 아님 |
| MVP 1차 | SMTP 계정 | 선택 |
| MVP 2차 | 이메일 발송 계정 | 권장 |
| 확장 | 카카오 알림톡 또는 문자 발송 API | 선택 |
| 확장 | Web Push VAPID Key | 선택 |
| 운영 | 도메인, HTTPS, Nginx | 권장 |

---

## 1. URL Fetch

URL Fetch에는 별도의 API Key가 필요 없다.

단, 아래 정책을 반드시 지킨다.

- User-Agent 설정
- Timeout 설정
- 도메인별 Rate Limit
- robots.txt 및 사이트 정책 확인
- 로그인 필요한 페이지 감시 금지
- 자동 구매/자동 예약 금지
- 과도한 요청 금지
- Fetch 실패 시 재시도 제한

---

## 2. Email SMTP

MVP에서 실제 알림을 보내려면 SMTP 계정이 필요하다.

선택지:

- Gmail SMTP
- Naver SMTP
- Daum/Kakao SMTP
- SendGrid
- Mailgun
- AWS SES

초기 MVP에서는 Gmail SMTP 또는 Naver SMTP 정도로 충분하다.

필요 변수:

```env
MAIL_HOST=
MAIL_PORT=
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_FROM=
```

주의:

- Gmail은 앱 비밀번호가 필요할 수 있다.
- 개인 메일 계정으로 대량 발송하지 않는다.
- 초기 테스트는 본인 이메일로만 진행한다.
- 운영 전에는 발송 제한과 스팸 정책을 확인한다.

---

## 3. 카카오 알림톡

카카오 알림톡은 MVP 필수가 아니다.

도입 난이도가 있다.

필요할 수 있는 것:

- 카카오 비즈니스 채널
- 알림톡 발송 대행사
- 템플릿 등록 및 승인
- 발송 비용
- 수신 동의 처리

초기 MVP에서는 제외한다.

8주차 이후 사용자 반응이 있을 때 검토한다.

---

## 4. 문자 SMS

문자 발송은 MVP 필수가 아니다.

선택지:

- NCloud SENS
- CoolSMS
- Toast SMS
- AWS SNS

필요할 수 있는 변수:

```env
SMS_PROVIDER=
SMS_ACCESS_KEY=
SMS_SECRET_KEY=
SMS_SENDER_PHONE=
```

초기 MVP에서는 제외한다.

---

## 5. Web Push

웹푸시는 선택사항이다.

필요 준비물:

- VAPID Public Key
- VAPID Private Key
- Service Worker
- 브라우저 알림 권한

필요 변수:

```env
WEB_PUSH_PUBLIC_KEY=
WEB_PUSH_PRIVATE_KEY=
WEB_PUSH_SUBJECT=
```

MVP에서는 이메일 알림이 더 단순하다.

---

## 6. Playwright / 동적 페이지 처리

POC에서는 Playwright를 기본 도입하지 않는다.

먼저 Jsoup 기반 HTML Fetch와 Text Extraction으로 검증한다.

Playwright는 아래 조건에서만 검토한다.

- 정적 HTML에서 본문 추출 실패율이 너무 높은 경우
- 꼭 감시해야 하는 페이지가 동적 렌더링인 경우
- Fetch 비용과 실행 시간이 감당 가능한 경우

주의:

- Playwright는 서버 리소스를 더 많이 사용한다.
- Scheduler와 함께 사용하면 부하가 커질 수 있다.
- 무리하게 동적 페이지를 감시하려 하지 않는다.

---

## 7. 외부 API 준비 우선순위

초기 준비 우선순위는 아래와 같다.

### POC 전

필수 준비 없음.

### MVP 전

- SMTP 계정
- 운영용 도메인
- HTTPS 설정
- Docker/Nginx 배포 환경

### 확장 전

- 카카오 알림톡 또는 문자 발송 API
- Web Push
- 알림 수신 동의 정책

---

## 지금 미리 준비하면 좋은 것

1. GitHub repository
2. Docker Desktop
3. PostgreSQL / Redis Docker Compose
4. SMTP 테스트 계정
5. 운영용 도메인 후보
6. 배포 서버 또는 EC2
7. 테스트용 정적 페이지 URL 10개
8. 실패 테스트용 동적 페이지 URL 5개

---

## 테스트 URL 선정 기준

POC 테스트용 URL은 아래처럼 분리한다.

### 성공 기대 URL

- 정적 HTML 페이지
- 공지사항 페이지
- 상품 상세 페이지
- 행사 안내 페이지
- 예약 안내 페이지

### 실패 기대 URL

- 로그인 필요 페이지
- JavaScript 렌더링 중심 페이지
- 무한 스크롤 페이지
- 봇 차단 페이지
- SNS 페이지

실패 케이스도 중요하다.

실패를 성공처럼 숨기지 말고, ParseFailureLog와 SourceHealth로 관리한다.
