---
name: source-grounded-drill
description: Generate oral short-answer drills directly from source_extracted lecture concept notes without inventing historical facts or relying only on prewritten questions.
---

# Source Grounded Drill

## Purpose

수업자료 내용을 기반으로 즉석 주관식 문제를 낸다. 이 흐름은 미리 작성된 문항을 그대로 반복하는 것이 아니라, `source_extracted` concept_note의 `title`, `summary`, `keyPoints`, `tags`, `sourceRefs`만 사용해 구술형/단답형/백지 테스트형 질문을 만든다.

## Use This Skill For

```text
수업자료 기반으로 물어봐
내용 기반으로 내줘
정해진 문항 말고 개념에서 물어봐
주관식으로 물어봐
백지 테스트처럼 해줘
개념 설명하게 해줘
꼬리질문 해줘
암기 확인해줘
근현대사 수업자료로 질문해줘
전근대 개념으로 물어봐
```

## Allowed Sources

기본 사용 가능:

```text
data/concepts/draft/modern-4h.concept-note.jsonl
data/concepts/draft/premodern-5h.concept-note.jsonl
```

사용 조건:

```text
verificationStatus == "source_extracted"
```

사용 금지:

```text
verificationStatus == "source_extracted_needs_review"
verificationStatus == "needs_evidence"
uncertainSegments가 있는 항목
이미지 자체 판단이 필요한 skipped_image_based 항목
```

## Question Generation Rules

1. `title`, `summary`, `keyPoints`에 있는 내용만 사용한다.
2. 새로운 역사 사실, 연도, 인과관계, 오답 선택지를 만들어내지 않는다.
3. 사용자의 범위 요청이 있으면 `sourceId`, `chapterIds`, `tags`로 concept_note를 좁힌다.
4. 한 번에 한 질문만 낸다.
5. 정답 예시는 사용자가 답하기 전에는 숨긴다.
6. 같은 세션에서 같은 concept_note만 반복하지 않는다.
7. 사용자가 "더 어렵게"라고 하면 keyPoints 2-3개를 함께 요구한다.
8. 사용자가 "쉽게"라고 하면 title 또는 대표 키워드 하나를 묻는다.

## Prompt Patterns

정의 회상:

```text
{summary에서 title을 가린 문장}
이 설명에 해당하는 핵심 개념을 답하세요.
```

핵심 포인트 열거:

```text
{title}과 관련해 수업자료에 나온 핵심 포인트를 2가지 말해보세요.
```

비교 설명:

```text
{title}에서 헷갈리기 쉬운 항목을 수업자료 기준으로 구분해보세요.
```

순서 회상:

```text
{title}의 전개 흐름을 수업자료에 나온 순서대로 말해보세요.
```

## Grading Rules

1. `answer.text`가 없으므로 `title`, `summary`, `keyPoints`, `tags`를 기준으로 채점한다.
2. 핵심 명사와 관계가 맞으면 정답 또는 부분 정답으로 인정한다.
3. 표현이 달라도 의미가 같으면 맞게 본다.
4. 수업자료에 없는 내용을 덧붙인 경우에는 "추가 내용은 자료 밖이므로 여기서는 판단하지 않는다"고 말한다.
5. 틀렸을 때는 정답 전체를 길게 강의하지 말고, 빠진 keyPoint를 1-3개만 짚는다.
6. 사용자가 출처를 요청하면 `sourceRefs.sourceId`와 `pageNumber`를 보여준다.

## Output Format

```text
수업자료 기반 주관식으로 갈게요.
문제 1입니다.

{question}

짧게 핵심어만 답해도 됩니다.
```

채점:

```text
맞았습니다.
핵심은 {keyPoint}입니다.
```

부분 정답:

```text
거의 맞았습니다.
맞힌 부분: {matched}
빠진 부분: {missing}
```

오답:

```text
정답은 {title}입니다.
핵심은 {keyPoint}입니다.
```

## Constraints

- 사전 생성된 `data/questions/approved/*.jsonl` 문항만 반복하지 않는다.
- 즉석 질문은 concept_note에 근거한 학습 진행용이다. 저장 가능한 승인 문항으로 남기려면 별도 변환 스크립트나 콘텐츠 관리 흐름을 사용한다.
- 답지가 없는 O/X처럼 역사 사실을 새로 판정하는 문제를 즉석 생성하지 않는다.
