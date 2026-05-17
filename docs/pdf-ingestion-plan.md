# PDF Ingestion Plan

PDF 처리의 핵심은 빠르게 텍스트만 뽑는 것이 아니라, 페이지별 출처를 유지하면서 사람이 검수할 수 있는 흐름을 만드는 것입니다. 한국사 자료는 표, 연표, 사료, 그림, 선택지가 섞일 가능성이 높으므로 자동화와 수동 검수를 분리합니다.

## 입력 원칙

1. 원본 PDF는 `sources/raw/`에 그대로 보관한다.
2. 파일명은 사람이 알아볼 수 있게 유지하되, 내부 ID는 안정적인 영문 kebab-case로 둔다.
3. 모든 페이지는 독립 작업 단위가 된다.
4. 추출 결과는 바로 문제로 만들지 않는다.
5. 페이지별 처리 상태를 기록해 큰 PDF도 중간부터 재개할 수 있게 한다.
6. 수업자료와 문제집 PDF는 전사와 구조화가 확실하면 외부 역사 검증 없이 `source_extracted`로 처리할 수 있다.
7. 웹 검증은 답안지가 없는 퀴즈처럼 Codex가 정답을 새로 판정해야 하는 경우에만 사용한다.
8. 이미지로 된 답을 선택하거나 이미지 자체를 보고 판단해야 하는 문제는 현재 기본 파이프라인에서 스킵한다.
9. 손글씨, 회전된 인쇄 텍스트, 말풍선이나 신문 이미지 안의 텍스트는 먼저 읽어 보고 전사를 시도한다.

## 초기 sourceId

| sourceId | 파일명 | 기본 분류 |
| --- | --- | --- |
| `modern-4h` | `4시간 근현대사 수업자료.pdf` | 근현대사 |
| `premodern-5h` | `5시간 전근대사 수업자료.pdf` | 전근대사 |
| `final-1` | `파이널1.pdf` | 파이널/전체범위 |
| `final-2` | `파이널2.pdf` | 파이널/전체범위 |

## 처리 단계

### 1. Register

`data/catalog/sources.json`에 원본 파일을 등록합니다.

기록할 항목:

- sourceId
- 원본 파일명
- 파일 크기
- 페이지 수
- 자료 유형
- 기본 분류
- 등록일
- 처리 상태

### 2. Extract

PDF를 페이지별 Markdown으로 추출합니다.

```text
sources/extracted/modern-4h/page-001.md
sources/extracted/modern-4h/page-002.md
```

추출 파일에는 메타데이터를 맨 위에 둡니다.

```markdown
---
sourceId: modern-4h
sourceFile: 4시간 근현대사 수업자료.pdf
pageNumber: 1
extractionMethod: text
status: extracted
---

추출된 본문
```

### 3. Normalize

사람이 한 페이지씩 읽으면서 정제합니다.

정제 기준:

- 줄바꿈과 문단 정리
- 표와 연표를 Markdown 표로 정리
- 문제 번호, 선택지, 정답 후보 분리
- 사료 원문과 해설 분리
- OCR 오류 수정
- 페이지의 핵심 주제 태그 기록

정제본은 다음 위치에 둡니다.

```text
sources/normalized/modern-4h/page-001.md
```

### 4. Concept Draft

정제된 페이지에서 개념 후보를 뽑습니다.

예시:

```markdown
## Concepts

### 무단 통치

- 시대: 일제강점기
- 핵심어: 헌병 경찰, 태형령, 조선 총독부
- 출처: modern-4h p.3
```

### 5. Question Draft

개념과 파이널 문제에서 문항 초안을 만듭니다.

초안은 `data/questions/draft/{sourceId}.{questionType}.jsonl`에 둡니다. 한 줄에 한 문제를 넣으면 대형 자료에서도 부분 수정과 diff 확인이 쉽습니다.

문제형 자료가 아니라 키워드와 빈출표현 표라면 `data/cards/draft/{sourceId}.keyword-card.jsonl`에 둡니다.

예시:

```text
data/questions/draft/final-1.fact-check.jsonl
data/questions/approved/final-1.fact-check.jsonl
data/cards/approved/final-2.keyword-card.jsonl
```

