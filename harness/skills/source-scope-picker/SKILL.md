---
name: source-scope-picker
description: Interpret Korean natural-language quiz scope, source, type, and count requests for this repository before selecting questions.
---

# Source Scope Picker

## Purpose

사용자의 자연어 요청에서 범위, 자료, 문제 유형, 문항 수를 추출한다.

## Use This Skill For

```text
전근대사 10문제
근현대 객관식
고려만
조선 후기
파이널1 OX
파이널2 키워드
쉬운 문제
어려운 순서 문제
```

## Workflow

1. 시대/자료 키워드를 찾는다.
2. 문제 유형 키워드를 찾는다.
3. 문항 수를 찾는다.
4. `data/catalog/quiz-manifest.json`에서 가능한 approved 세트를 확인한다.
5. `파이널2`이면 채점형 퀴즈가 아니라 keyword-card-review로 라우팅한다.
6. 범위가 없으면 approved 전체, 수량이 없으면 10문제를 기본값으로 둔다.

## Keyword Mapping

```text
전근대, 고대, 삼국, 통일신라, 발해, 고려, 조선 -> premodern-5h or matching chapterIds
근현대, 현대, 광복 이후 -> modern-4h or liberation-and-modern-korea
파이널1, OX, 오엑스, 참거짓 -> final-1 fact_check
객관식, 선다형, 번호 -> multiple_choice
순서, 배열, 연표 -> chronology
키워드, 빈출표현, 암기카드, 파이널2 -> keyword-card-review
```

## Outputs

세션 내부에서 다음 값을 만든다.

```text
scopeLabel
sourceIds
chapterIds
questionTypes
targetQuestionCount
difficultyPreference
route
```

## Constraints

- 사용자에게 manifest나 JSONL 용어를 말하지 않는다.
- 해당 범위에 문항이 없으면 가능한 범위를 짧게 제안한다.
