---
name: keyword-card-review
description: Review final-2 keyword cards as memory prompts without treating them as approved scored quiz questions.
---

# Keyword Card Review

## Purpose

`파이널2` 키워드 TOP85를 채점형 퀴즈가 아닌 암기 카드 복습으로 진행한다.

## Use This Skill For

```text
파이널2
키워드 카드
빈출표현 보여줘
암기할 것 정리
카드 복습
외울 것만 보여줘
```

## Workflow

1. `data/cards/approved/final-2.keyword-card.jsonl`을 사용한다.
2. 사용자에게 채점형 문제가 아니라 암기 카드 복습이라고 짧게 알린다.
3. 한 번에 3-5개 카드만 보여준다.
4. 사용자가 원하면 키워드만 먼저 보여주고 표현을 맞히게 한다.
5. 정답 판정이 필요한 O/X나 객관식으로 바꾸지 않는다.
6. 사용자가 문항화를 요청하면 content-curator 흐름으로 전환한다.

## Modes

키워드 먼저:

```text
키워드: {keyword}
떠올릴 표현을 말해보세요.
```

표현 복습:

```text
{keyword}
- {expression}
- {expression}
```

빈칸 후보:

```text
{keyword}: {expression with blank}
```

## Constraints

- `final-2` 카드는 승인된 복습 카드이지만 기본 채점형 퀴즈로 쓰지 않는다.
- 웹 검증은 하지 않는다. PDF 정리본 카드 복습으로만 다룬다.
