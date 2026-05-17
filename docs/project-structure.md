# Project Structure

이 프로젝트는 PDF 자료 수집, 콘텐츠 정제, 퀴즈 앱, 복습 기록을 분리해서 운영합니다. 처음에는 작은 로컬 프로젝트로 시작하되, 나중에 웹 앱, 모바일 앱, 서버 저장소로 확장할 수 있게 경계를 나눕니다.

## 최상위 구조

```text
korean-history-quiz/
  apps/
  packages/
  sources/
  data/
  docs/
  scripts/
  tests/
```

## apps

사용자가 직접 사용하는 화면을 둡니다.

```text
apps/
  web/
    quiz/              # 문제 풀이 화면
    review/            # 복습 큐
    wrong-notes/       # 오답노트
    stats/             # 챕터별 정답률, 반복 오답
```

초기 구현은 로컬 웹 앱을 권장합니다. 한국사 자료는 텍스트와 표가 많고, 복습 기록을 눈으로 훑는 경험이 중요하므로 CLI보다 브라우저 UI가 적합합니다.

## packages

앱과 데이터 처리에서 공유할 핵심 로직을 분리합니다.

```text
packages/
  core/
    quiz-session        # 문제 출제, 정답 판정, 세션 상태
    review-scheduler    # 오답 재출제, 간격 반복
    scoring             # 정답률, 약점 계산

  content/
    schema              # Source, Page, Chapter, Question, KeywordCard 모델
    validation          # 문항 필수값, 중복, 태그 검증
    loaders             # JSON/JSONL 로딩

  ingest/
    pdf                 # PDF 페이지 추출
    ocr                 # 이미지형 PDF 대응
    normalize           # 추출 텍스트 정리
    question-draft      # 문항 초안 생성
```

## sources

원본과 중간 산출물을 보관합니다. 여기는 학습 자료의 출처를 보존하는 영역입니다.

```text
sources/
  raw/
    4시간 근현대사 수업자료.pdf
    5시간 전근대사 수업자료.pdf
    파이널1.pdf
    파이널2.pdf

  extracted/
    modern-4h/
      page-001.md
      page-002.md

  normalized/
    modern-4h/
      page-001.md
```

`extracted`는 기계 추출에 가깝고, `normalized`는 사람이 읽으면서 고친 정제본입니다. 두 단계를 분리해야 PDF 추출 품질이 낮아도 원인을 추적할 수 있습니다.

## data

앱이 직접 읽을 수 있는 구조화 데이터를 둡니다.

```text
data/
  catalog/
    sources.json
    chapters.json
    taxonomy.json

  questions/
    draft/
      modern-4h.questions.jsonl
    approved/
      modern-4h.questions.jsonl

  concepts/
    draft/
      modern-4h.concept-note.jsonl

  cards/
    draft/
      final-2.keyword-card.jsonl
    approved/
      final-2.keyword-card.jsonl

  source-analysis/
    draft/
      premodern-5h.skipped-image-based.jsonl
      premodern-5h.item-review.jsonl

  review/
    attempts.jsonl
    wrong-notes.jsonl
    review-state.json
```

초기에는 JSON/JSONL로 시작합니다. 문항 수가 많아지고 검색, 통계, 동기화가 필요해지면 SQLite로 이전할 수 있습니다. 이때도 데이터 모델을 유지하면 앱 코드는 크게 흔들리지 않습니다.

`questions`는 채점 가능한 문항이고, `cards`는 키워드와 빈출표현을 저장하는 학습 재료입니다. 카드가 바로 정답 문항은 아니므로, 문항 변환과 채점 검수 후 출제합니다.

`concepts`는 강의자료에서 뽑은 출제 재료입니다. `source-analysis`는 사료형 후보, 항목별 추출 기록, 이미지 기반 스킵 기록을 둡니다.

## docs

프로젝트 설계와 작업 규칙을 기록합니다.

```text
docs/
  project-structure.md
  data-model.md
  pdf-ingestion-plan.md
  taxonomy.md
  roadmap.md
```

## scripts

반복 작업을 실행하는 얇은 명령을 둡니다.

```text
scripts/
  ingest-pdf
  validate-content          # JSON/JSONL, ID 중복, 선택지 참조, manifest count 검증
  promote-questions
  export-review-report
```

스크립트는 앱 로직을 직접 품지 않고, `packages/ingest`, `packages/content`, `packages/core`를 호출하는 진입점 역할만 합니다.

## tests

데이터가 깨지지 않았는지 확인합니다.

```text
tests/
  content-schema
  question-validation
  review-scheduler
```

초기 테스트의 우선순위는 UI보다 데이터 검증입니다. PDF에서 만든 문제가 늘어날수록 중복 ID, 빠진 정답, 잘못된 태그, 출처 누락이 더 큰 문제가 됩니다.
