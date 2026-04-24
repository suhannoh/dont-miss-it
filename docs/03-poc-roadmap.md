# POC Roadmap

이 문서는 1주 POC 구현 순서를 정의한다.

POC의 목표는 정식 서비스를 완성하는 것이 아니다.

목표는 다음 질문에 답하는 것이다.

> 사용자가 입력한 URL에서 텍스트를 안정적으로 가져오고, 이전 스냅샷과 비교해 의미 있는 변경을 감지할 수 있는가?

---

## POC 핵심 검증 항목

| 검증 항목 | 통과 기준 |
|---|---|
| HTML Fetch | 정적 페이지를 안정적으로 수집할 수 있음 |
| Text Extraction | 본문/공지/상품 상태 문구를 추출할 수 있음 |
| Snapshot 저장 | 이전/현재 상태 비교가 가능함 |
| Diff 계산 | 변경된 핵심 문장을 확인할 수 있음 |
| Keyword Detection | 예약, 품절, 재입고, 운영시간, 공지 관련 문구 감지 가능 |
| ChangeEvent 생성 | 의미 있는 변화가 이벤트로 저장됨 |
| 중복 방지 | 같은 변경이 반복 이벤트로 생성되지 않음 |
| 실패 로그 | Fetch 실패, Timeout, Parse 실패가 기록됨 |
| SourceHealth | 도메인별 성공/실패 상태 확인 가능 |

---

## Day 1

목표:

- 기본 도메인 생성
- URL 등록 API 구현

작업:

- WatchPage Entity 생성
- PageSnapshot Entity 생성
- ChangeEvent Entity 생성
- ParseFailureLog Entity 생성
- SourceHealth Entity 생성
- WatchPageRepository 생성
- URL 등록 API 구현

완료 기준:

- URL을 등록할 수 있다.
- 등록된 URL 목록을 조회할 수 있다.

---

## Day 2

목표:

- HTML Fetch 구현

작업:

- HtmlFetcher 구현
- Timeout 설정
- User-Agent 설정
- Redirect 허용
- HTTP status 기록
- Fetch 실패 시 ParseFailureLog 저장

완료 기준:

- URL을 입력하면 HTML을 가져올 수 있다.
- 실패한 URL은 실패 로그로 남는다.

---

## Day 3

목표:

- Text Extraction 구현

작업:

- Jsoup 기반 TextExtractor 구현
- script/style/nav/footer 제거
- 본문 텍스트 추출
- contentHash 생성
- PageSnapshot 저장

완료 기준:

- HTML에서 의미 있는 텍스트를 추출할 수 있다.
- Snapshot이 DB에 저장된다.
- 너무 짧은 텍스트는 PARSE_FAILED로 처리된다.

---

## Day 4

목표:

- Snapshot Diff 구현

작업:

- 이전 Snapshot 조회
- 현재 Snapshot과 contentHash 비교
- hash가 같으면 변경 없음 처리
- hash가 다르면 Diff 수행
- 변경 문장 추출

완료 기준:

- 이전 상태와 현재 상태를 비교할 수 있다.
- 변경된 문장 후보를 확인할 수 있다.

---

## Day 5

목표:

- ChangeType 감지와 ChangeEvent 생성

작업:

- ChangeType enum 생성
- 키워드 기반 ChangeTypeDetector 구현
- RESERVATION_OPEN 감지
- SOLD_OUT_DETECTED 감지
- RESTOCK_DETECTED 감지
- OPERATION_TIME_CHANGED 감지
- NOTICE_CHANGED 감지
- ChangeEvent 생성

완료 기준:

- 변경 문장에서 ChangeType을 분류할 수 있다.
- ChangeEvent가 DB에 저장된다.

---

## Day 6

목표:

- Redis Lock과 중복 이벤트 방지

작업:

- Redis 설정
- watch:page:lock:{watchPageId} 적용
- notify:sent:{watchPageId}:{changeType}:{contentHash} 적용
- 동일 변경 이벤트 중복 생성 방지

완료 기준:

- 같은 WatchPage에 대한 중복 Fetch가 방지된다.
- 같은 변경 내용으로 ChangeEvent가 반복 생성되지 않는다.

---

## Day 7

목표:

- 관리자 조회 API와 POC 결과 정리

작업:

- ChangeEvent 조회 API
- ParseFailureLog 조회 API
- SourceHealth 조회 API
- 강제 Fetch API
- 테스트 URL 성공/실패 케이스 정리
- POC 한계 정리

완료 기준:

- 관리자 API로 변경 이벤트와 실패 로그를 확인할 수 있다.
- POC 결과를 문서로 남길 수 있다.

---

## POC 실패 조건

아래 중 2개 이상이면 프로젝트 방향을 재검토한다.

- 대부분의 페이지가 동적 렌더링이라 텍스트 추출이 안 됨
- Diff 결과가 너무 지저분해 알림 가치가 없음
- 키워드 기반 감지가 너무 부정확함
- 사이트 차단 또는 Timeout이 잦음
- URL 입력 UX가 너무 어색함
- 이메일 알림까지 연결해도 사용 시나리오가 약함

---

## POC 이후 MVP 확장

POC가 통과하면 4주 MVP에서는 아래를 구현한다.

- Scheduler 적용
- 이메일 알림
- NotificationLog
- 알림 클릭 이벤트
- 공식 페이지 이동 이벤트
- 관리자 대시보드
- Docker / Nginx / GitHub Actions 배포
