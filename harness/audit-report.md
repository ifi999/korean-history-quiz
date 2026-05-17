# Harness Audit Report

## Mode

New build.

## Findings

- 기존 repo에는 `AGENTS.md`, `START_HERE.md`, 데이터 모델, 검증 스크립트가 이미 있다.
- 친구가 자연어로 학습하는 흐름은 문서화되어 있었지만, 키워드 트리거와 기능별 스킬 경계가 분리되어 있지 않았다.
- approved 문항과 draft 카드의 경계가 중요하다. 특히 `파이널2`는 키워드 카드이므로 일반 채점형 퀴즈와 분리해야 한다.

## Decisions

- repo-contained harness를 선택했다. 친구 PC에 별도 Codex 스킬 설치를 요구하지 않는다.
- `AGENTS.md`에 하네스 라우팅 섹션을 추가해 Codex가 `harness/` 문서를 참고하도록 한다.
- 학습 기능은 5개 스킬로 나눈다: 범위 해석, 퀴즈 시작, 채점, 오답 복습, 키워드 카드 복습.
- 역할은 4개로 나눈다: study-coach, quiz-master, review-coach, content-curator.

## Risks

- Codex가 repo-local `harness/skills/*`를 자동 설치형 스킬처럼 실행하지는 않는다. 따라서 `AGENTS.md` 라우팅이 중요하다.
- 친구가 `파이널2 문제 내줘`라고 말할 때 채점형 문제로 오해할 수 있다. 하네스에서 카드 복습으로 라우팅한다.
- 세션 상태는 기본적으로 대화 안에서만 유지된다. 장기 오답 저장은 별도 구현 전까지 명시 요청이 있을 때만 다룬다.

## Validation

- 하네스 파일이 실제 repo 경로를 가리키는지 확인한다.
- `scripts/validate-content`로 기존 콘텐츠 데이터 무결성을 확인한다.
- 친구용 문서에는 파일 구조보다 말 걸기 예시를 우선한다.
