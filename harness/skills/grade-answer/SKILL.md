---
name: grade-answer
description: Grade a user's short Korean quiz answer against the current question without exposing hidden answer data early.
---

# Grade Answer

## Purpose

사용자의 답을 직전 문항 기준으로 채점하고 짧은 해설을 제공한다.

## Use This Skill For

```text
1번
3
O
X
모르겠어
패스
해설 보여줘
```

## Workflow

1. 직전 문항 ID를 확인한다.
2. 사용자의 답변을 정규화한다.
3. `모르겠어`, `패스`, `해설`이면 skipped로 처리하고 해설을 제공한다.
4. 객관식은 번호와 `answer.choiceId`를 비교한다.
5. fact_check는 O/X 판정과 사용자가 고친 부분을 비교한다.
6. 정답 여부, 정답, 핵심 암기 포인트, 틀린 선택지 이유를 짧게 말한다.
7. 세션 상태의 correct/wrong/skipped와 weakTags를 갱신한다.
8. 다음 문제를 낼지 자연스럽게 이어간다.

## Answer Normalization

```text
1, 1번, 정답 1번 -> choiceId "1"
O, o, 오, 맞아 -> correct verdict
X, x, 엑스, 틀려 -> incorrect verdict
모르겠어, 몰라, 패스, 해설 -> skipped
```

## Output Format

정답:

```text
맞았습니다.
핵심은 {암기 포인트}입니다.

다음 문제로 갈게요.
```

오답:

```text
아쉽지만 정답은 {정답}입니다.
핵심은 {암기 포인트}입니다.
헷갈린 부분은 {짧은 비교}입니다.

다음 문제로 갈까요?
```

## Constraints

- 장황한 강의보다 시험 포인트를 우선한다.
- evidenceRefs는 사용자가 출처를 요청한 경우에만 보여준다.
