# Zero Setup Codex Mode

이 프로젝트의 실제 사용자는 한국사 학습자입니다. 사용자가 SQLite, JSONL, 빌드 도구, 스크립트를 알 필요가 없어야 합니다. 저장소 내부는 확장 가능하게 설계하되, 사용 경험은 Codex와 대화하는 방식으로 단순화합니다.

## 목표 사용자 경험

친구가 해야 할 일:

```text
1. Git에서 프로젝트를 받는다.
2. Codex Windows 앱에서 폴더를 연다.
3. "한국사 퀴즈 시작해줘"라고 말한다.
```

친구가 하지 않아도 되는 일:

```text
- DB 실행
- SQLite 이해
- JSON 파일 편집
- Node/Python 설치
- 빌드 명령 실행
- PDF 추출
```

## 내부 구조와 사용자 경험 분리

내부적으로는 대량 문항을 위해 파일을 분리합니다.

```text
data/questions/approved/
  final-1.fact-check.jsonl
  premodern-5h.fact-check.jsonl
  modern-4h.multiple-choice.jsonl

data/cards/draft/
  final-2.keyword-card.jsonl
```

하지만 사용자는 이 파일을 직접 다루지 않습니다. Codex가 필요한 문항을 찾아서 한 문제씩 대화로 냅니다.

## 대량 문항 확장 전략

수백, 수천 문항이 생겨도 한 파일에 몰아넣지 않습니다.

분할 기준:

```text
1차 기준: 자료별 final-1, final-2, premodern-5h, modern-4h
2차 기준: 유형별 fact_check, multiple_choice, chronology, source_analysis, keyword_card
예외 기준: 대형 PDF는 챕터/파트별 추가 분할
상태 기준: draft, source_extracted, source_extracted_needs_review, needs_evidence, approved
```

이렇게 나누면 Codex가 `rg` 같은 빠른 검색으로 필요한 문항만 찾을 수 있고, Git diff도 작게 유지됩니다.

## SQLite의 위치

SQLite는 사용자가 알아야 하는 개념이 아닙니다.

나중에 앱이나 대량 검색이 필요하면 내부 캐시로 쓸 수 있습니다.

```text
data/runtime/content-cache.sqlite
```

이 파일은 생성 가능한 캐시이며 Git에 올리지 않습니다. 없어도 Codex 대화형 퀴즈는 JSONL 원본을 읽어서 진행할 수 있어야 합니다.

## 출제 규칙

Codex는 다음 순서로 행동합니다.

```text
1. 사용자의 요청 범위를 파악한다.
2. approved 문항에서만 후보를 고른다.
3. 한 문제를 낸다.
4. 답변 전에는 정답과 해설을 숨긴다.
5. 답변 후 채점한다.
6. 틀린 경우 오답 포인트를 요약한다.
7. 사용자가 원하면 오답 기록을 data/review/에 남긴다.
```

`keyword_card`는 예외적으로 채점형 문항이 아닙니다. 사용자가 명시적으로 파이널2 카드 복습을 요청하면 암기 목록이나 빈칸 후보로 보여줄 수 있습니다. 파이널2 카드는 PDF 정리본 전사 자료이므로 복습 카드로는 승인되어 있지만, `approved` 문항처럼 정답 채점에 쓰려면 별도 문항 변환이 필요합니다.

## 자연어 하네스

친구의 자연어 요청은 repo 안의 하네스 문서를 기준으로 해석합니다.

```text
harness/orchestrator-guide.md
harness/trigger-map.md
harness/skills/*/SKILL.md
```

예시:

```text
문제 내줘
  -> approved 문항에서 기본 10문제

전근대 객관식 10문제
  -> premodern-5h approved 객관식/순서 배열

3번
  -> 직전 문항 채점

오답 다시
  -> 현재 세션 오답 복습

파이널2 키워드
  -> 채점형 퀴즈가 아니라 keyword_card 복습
```

## approved의 의미

`approved`는 사용자가 역사 지식으로 검수했다는 뜻이 아닙니다.

```text
approved = 근거가 연결되어 출제 가능한 상태
```

검증 흐름:

```text
ai_draft
  AI가 문항과 정답 초안을 생성

needs_evidence
  근거가 부족하거나 충돌함

source_extracted
  제공된 수업자료/문제집 PDF에서 신뢰도 높게 전사함

source_extracted_needs_review
  손글씨, 흐림, 회전 이미지 등으로 일부 전사 확인이 필요함

evidence_verified
  신뢰 가능한 자료로 정답 확인

approved
  대화형 퀴즈에 출제 가능
```

## 친구용 안전장치

친구가 잘못된 미검증 문제를 풀지 않도록 기본값을 보수적으로 둡니다.

- 기본 출제 대상은 `approved`만
- 근거 없는 정답은 단정 금지
- 모호한 문제는 출제 제외
- 해설에는 핵심 암기 포인트 포함
- 오답 기록은 개인 데이터이므로 Git에 올리지 않음
