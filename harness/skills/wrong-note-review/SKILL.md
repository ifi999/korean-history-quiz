---
name: wrong-note-review
description: Review wrong answers and weak tags from the current Korean history quiz session.
---

# Wrong Note Review

## Purpose

현재 세션의 오답과 약한 태그를 복습한다.

## Use This Skill For

```text
오답 복습
틀린 것 다시
방금 틀린 문제
약한 부분 알려줘
취약 태그
오늘 틀린 문제 요약
```

## Workflow

1. 현재 대화 세션의 wrongQuestionIds와 weakTags를 확인한다.
2. 오답이 없으면 축약 요약 후 같은 범위에서 추가 문제를 제안한다.
3. 오답이 있으면 틀린 이유를 태그 단위로 묶는다.
4. 사용자가 "다시 내줘"라고 하면 같은 문항 또는 같은 태그의 approved 문항을 낸다.
5. 사용자가 "기록해줘"라고 할 때만 `data/review/` 저장 여부를 확인한다.
6. 파일로 저장할 때는 `docs/review-log-format.md`의 JSONL 형식을 따른다.

## Output Format

```text
이번 세션 오답 포인트입니다.

- {tag}: {짧은 설명}
- {tag}: {짧은 설명}

먼저 {tag} 문제를 다시 풀어볼게요.
```

## Constraints

- 개인 오답 기록은 기본적으로 Git에 올리지 않는다.
- 파일 저장은 사용자가 명시적으로 원할 때만 진행한다.
- 저장 전에는 개인 기록을 로컬 파일로 남긴다는 점을 짧게 알린다.
