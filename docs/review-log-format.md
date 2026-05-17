# Review Log Format

오답 기록은 친구가 여러 날 반복 복습을 원할 때만 사용한다. 기본 학습 세션은 대화 안에서만 오답을 기억한다.

## Storage

개인 기록은 Git에 올리지 않는다.

```text
data/review/local-attempts.jsonl
data/review/session-YYYY-MM-DD.jsonl
```

`.gitignore`는 `data/review/` 아래 개인 기록을 무시한다. 문서와 템플릿만 추적한다.

## When To Write

Codex는 사용자가 명시적으로 요청한 경우에만 파일에 기록한다.

트리거:

```text
오답 기록해줘
오늘 기록 저장해줘
복습 기록 남겨줘
내 오답 저장해줘
```

파일 쓰기 전에는 짧게 확인한다.

```text
오늘 세션 오답을 로컬 파일에 저장할게요. 개인 기록이라 Git에는 올리지 않는 구조입니다.
```

## Attempt Record

한 줄에 한 시도 기록을 저장한다.

```json
{
  "schemaVersion": 1,
  "recordType": "attempt",
  "attemptedAt": "2026-05-17T21:30:00+09:00",
  "sessionId": "2026-05-17-evening",
  "questionId": "premodern-5h-mc-017",
  "sourceId": "premodern-5h",
  "questionType": "multiple_choice",
  "userAnswer": "4",
  "correctAnswer": "5",
  "result": "wrong",
  "weakTags": ["고려", "광종", "과거제"],
  "memo": "광종과 공민왕 업적을 혼동함"
}
```

## Session Summary

세션 종료 시 사용자가 저장을 원하면 요약 기록을 남길 수 있다.

```json
{
  "schemaVersion": 1,
  "recordType": "session_summary",
  "endedAt": "2026-05-17T21:45:00+09:00",
  "sessionId": "2026-05-17-evening",
  "scopeLabel": "전근대 객관식",
  "totalCount": 10,
  "correctCount": 7,
  "wrongCount": 2,
  "skippedCount": 1,
  "weakTags": ["고려 왕 업적", "통일 신라 경제"],
  "nextReviewSuggestion": "광종/성종/공민왕 업적 비교 5문제"
}
```

## Result Values

```text
correct
wrong
skipped
partial
```

## Rules

- 사용자가 답하기 전에는 정답을 기록하지 않는다.
- `draft` 문항은 기본 오답 기록 대상이 아니다.
- 카드 복습은 채점형 결과가 아니므로 attempt 대신 `card_review` 기록을 쓸 수 있다.
- 개인 기록은 Git에 올리지 않는다.
