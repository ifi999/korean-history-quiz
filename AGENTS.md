# Codex Study Agent Instructions

이 저장소는 한국사능력검정시험 합격을 위한 대화형 퀴즈 프로젝트입니다. 사용자는 한국사 지식, JSON/JSONL, SQLite, 빌드 도구를 몰라도 됩니다. Codex는 저장소의 자료를 읽고 문제를 내며, 채점과 오답 복습까지 대화로 진행합니다.

이 파일은 이 프로젝트에서 Codex가 따라야 할 최상위 운영 규칙입니다. 데이터 모델의 상세 형식은 `docs/data-model.md`, PDF 처리 흐름은 `docs/pdf-ingestion-plan.md`, 친구용 사용법은 `START_HERE.md`를 참고합니다.

## Core Mission

1. 사용자가 "퀴즈 시작", "문제 내줘", "오답 복습"처럼 말하면 바로 학습 세션을 시작한다.
2. 사용자가 파일 구조, DB, 스크립트, JSONL을 직접 다루게 하지 않는다.
3. 친구가 Windows Codex 앱에서 폴더를 열고 프롬프트만 주고받아 학습할 수 있게 유지한다.
4. 정답은 사용자가 답하기 전에는 노출하지 않는다.
5. 채점 후에는 정답, 틀린 부분, 올바른 설명, 핵심 암기 포인트를 짧게 알려준다.
6. 역사 사실을 새로 판정해야 하는 상황에서는 근거 없이 단정하지 않는다.
7. 데이터가 늘어나도 출처, 페이지, 유형, 상태, 태그를 잃지 않는다.

## Friend Study Harness

친구의 일반 학습 요청은 repo-local 하네스를 따른다. 하네스는 설치형 스킬이 아니라 이 저장소 안의 운영 지침이다.

우선 참고 파일:

```text
harness/orchestrator-guide.md
harness/trigger-map.md
harness/skills/start-quiz/SKILL.md
harness/skills/source-scope-picker/SKILL.md
harness/skills/grade-answer/SKILL.md
harness/skills/wrong-note-review/SKILL.md
harness/skills/keyword-card-review/SKILL.md
```

자연어 트리거:

```text
퀴즈 시작, 문제 내줘, 공부 시작, 한국사 해보자, 랜덤 문제
  -> source-scope-picker 필요 시 적용 후 start-quiz

전근대, 근현대, 고려, 조선, 현대사, 파이널1, 객관식, OX, 순서, 10문제
  -> source-scope-picker

1번, 2, O, X, 맞아, 틀려
  -> 직전 문항에 대한 grade-answer

모르겠어, 패스, 해설, 답 알려줘
  -> grade-answer의 skipped/해설 흐름

오답, 틀린 것, 다시 내줘, 복습, 취약, 약한 부분
  -> wrong-note-review

파이널2, 키워드, 빈출표현, 암기카드, 외울 것
  -> keyword-card-review
```

친구 학습 모드에서는 내부 파일명, JSONL, SQLite, manifest를 먼저 설명하지 않는다. 사용자의 요청이 `PDF 추가`, `문항 생성`, `검증`, `승격`, `하네스 수정`처럼 콘텐츠 관리나 개발 작업이면 학습 모드에서 빠져나와 일반 작업 모드로 처리한다.

## Default Quiz Behavior

기본 출제 대상은 `data/questions/approved/`의 문항입니다.

```text
1. `data/catalog/quiz-manifest.json`에서 approvedQuestionSets를 확인한다.
2. 사용자가 범위를 지정하면 sourceId, chapterIds, tags, questionType으로 좁힌다.
3. 범위가 없으면 approved 문항 전체에서 최대 10문제를 balanced random으로 고른다.
4. 한 번에 한 문제씩 낸다.
5. 사용자가 답하면 즉시 채점한다.
6. 틀린 경우 오답 원인을 태그 단위로 짧게 정리한다.
7. 세션 마지막에는 정답률, 취약 태그, 다음 복습 대상을 요약한다.
```

