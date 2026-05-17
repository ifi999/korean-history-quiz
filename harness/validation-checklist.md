# Harness Validation Checklist

## Natural Language Routing

- [ ] `퀴즈 시작`이 `start-quiz`로 이어지는가
- [ ] `전근대사 객관식 10문제`가 범위 해석 후 출제로 이어지는가
- [ ] `근현대사 수업자료 기반 빈칸 문제`가 source-recall fill_blank 출제로 이어지는가
- [ ] `정해진 문항 말고 수업자료 내용 기반 주관식`이 source-grounded-drill로 이어지는가
- [ ] `랜덤으로 골고루 10문제`가 파일 순서가 아닌 balanced random 출제로 이어지는가
- [ ] `3번` 같은 짧은 답변을 직전 문제의 답으로 처리하는가
- [ ] `모르겠어`, `패스`, `해설`을 강제 오답으로 몰지 않는가
- [ ] `오답만 다시`가 현재 세션 오답 복습으로 이어지는가
- [ ] `파이널2`가 채점형 퀴즈가 아니라 키워드 카드 복습으로 이어지는가

## Content Safety

- [ ] 기본 출제는 `data/questions/approved/`만 사용하는가
- [ ] 수업자료 기반 source-recall 문항은 `source_extracted_needs_review` concept_note를 포함하지 않는가
- [ ] source-grounded-drill은 `title`, `summary`, `keyPoints` 밖의 역사 사실을 새로 만들지 않는가
- [ ] 전체 랜덤 출제에서 큰 문항 세트 하나가 계속 연속 출제되지 않는가
- [ ] `data/cards/approved/final-2.keyword-card.jsonl`은 복습 카드로만 쓰는가
- [ ] 정답과 해설을 사용자 답변 전에는 숨기는가
- [ ] 출제한 문항의 `id`를 내부 세션 상태로 기억하는가
- [ ] 세션 종료 시 정답률과 취약 태그를 요약하는가
- [ ] 오답 저장 요청이 있을 때만 `data/review/`에 기록하는가
- [ ] 저장 형식이 `docs/review-log-format.md`와 맞는가

## Tone

- [ ] 사용자에게 JSON, JSONL, SQLite, 파일 경로를 요구하지 않는가
- [ ] 한 번에 한 문제씩 진행하는가
- [ ] 설명은 시험 암기 포인트 중심으로 짧게 제공하는가
- [ ] 사용자가 어려워하면 난이도를 낮추거나 힌트를 제공하는가

## Repo Validation

```text
scripts/validate-content
```

콘텐츠를 바꾸지 않은 하네스 문서 수정이어도 위 명령으로 기존 데이터 무결성이 유지되는지 확인한다.
