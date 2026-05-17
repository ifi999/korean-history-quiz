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
6. fill_blank, short_answer는 `answer.text`, `answer.acceptableAnswers`, `answer.keyTerms`와 의미가 맞는지 비교한다.
7. matching은 사용자가 연결한 항목이 `answer.pairs` 또는 해설의 핵심 관계와 맞는지 비교한다.
8. 정답 여부, 정답, 핵심 암기 포인트, 틀린 선택지 이유를 짧게 말한다.
9. 세션 상태의 correct/wrong/skipped와 weakTags를 갱신한다.
10. 다음 문제를 낼지 자연스럽게 이어간다.

## Answer Normalization

```text
1, 1번, 정답 1번 -> choiceId "1"
O, o, 오, 맞아 -> correct verdict
X, x, 엑스, 틀려 -> incorrect verdict
모르겠어, 몰라, 패스, 해설 -> skipped
빈칸/단답 -> 띄어쓰기와 일부 조사 차이는 무시하고 핵심 명사가 맞으면 정답 처리
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
- 수업자료 기반 fill_blank/short_answer는 exact string matching만으로 오답 처리하지 않는다. 예를 들어 `흥선대원군의 국내 정치`의 답으로 `흥선대원군`처럼 핵심 개념을 맞히면 정답 또는 부분 정답으로 인정한다.
- 사용자가 핵심 개념은 맞혔지만 범위가 좁거나 넓으면 `거의 맞았습니다`로 처리하고 정확한 표현을 알려준다.
