# Domain Sketch

이 문서는 POC에서 사용할 핵심 도메인 초안이다.

초기에는 모든 필드를 완벽하게 구현하지 않아도 된다.
단, 도메인 이름과 책임은 유지한다.

---

## WatchPage

사용자가 감시 등록한 공식 페이지다.

역할:

- 감시 대상 URL 저장
- 현재 감시 상태 관리
- 마지막 Fetch 성공/실패 시각 관리
- 실패 횟수 관리

주요 필드:

```text
id
url
title
domain
status
lastFetchedAt
lastSuccessAt
lastFailedAt
failureCount
createdAt
updatedAt

상태 예시:

ACTIVE
PAUSED
UNSTABLE
FAILED
WatchKeyword

감시할 키워드와 ChangeType을 저장한다.

역할:

사용자가 어떤 변화를 감시할지 설정
특정 키워드와 변경 유형 연결

주요 필드:

id
watchPageId
keyword
changeType
createdAt
PageSnapshot

특정 시점에 추출한 페이지 텍스트 스냅샷이다.

역할:

이전 상태와 현재 상태 비교를 위한 기준 데이터
contentHash 기반 변경 여부 판단

주요 필드:

id
watchPageId
contentHash
extractedText
fetchedAt
createdAt
ChangeEvent

이전 Snapshot과 현재 Snapshot을 비교해 생성된 변경 이벤트다.

역할:

행동 가능한 변화 기록
알림 발송 대상
관리자 대시보드에서 확인할 운영 이벤트

주요 필드:

id
watchPageId
changeType
previousSnapshotId
currentSnapshotId
matchedKeyword
beforeText
afterText
contentHash
notificationSent
createdAt
ChangeType

초기 POC에서는 아래 유형만 사용한다.

RESERVATION_OPEN
SOLD_OUT_DETECTED
RESTOCK_DETECTED
OPERATION_TIME_CHANGED
NOTICE_CHANGED
PAGE_UNAVAILABLE
PARSE_FAILED
ParseFailureLog

Fetch 또는 Parse 실패 기록이다.

역할:

동적 페이지, 차단 페이지, timeout, 파싱 실패를 추적
SourceHealth 통계의 근거 데이터

주요 필드:

id
watchPageId
url
reason
httpStatus
errorMessage
occurredAt
SourceHealth

도메인별 수집 성공률과 실패 상태를 관리한다.

역할:

특정 도메인의 fetch 성공률 관리
parse failure / timeout / failure count 관리
관리자 대시보드에서 source 상태 표시

주요 필드:

id
domain
successCount
failureCount
parseFailureCount
timeoutCount
lastSuccessAt
lastFailedAt
healthStatus
updatedAt

상태 예시:

HEALTHY
UNSTABLE
DOWN
NotificationLog

알림 발송 결과를 기록한다.

초기 POC에서는 실제 알림 발송을 하지 않아도 된다.
MVP 단계에서 이메일 알림과 함께 구현한다.

주요 필드:

id
changeEventId
channel
status
sentAt
failedReason
clickedAt
officialLinkClickedAt
AdminMetric

관리자 대시보드용 운영 지표다.

초기에는 별도 테이블 없이 집계 API로 처리해도 된다.
MVP 이후 일별 집계 테이블로 분리할 수 있다.

예시 지표:

watchPageCount
activeWatchPageCount
changeEventCount
parseFailureCount
fetchSuccessRate
notificationClickRate
officialLinkClickRate