짧은 PDF는 페이지별로 JSONL을 나누지 않습니다. 페이지 번호는 각 문항의 `sourceRefs.pageNumber`에 기록합니다. 수백 쪽 규모의 대형 PDF에서만 챕터나 파트 단위 분할을 검토합니다.

수업자료에서 확실히 추출된 `source_extracted` concept_note는 빈칸/단답 회상형 문항으로 변환할 수 있습니다. 이때 새 내용을 추론하지 않고 summary/keyPoints 표현만 사용합니다.

```text
scripts/generate-source-recall-questions
```

이 스크립트는 `source_extracted_needs_review` concept_note를 자동 승인 문항에서 제외합니다.

### 6. Validate

검증 항목:

- 문제 ID 중복 없음
- 출처와 페이지 번호 존재
- 정답 존재
- 객관식 선택지 ID와 정답 ID 일치
- 챕터 태그 존재
- 해설 존재
- 승인 문항에는 `status: approved`

검증 유형:

```text
fact verification:
  파이널1처럼 답지가 없는 O/X, 오류 수정형 자료에서 필요하다.
  Codex가 역사 사실의 참거짓을 새로 판정하므로 검증 완료 Fact나 신뢰 자료를 연결한다.
  웹 검증은 이 경우처럼 답안지가 없고 내부 자료만으로 정답을 확정하기 어려울 때만 사용한다.

source extraction review:
  수업자료/문제집 PDF에서 보이는 텍스트를 그대로 구조화할 때 적용한다.
  전사와 구조화가 확실하면 `source_extracted`로 기록하고 별도 외부 검증과 웹 검증을 생략한다.
  손글씨나 흐릿한 이미지처럼 일부 판독이 애매하면 `source_extracted_needs_review`와 `uncertainSegments`를 남긴다.
```

### 7. Promote

검수 완료 문항만 `data/questions/approved/`로 이동하거나 복사합니다.

앱은 기본적으로 `approved`만 읽습니다. `draft`는 관리자 또는 검수 화면에서만 사용합니다.

키워드 카드는 PDF 정리본 전사가 확실하면 웹 검증 없이 `data/cards/approved/`로 승격할 수 있습니다. 이후 카드에서 빈칸, 단답, 연결형, O/X 같은 실제 채점형 문항을 별도로 생성합니다.

## 페이지 검수 체크리스트

각 페이지를 읽을 때 다음을 확인합니다.

- 이 페이지의 시대 범위는 무엇인가?
- 핵심 사건, 제도, 인물, 단체는 무엇인가?
- 연도 또는 순서 암기가 필요한가?
- 사료 문제로 만들 수 있는 문장이 있는가?
- 서로 헷갈리기 쉬운 비교 대상이 있는가?
- 기존 챕터에 들어가는가, 새 챕터가 필요한가?
- 파이널 문제라면 실제 약점 태그는 무엇인가?

## 이미지형 PDF 대응

나중에 큰 PDF가 이미지 스캔본이면 OCR이 필요합니다. 이 경우에도 출력은 동일하게 페이지별 Markdown으로 맞춥니다.

우선순위:

1. PDF 내장 텍스트 추출
2. OCR 추출
3. 페이지 이미지와 사람이 작성한 수동 정리 병행

OCR 결과는 반드시 `extractionMethod: ocr`로 표시합니다.

단, 사진·그림·지도·문화유산 이미지 자체가 정답 판단의 핵심인 문제는 OCR 대상이 아니라 `skipped_image_based`로 기록합니다. 같은 개념을 텍스트 강의자료에서 확보할 수 있으면 그쪽을 우선 사용합니다.

손글씨 정리, 회전된 인쇄 텍스트, 신문/말풍선 이미지 안의 텍스트는 이미지 문제로 보지 않습니다. 읽을 수 있으면 전사하고, 확신이 낮은 단어만 별도로 표시합니다.

## 대형 PDF 처리 전략

큰 PDF를 한 번에 처리하지 않습니다.

- 20쪽 단위 배치 처리
- 페이지별 상태 기록
- 추출 실패 페이지 목록 별도 관리
- 정제 완료 페이지부터 문항화
- 자동 생성 문항은 반드시 검수 후 승인

이 방식이면 300쪽 PDF가 들어와도 `1-20쪽`, `21-40쪽`처럼 나눠 진행할 수 있습니다.
