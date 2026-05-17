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
3. 범위와 유형에 맞는 문항 후보를 만든 뒤 balanced random으로 고른다.
4. 한 문제만 출력한다.
5. 정답, 해설, evidenceRefs는 숨긴다.
6. 사용자의 답변을 기다린다.
7. 답변이 오면 `grade-answer` 흐름으로 이어간다.

## Selection Strategy

기본값은 순차 출제가 아니라 골고루 섞는 출제다.

```text
1. 후보 문항을 sourceId, chapterIds, questionType, difficulty, tags로 나눈다.
2. 전체 범위 요청이면 approvedQuestionSets를 균등하게 순환한다.
3. 특정 범위 요청이면 해당 범위 안에서 itemNumber/pageNumber 연속 출제를 피한다.
4. 같은 세션의 usedQuestionIds는 제외한다.
5. 오답 복습 요청이 아닌 이상 weakTags만 과도하게 반복하지 않는다.
6. 사용자가 "처음부터", "순서대로"라고 명시한 경우에만 itemNumber 순서를 따른다.
```

현재 approved 세트 기준 예시:

```text
전체 랜덤 10문제
  -> final-1 fact_check, premodern-5h 객관식/순서 배열, modern-4h 객관식을 섞는다.
  -> final-1 문항 수가 가장 많더라도 final-1만 연속으로 내지 않는다.

전근대사 객관식 10문제
  -> premodern-5h 안에서 고대/고려/조선 태그와 페이지가 몰리지 않게 섞는다.
```

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

fill_blank / short_answer:

```text
수업자료 기반 개념 회상으로 진행할게요.
문제 1입니다.

{prompt}

답은 핵심 용어나 사건명만 말해도 됩니다.
```

## Constraints

- 한 번에 여러 문제를 한꺼번에 보여주지 않는다.
- draft 문항은 기본 출제하지 않는다.
- `파이널2` 요청은 `keyword-card-review`로 넘긴다.
- 같은 세션에서 같은 문제를 반복하지 않는다. 오답 복습 요청은 예외다.
- 기본 출제에서 JSONL 파일 순서를 그대로 따라가지 않는다.
- `modern-4h.source-recall`과 `premodern-5h.source-recall`은 수업자료에서 확실히 추출된 개념만 사용한 fill_blank 문항이다. 추측형 문항으로 취급하지 않는다.
