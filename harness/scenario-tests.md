# Harness Scenario Tests

이 문서는 친구가 실제로 말할 법한 문장을 기준으로 Codex가 어떤 흐름을 선택해야 하는지 점검한다.

## Scenario 1. Plain Start

User:

```text
문제 내줘
```

Expected route:

```text
source-scope-picker -> start-quiz
```

Expected behavior:

- approved 전체에서 최대 10문제를 목표로 한다.
- 한 문제만 낸다.
- 정답과 해설은 숨긴다.
- 파일명, JSONL, manifest를 설명하지 않는다.

## Scenario 2. Scoped Objective Quiz

User:

```text
전근대사 객관식 10문제
```

Expected route:

```text
source-scope-picker -> start-quiz
```

Expected behavior:

- `premodern-5h` approved 객관식/순서 배열 문항을 우선한다.
- 선택지가 있는 문제를 한 문제씩 낸다.
- 사용자가 `3번`처럼 답하면 `grade-answer`로 이어진다.

## Scenario 2-1. Balanced Random Quiz

User:

```text
랜덤으로 골고루 10문제 내줘
```

Expected route:

```text
source-scope-picker -> start-quiz
```

Expected behavior:

- approved 전체에서 고른다.
- JSONL 파일 순서나 itemNumber 순서대로 내지 않는다.
- `final-1` 문항 수가 가장 많더라도 `final-1`만 연속 출제하지 않는다.
- 가능한 범위에서 source set, chapter, tag, questionType을 섞는다.
- 같은 세션에서 이미 낸 문제는 다시 내지 않는다.

## Scenario 3. Fact Check

User:

```text
파이널1 OX로 해줘
```

Expected route:

```text
source-scope-picker -> start-quiz
```

Expected behavior:

- `final-1.fact-check` approved 문항을 사용한다.
- O/X와 틀린 부분 수정을 요구한다.
- 답변 전 정답 문장과 correction을 숨긴다.

## Scenario 4. Short Answer

Context:

Codex가 객관식 문제를 낸 직후.

User:

```text
3
```

Expected route:

```text
grade-answer
```

Expected behavior:

- 직전 문항의 답으로 해석한다.
- 정답 여부와 핵심 암기 포인트를 짧게 말한다.
- 다음 문제로 이어갈지 묻는다.

## Scenario 5. Skip Or Explanation

Context:

Codex가 문제를 낸 직후.

User:

```text
모르겠어
```

Expected route:

```text
grade-answer skipped flow
```

Expected behavior:

- 강제 오답으로 몰지 않는다.
- 정답과 해설을 보여준다.
- 세션 상태에는 skipped로 남긴다.

## Scenario 6. Wrong Note Review

User:

```text
방금 틀린 것 다시 내줘
```

Expected route:

```text
wrong-note-review
```

Expected behavior:

- 현재 대화 세션의 wrongQuestionIds를 우선한다.
- 같은 문제 또는 같은 weakTags의 approved 문항을 낸다.
- 저장된 오답 파일은 사용자가 요청하지 않으면 열지 않는다.

## Scenario 7. Keyword Card Review

User:

```text
파이널2 키워드 보여줘
```

Expected route:

```text
keyword-card-review
```

Expected behavior:

- `data/cards/approved/final-2.keyword-card.jsonl`을 사용한다.
- 채점형 문제가 아니라 암기 카드 복습이라고 짧게 안내한다.
- 한 번에 3-5개 카드만 보여준다.

## Scenario 8. Admin Request

User:

```text
새 PDF로 문항 만들어줘
```

Expected route:

```text
content-curator / AGENTS.md PDF ingestion workflow
```

Expected behavior:

- 친구 학습 모드에서 빠져나온다.
- PDF 구조 확인, 페이지별 추출, 정제, 상태 기록을 따른다.
- 단순 정리본은 웹 검증 없이 source_extracted로 처리한다.

## Pass Criteria

- 모든 일반 학습 요청은 한 문제씩 진행한다.
- `파이널2`는 카드 복습으로만 처리한다.
- 짧은 답변은 직전 문제의 답으로 처리한다.
- 오답 저장은 명시 요청 전에는 파일에 쓰지 않는다.