`draft`, `ai_draft`, `needs_evidence`, `source_extraction_pending` 상태는 기본 채점형 퀴즈에 쓰지 않습니다. 사용자가 명시적으로 "초안도 보여줘", "파이널2 카드 복습"처럼 요청한 경우에만 검토용으로 사용합니다.

## Question Selection Rules

기본 출제는 파일 순서나 itemNumber 순서가 아닙니다. 사용자가 명시적으로 "순서대로 풀자"라고 하지 않는 한, Codex는 다음 방식으로 골고루 섞어 출제합니다.

```text
1. 요청 범위에 맞는 approved 문항 후보를 만든다.
2. 후보를 sourceId, chapterIds, questionType, difficulty, tags 기준으로 묶는다.
3. 전체 범위 요청이면 큰 세트가 작은 세트를 압도하지 않도록 source set을 먼저 균등하게 섞는다.
4. 특정 범위 요청이면 해당 범위 안에서 pageNumber나 itemNumber가 연속되지 않게 섞는다.
5. 같은 세션에서 이미 낸 currentQuestionId는 다시 내지 않는다.
6. 오답 복습 요청일 때만 wrongQuestionIds 또는 weakTags를 우선한다.
7. 후보가 부족할 때만 남은 문항을 재사용하거나 가까운 범위를 제안한다.
```

예시:

```text
"랜덤 문제" / "아무거나 내줘"
  -> 파이널1 120문항만 앞에서부터 내는 방식이 아니라, 파이널1/전근대/근현대 승인 세트를 섞어 낸다.

"전근대사 객관식 10문제"
  -> premodern-5h 안에서 페이지 번호와 태그가 몰리지 않도록 섞어 낸다.

"순서 문제"
  -> chronology 유형 요청이다. 파일 순서대로 출제하라는 뜻으로 보지 않는다.

"처음부터 순서대로"
  -> 이 경우에만 itemNumber 순차 출제를 허용한다.
```

## Data Status Rules

상태의 의미를 혼동하지 않습니다.

```text
ai_draft
  AI가 만든 초안

source_extracted
  제공된 PDF의 내용을 신뢰도 높게 전사/구조화함

source_extracted_needs_review
  손글씨, 흐림, 회전 이미지 등으로 일부 전사 확인이 필요함

source_extraction_pending
  페이지 전체 스킵은 아니지만 문항 단위 분리가 아직 필요함

needs_evidence
  정답 판단을 위해 추가 근거가 필요함

evidence_verified
  신뢰 자료 또는 승인 Fact로 정답 확인

approved
  대화형 채점 퀴즈에 출제 가능

skipped_image_based
  이미지 자체 판단이 핵심이라 텍스트 퀴즈 데이터화에서 제외함
```

중요한 구분:

- `source_extracted`는 "자료에서 정확히 옮겼다"는 뜻이지, 곧바로 채점형 문항이라는 뜻은 아닙니다.
- `approved`는 사용자가 역사 지식으로 승인했다는 뜻이 아니라, 이 프로젝트의 검증 절차를 통과해 출제 가능하다는 뜻입니다.
- 답이 PDF에 명시되어 있거나 수업자료 표/정리 내용을 그대로 카드화한 경우에는 `source_extracted`를 사용할 수 있습니다.
- 답이 PDF에 없고 Codex가 참거짓이나 정답 번호를 새로 판정해야 하는 경우에는 `needs_evidence` 또는 별도 검증 절차가 필요합니다.
- 기출 문제 페이지에서 발문과 선택지를 잘 전사했더라도, 정답표가 없으면 문항 전체의 `verificationStatus`는 `needs_evidence`로 둡니다. 이때 `questionExtractionStatus: source_extracted`, `answerVerificationStatus: needs_evidence`처럼 원문 추출과 정답 검증을 분리합니다.

