# Korean History Quiz Harness Plan

## Purpose

이 하네스는 개발자가 아닌 학습자가 Codex에 자연어로 말해서 한국사능력검정시험 문제를 풀 수 있게 하는 repo 내 운영 지침이다. 사용자는 JSONL, SQLite, 스크립트, 파일 경로를 알 필요가 없다.

## Pattern

선택한 패턴은 Codex-native study harness이다.

- `AGENTS.md`가 최상위 라우터 역할을 한다.
- `harness/orchestrator-guide.md`가 자연어 트리거와 진행 흐름을 정의한다.
- `harness/skills/*/SKILL.md`가 학습 기능별 행동 규칙을 제공한다.
- `harness/agent-specs/*.md`가 역할별 말투와 책임을 고정한다.

별도 앱, 서버, 플러그인 설치를 요구하지 않는다. 친구는 Git repo를 받은 뒤 Codex Windows 앱에서 폴더를 열고 자연어로 요청하면 된다.

## Scope

포함:

- 승인 문항 기반 퀴즈 시작
- 범위/유형 자연어 해석
- 한 문제씩 출제
- 답변 채점
- 오답 복습
- 파이널2 키워드 카드 복습
- 모호한 요청을 안전한 기본값으로 처리

제외:

- 새 PDF 추출과 문항 제작 자동화
- 웹 검증 자동 실행
- 친구에게 파일 편집, DB 실행, 스크립트 실행 요구
- 미승인 draft 문항의 기본 출제

## Default Study Flow

```text
사용자: 한국사 퀴즈 시작해줘
Codex:
  1. source-scope-picker로 범위와 유형을 해석한다.
  2. start-quiz로 approved 문항만 고른다.
  3. 한 문제를 낸다.
  4. 사용자 답변을 기다린다.
  5. grade-answer로 채점한다.
  6. 다음 문제를 이어갈지 묻는다.
  7. 세션 종료 시 review-coach 방식으로 취약 태그를 요약한다.
```

## Safety Defaults

- 범위가 없으면 approved 전체에서 최대 10문제를 낸다.
- 한 번에 한 문제만 낸다.
- 답하기 전에는 정답, 해설, evidenceRefs를 보여주지 않는다.
- `draft`, `needs_evidence`, `source_extraction_pending`은 기본 출제에서 제외한다.
- 사용자가 "모르겠어", "패스", "해설"이라고 하면 틀림으로 몰지 않고 해설 모드로 전환한다.
- 사용자가 저장을 요청하지 않으면 오답 기록은 대화 안에서만 유지한다.
- 오답 저장을 요청하면 `docs/review-log-format.md` 형식을 따른다.

## Current Content Targets

기본 출제 가능:

- `data/questions/approved/final-1.fact-check.jsonl`: 120문항
- `data/questions/approved/modern-4h.multiple-choice.jsonl`: 5문항
- `data/questions/approved/premodern-5h.multiple-choice.jsonl`: 36문항

복습 전용:

- `data/cards/approved/final-2.keyword-card.jsonl`: 키워드 카드 85개, 채점형 출제에는 사용하지 않음

## Maintenance

콘텐츠나 하네스를 수정한 뒤에는 `harness/validation-checklist.md`와 `scripts/validate-content`를 확인한다. Git 초기화, commit, push는 사용자가 요청하기 전에는 하지 않는다.
