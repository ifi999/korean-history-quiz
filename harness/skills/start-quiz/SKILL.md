---
name: start-quiz
description: Start a one-question-at-a-time Korean history quiz session using only approved question data.
---

# Start Quiz

## Purpose

승인된 문항으로 대화형 퀴즈 세션을 시작한다.

## Use This Skill For

```text
퀴즈 시작
문제 내줘
한국사 해보자
랜덤으로 내줘
전근대사 객관식 10문제
파이널1 OX 풀자
```

## Workflow

1. 필요한 경우 `source-scope-picker`를 먼저 적용한다.
2. `data/catalog/quiz-manifest.json`의 approvedQuestionSets만 사용한다.
3. 범위와 유형에 맞는 문항 후보를 고른다.
4. 한 문제만 출력한다.
5. 정답, 해설, evidenceRefs는 숨긴다.
6. 사용자의 답변을 기다린다.
7. 답변이 오면 `grade-answer` 흐름으로 이어간다.

## Output Format

객관식:

```text
{scopeLabel}으로 {targetQuestionCount}문제 진행할게요.
문제 1입니다.

{prompt}

1. {choice}
2. {choice}
...

답은 번호만 말해도 됩니다.
```

fact_check:

```text
파이널1 O/X로 진행할게요.
문제 1입니다.

다음 문장이 맞으면 O, 틀리면 X로 답하세요.
가능하면 틀린 부분도 함께 고쳐보세요.

"{statement}"
```

## Constraints

- 한 번에 여러 문제를 한꺼번에 보여주지 않는다.
- draft 문항은 기본 출제하지 않는다.
- `파이널2` 요청은 `keyword-card-review`로 넘긴다.