## Verification Policy

검증은 두 종류로 나눕니다.

```text
fact verification
  파이널1처럼 답지가 없는 O/X, 오류 수정형, 참거짓 판정형 자료에 필요하다.
  Codex가 역사 사실을 새로 판정하므로 승인 Fact나 신뢰 자료가 필요하다.

source extraction review
  4시간/5시간 수업자료처럼 제공된 PDF 내용을 그대로 구조화할 때 적용한다.
  전사와 구조화가 확실하면 외부 역사 검증 없이 `source_extracted`로 처리한다.
  손글씨처럼 일부 판독이 애매하면 `source_extracted_needs_review`와 `uncertainSegments`를 남긴다.
```

웹 검증은 기본값이 아닙니다. 웹 검증은 답안지가 없는 채점형 퀴즈, 정답표 없이 Codex가 객관식 정답 번호를 새로 판정하는 경우, 내부 자료끼리 충돌하거나 제공 자료만으로 판정이 불가능한 경우에만 사용합니다. 단순 정리본, 수업자료 표, 백지 테스트, 키워드 카드처럼 제공 PDF의 내용을 옮기고 구조화하는 작업은 웹 검증 없이 `source_extracted`로 처리합니다.

정답 검증이 필요한 대표 케이스:

```text
- 답지가 없는 O/X 문항
- 객관식 기출 이미지를 보고 Codex가 정답 번호를 추론한 경우
- 해설이나 정답표 없이 여러 선택지 중 하나를 골라야 하는 경우
- 시대 판별, 순서 배열처럼 한 단계라도 역사 판단이 들어가는 경우
```

정답 검증이 비교적 필요 없는 대표 케이스:

```text
- 수업자료의 개념 표를 그대로 concept_note로 옮긴 경우
- 백지 테스트의 항목을 회상 카드로 옮긴 경우
- PDF 안에 정답, 해설, 정답표가 함께 명시된 문제를 그대로 전사한 경우
```

근거 우선순위:

```text
1. 이 저장소의 `data/facts/approved/`
2. 제공된 PDF에서 정제한 내용
3. 답안지 없는 퀴즈 검증에 한해 한국사능력검정시험 공식 또는 공신력 있는 한국사 자료
4. 일반 웹 자료는 답안지 없는 퀴즈 검증의 보조 참고로만 사용
```

사용자가 검증된 Fact를 알지 못한다는 점을 전제로 합니다. 따라서 `approved` 승격을 사용자 감에 맡기지 말고, Codex가 근거 파일과 출처를 남겨야 합니다.

## PDF Ingestion Rules

원본 PDF는 `sources/raw/`에 그대로 보관하고, 페이지별 작업 흔적을 남깁니다.

```text
sources/extracted/{sourceId}/page-XXX.md
sources/normalized/{sourceId}/page-XXX.md
data/concepts/draft/{sourceId}.concept-note.jsonl
data/questions/{status}/{sourceId}.{type}.jsonl
data/cards/{status}/{sourceId}.{type}.jsonl
data/source-analysis/draft/*.jsonl
```

처리 순서:

```text
1. PDF 구조를 먼저 확인한다.
2. 페이지별 extracted Markdown을 만든다.
3. normalized Markdown에서 사람이 읽을 수 있게 정리한다.
4. 페이지 유형을 concept, self_test, answered_question, unanswered_question, image_judgment, mixed_page로 분류한다.
5. 개념, 카드, 문항, 스킵/검토 대기 기록을 분리한다.
6. 답지 없는 문제는 원문 추출 상태와 정답 검증 상태를 분리한다.
7. catalog 파일의 카운트와 상태를 함께 갱신한다.
8. JSON/JSONL 파싱과 ID 중복을 확인한다.
```

짧은 PDF는 페이지별 JSONL로 쪼개지 않습니다. 기본 분할 기준은 `자료별 + 유형별`입니다. 수백 쪽 이상의 대형 PDF에서 파일이 너무 커질 때만 챕터/파트 단위 분할을 검토합니다.

SQLite는 사용자가 알아야 하는 저장소가 아닙니다. 나중에 검색/통계가 커지면 `data/runtime/content-cache.sqlite` 같은 생성 가능한 내부 캐시로만 사용하고, 없어도 JSONL 원본으로 대화형 퀴즈가 가능해야 합니다.

문항을 `draft`에서 `approved`로 승격할 때는 같은 ID를 두 상태에 동시에 남기지 않습니다. draft를 보존해야 한다면 `final-1-draft-001`처럼 ID를 분리하거나, 별도 archive 정책을 둡니다.

## Image, Handwriting, And Mixed Pages

이 프로젝트에서 한 번 잘못 처리했던 지점은 "이미지가 들어간 페이지를 통째로 스킵"한 것입니다. 앞으로는 페이지 단위가 아니라 항목 단위로 판단합니다.

스킵하는 경우:

```text
- 문화유산 사진을 보고 고르는 문제
- 지도 모양 자체를 보고 판단하는 문제
- 그림/도판 자체 식별이 정답의 핵심인 문제
```

스킵하지 않고 먼저 시도하는 경우:

```text
- 손글씨 정리
- 회전된 인쇄 텍스트
- 말풍선, 신문, 화면 캡처 안의 텍스트
- 이미지가 섞였지만 발문과 선택지가 텍스트로 판독 가능한 기출 문제
```

처리 방식:

```text
1. 도판 자체 판단이 핵심인 항목만 `skipped_image_based`로 기록한다.
2. 텍스트를 읽을 수 있으면 전사한다.
3. 일부 단어가 불확실하면 `uncertainSegments`를 남기고 `source_extracted_needs_review`로 둔다.
4. 한 페이지 안에 텍스트 문제와 이미지 판별 문제가 섞이면 `source_extraction_pending`으로 두고 항목별로 다시 분리한다.
5. 답지가 없는 객관식 문항은 발문/선택지 전사를 `source_extracted`로 보되, 정답 번호는 `needs_evidence`로 둔다.
6. 같은 개념이 텍스트 강의나 백지 테스트에 있으면 그 텍스트 자료를 우선 사용한다.
```

## Fact Check Questions

`fact_check`는 단순 O/X가 아니라 틀린 부분을 고치는 사실 검증형 문항입니다.

출제 형식:

```text
다음 문장이 맞으면 O, 틀리면 X로 답하세요.
가능하면 틀린 부분도 함께 고쳐보세요.
```

채점 형식:

```text
- O/X 판정
- 틀린 부분
- 올바른 문장
- 핵심 암기 포인트
- 근거 출처
```

답지가 없는 O/X 자료는 Codex가 정답을 새로 판정하는 것이므로, `approved`로 쓰려면 반드시 근거 Fact 또는 신뢰 자료 연결이 필요합니다.

## Cards And Concepts

`concept_note`는 강의자료에서 뽑은 출제 재료입니다. 곧바로 채점형 문제가 아닙니다.

`keyword_card`는 키워드와 빈출표현을 저장합니다. 예를 들어 `파이널2.pdf`는 O/X 문항이 아니라 `keyword_card` 85개로 정리되어 있습니다.

`self_test_card`는 백지 테스트나 암기표를 회상 카드로 바꾼 것입니다. 수업자료에서 그대로 추출한 경우 `source_extracted`를 사용할 수 있지만, 채점형 승인 문제로 쓰려면 문항 변환이 필요합니다.

카드 사용 원칙:

```text
1. 기본 채점 퀴즈에는 approved 질문만 사용한다.
2. draft 카드는 사용자가 명시적으로 요청한 복습 모드에서만 보여준다.
3. 카드 표현을 O/X나 객관식으로 바꿀 때 정답 판단이 새로 필요하면 검증을 추가한다.
4. approved 카드가 생기면 빈칸, 단답, 연결형, O/X 문항의 재료로 사용할 수 있다.
```

## Current Source Map

현재 등록된 초기 자료:

```text
modern-4h
  4시간 근현대사 수업자료
  concept_note 33개
  16쪽 지도 활동지는 skipped_image_based
  19쪽 혼합 기출 페이지는 multiple_choice approved 5개로 승격 완료
  발문/선택지는 source_extracted, 정답 번호는 근거 Fact로 evidence_verified

premodern-5h
  5시간 전근대사 수업자료
  concept_note 24개
  self_test_card 11개
  16쪽 손글씨는 source_extracted_needs_review 1개 포함
  34쪽 도판 선택 문제는 skipped_image_based
  4,7,10,13,19,23,28,31쪽 혼합 기출 페이지는 multiple_choice/chronology approved 36개로 승격 완료
  발문/선택지는 source_extracted, 정답 번호는 근거 Fact로 evidence_verified
  premodern-5h-mc-031처럼 ㄱ/ㄴ/ㄷ/ㄹ 보기 조합형 문항은 원문 보기 문장을 발문에 함께 보존해야 한다.

final-1
  답지 없는 O/X, 오류 수정형 fact_check
  1-120번 approved 완료
  approved Facts와 연결됨

final-2
  키워드 TOP85 기출표현 카드
  keyword_card 85개 approved card
  PDF 정리본 전사 자료이므로 웹 검증 없이 source_extracted로 승인
  채점형 문제로 쓰기 전에는 별도 문항 변환 필요
```

## Review And Wrong Notes

사용자가 원하면 오답은 `data/review/` 아래에 로컬 기록으로 남깁니다. 사용자가 파일 저장을 원하지 않으면 대화 안에서만 요약합니다.

오답 기록 형식은 `docs/review-log-format.md`를 따릅니다.

기록할 내용:

```text
- 문항 ID
- 답변 시각
- 사용자의 답
- 정답 여부
- 틀린 이유
- 복습 필요 태그
```

개인 오답 기록은 Git에 올리지 않는 것을 기본으로 합니다.

## Validation Checklist

콘텐츠를 추가하거나 카탈로그를 수정한 뒤에는 최소한 다음을 확인합니다.

```text
jq empty data/catalog/*.json
jq -c empty data/concepts/draft/*.jsonl data/cards/draft/*.jsonl data/questions/*/*.jsonl data/facts/*/*.jsonl data/source-analysis/draft/*.jsonl
jq -r '.id' data/concepts/draft/*.jsonl data/cards/draft/*.jsonl data/questions/*/*.jsonl data/facts/*/*.jsonl data/source-analysis/draft/*.jsonl | sort | uniq -d
```

스크립트가 필요하면 다음 명령을 우선 사용합니다.

```text
scripts/validate-content
```

추가로 확인할 것:

```text
- manifest의 count와 실제 JSONL line count가 맞는가
- sourceRefs.sourceId와 pageNumber가 있는가
- 상태값이 `data/catalog/taxonomy.json`에 있는가
- draft와 approved가 같은 문항 ID를 공유하지 않는가
- 이미지 혼합 페이지를 통째로 스킵하지 않았는가
- `source_extracted`를 `approved`와 혼동하지 않았는가
- 답지 없는 객관식/순서 배열 문항을 `source_extracted`만으로 검증 완료 처리하지 않았는가
- 객관식/순서 배열 draft에는 필요한 경우 `answerVerificationStatus`가 분리되어 있는가
```

## Git And Collaboration

사용자가 명시적으로 요청하기 전에는 `git init`, commit, push를 하지 않습니다. 이 프로젝트는 어느 정도 구조와 데이터가 안정된 뒤 사용자가 Git 초기화를 진행할 예정입니다.

작업 중 파일을 수정했다면 최종 답변에서 핵심 변경 파일과 검증 결과만 짧게 알려줍니다.
