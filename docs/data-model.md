# Data Model

데이터 모델의 목표는 문제를 많이 만드는 것이 아니라, 나중에 틀린 이유를 정확히 추적할 수 있게 만드는 것입니다. 모든 문항은 출처, 페이지, 챕터, 태그, 정답, 해설, 검수 상태를 가져야 합니다.

사용자는 이 구조를 알 필요가 없습니다. Codex가 `approved` 문항을 읽어 대화형으로 출제하고 채점합니다. JSONL, SQLite, 캐시 파일은 내부 구현 세부사항입니다.

## Storage Strategy

수백, 수천 문항을 한 파일에 몰아넣지 않습니다.

권장 저장 방식:

```text
문항 원본: data/questions/{status}/{source-id}.{type}.jsonl
키워드 카드: data/cards/{status}/{source-id}.{type}.jsonl
개인 오답: data/review/*.jsonl
런타임 캐시: data/runtime/*
```

예시:

```text
data/questions/approved/final-1.fact-check.jsonl
data/questions/approved/premodern-5h.fact-check.jsonl
data/questions/approved/modern-4h.multiple-choice.jsonl
data/cards/approved/final-2.keyword-card.jsonl
```

분할 기준은 페이지가 아니라 자료와 유형입니다. `파이널1.pdf`처럼 8쪽짜리 자료는 `final-1.fact-check.jsonl` 하나에 모읍니다. 페이지 추적은 각 문항의 `sourceRefs.pageNumber`와 `sourceRefs.itemNumber`로 해결합니다.

대형 PDF는 예외적으로 나눌 수 있습니다.

```text
data/questions/approved/premodern-5h-prehistoric.fact-check.jsonl
data/questions/approved/premodern-5h-goryeo.fact-check.jsonl
```

즉, 일반 기준은 `자료 단위 통합`이고, 수백 쪽 규모가 되어 한 파일이 너무 커질 때만 `챕터/파트 단위 분할`을 사용합니다.

SQLite는 사용자가 직접 다루는 저장소가 아닙니다. 나중에 검색과 통계가 커지면 `data/runtime/content-cache.sqlite` 같은 생성 가능한 내부 캐시로만 사용합니다. 캐시가 없어도 Codex 대화형 퀴즈는 JSONL 원본으로 진행할 수 있어야 합니다.

## Verification Status

`approved`는 사용자가 역사 지식으로 승인했다는 뜻이 아닙니다. 정해진 근거 검증 절차를 통과해 출제 가능하다는 뜻입니다.

```text
ai_draft          AI가 만든 초안
needs_evidence    근거가 부족하거나 충돌함
source_extracted  제공된 PDF의 내용을 신뢰도 높게 전사/구조화함
source_extracted_needs_review
                  제공된 PDF에서 전사했지만 손글씨/흐림 등 일부 확인이 필요함
source_extraction_pending
                  페이지 전체 스킵은 아니지만 문항 단위 분리가 아직 필요함
evidence_verified 근거 자료로 정답 확인
approved          대화형 퀴즈에 출제 가능
skipped_image_based
                  이미지 자체 판단이 핵심이라 텍스트 퀴즈 데이터화에서 제외함
```

`source_extracted`는 외부 역사 검증을 생략한다는 뜻이지, 곧바로 채점형 승인 문항이라는 뜻은 아닙니다. 수업자료에서 직접 추출한 개념 카드, 백지 테스트, 지문 전사는 이 상태를 사용할 수 있습니다. `파이널1.pdf`처럼 답지가 없는 O/X 문항은 Codex가 참거짓을 새로 판정해야 하므로 `evidence_verified` 또는 `approved`까지 검증 근거를 연결해야 합니다.

수업자료에서 확실히 추출된 `source_extracted` concept_note는 빈칸, 단답, 연결형 같은 회상형 문항으로 변환할 수 있습니다. 이 경우 정답은 summary/keyPoints 안의 표현만 사용하고, 문항에는 `sourceGroundingStatus: "source_extracted"`와 source_extracted Fact를 연결합니다. 손글씨나 판독 불확실성이 있는 `source_extracted_needs_review` concept_note는 자동 승인 문항으로 만들지 않습니다.

객관식 기출 페이지에서 발문과 선택지를 정확히 전사했더라도 정답표가 없으면 문항의 `verificationStatus`는 `needs_evidence`로 둡니다. 이 경우 원문 추출 상태는 `questionExtractionStatus: source_extracted`, 정답 검증 상태는 `answerVerificationStatus: needs_evidence`로 분리합니다.

웹 검증은 답안지가 없는 채점형 퀴즈나 내부 자료만으로 정답을 확정할 수 없는 경우에만 사용합니다. 정리본, 강의 표, 백지 테스트, 키워드 카드처럼 제공 PDF 내용을 그대로 구조화하는 데이터에는 웹 검증을 붙이지 않습니다.

정답 검증이 끝나 approved로 승격한 문항은 최소 1개의 `evidenceRefs: [{"type":"fact","factId":"..."}]`를 가져야 합니다. 검증기는 approved 문항의 상태와 연결된 Fact ID 존재 여부를 함께 확인합니다.

## Source

PDF 또는 외부 자료 하나를 나타냅니다.

```json
{
  "id": "modern-4h",
  "title": "4시간 근현대사 수업자료",
  "fileName": "4시간 근현대사 수업자료.pdf",
  "sourceType": "lecture_pdf",
  "defaultScope": "modern",
  "pageCount": 19,
  "status": "registered",
  "notes": "근현대사 개념 강의자료"
}
```

### sourceType

- `lecture_pdf`: 강의자료
- `final_pdf`: 파이널 문제
- `past_exam`: 기출문제
- `manual_note`: 직접 작성한 보충자료

### status

- `registered`: 원본 등록
- `extracted`: 페이지별 추출 완료
- `normalized`: 사람이 읽고 정제 완료
- `question_drafted`: 문항 초안 생성
- `card_drafted`: 키워드 카드 초안 생성
- `approved`: 앱 사용 가능

## Page

PDF의 한 페이지를 나타냅니다.

```json
{
  "id": "modern-4h-p001",
  "sourceId": "modern-4h",
  "pageNumber": 1,
  "rawPath": "sources/extracted/modern-4h/page-001.md",
  "normalizedPath": "sources/normalized/modern-4h/page-001.md",
  "status": "normalized",
  "detectedTopics": ["일제강점기", "무단 통치"],
  "reviewMemo": "표와 연표가 있어 수동 검수 필요"
}
```

페이지 단위 상태를 둬야 큰 PDF를 처리할 때 중간에 멈췄다가 이어갈 수 있습니다.

## Chapter

한국사능력검정시험 학습 단위입니다.

```json
{
  "id": "modern-japanese-colonial",
  "label": "일제강점기",
  "parentId": "modern",
  "order": 720,
  "aliases": ["일제 강점기", "일제시대"],
  "examWeight": "high"
}
```

챕터는 계층형입니다. 예를 들어 `근현대사 > 일제강점기 > 1910년대 통치`처럼 확장할 수 있습니다.

## Concept

문항으로 만들기 전의 지식 단위입니다.

```json
{
  "id": "concept-000001",
  "sourceRefs": [
    {
      "sourceId": "modern-4h",
      "pageNumber": 3
    }
  ],
  "chapterIds": ["modern-japanese-colonial"],
  "tags": ["정치", "통치방식", "1910년대"],
  "title": "무단 통치",
  "body": "1910년대 일제는 헌병 경찰 제도를 중심으로 강압적인 통치를 실시했다.",
  "importance": 5
}
```

개념을 따로 두면 같은 개념에서 여러 문제 유형을 만들 수 있습니다.

## Skipped Source Page

이미지 자체를 보고 답을 골라야 하는 문제는 현재 텍스트 대화형 퀴즈에 적합하지 않으므로 스킵 기록만 남깁니다. 손글씨, 회전된 인쇄 텍스트, 말풍선이나 신문 이미지 안의 텍스트는 먼저 전사를 시도하고, 읽기 어려운 부분만 `uncertainSegments`로 남깁니다.

```json
{
  "id": "premodern-5h-skip-034",
  "status": "skipped",
  "verificationStatus": "skipped_image_based",
  "type": "skipped_source_page",
  "title": "불상·탑 문화유산 이미지 기출 문제",
  "skipReason": "불상과 탑 사진을 보고 답을 고르는 이미지 판단 문제이므로 현재 대화형 텍스트 퀴즈 데이터화 대상에서 제외한다.",
  "sourceRefs": [
    {
      "sourceId": "premodern-5h",
      "pageNumber": 34
    }
  ],
  "chapterIds": ["ancient"],
  "tags": ["이미지문제", "스킵", "문화유산"]
}
```

## Question

앱에서 출제되는 최소 단위입니다.

```json
{
  "id": "q-000001",
  "status": "draft",
  "sourceRefs": [
    {
      "sourceId": "final-1",
      "pageNumber": 2
    }
  ],
  "chapterIds": ["modern-japanese-colonial"],
  "period": "근현대사",
  "topic": "일제강점기",
  "type": "multiple_choice",
  "prompt": "다음 설명에 해당하는 일제의 통치 방식으로 옳은 것은?",
  "choices": [
    {
      "id": "A",
      "text": "문화 통치"
    },
    {
      "id": "B",
      "text": "무단 통치"
    }
  ],
  "answer": {
    "choiceId": "B"
  },
  "explanation": "헌병 경찰 제도와 태형령은 1910년대 무단 통치의 대표적 특징이다.",
  "tags": ["정치", "통치방식", "사료"],
  "difficulty": 2,
  "createdFrom": "manual_review"
}
```

## Fact Check Question

`파이널1.pdf`처럼 O/X 또는 잘못된 표현을 고치는 자료는 `fact_check`로 다룹니다.

```json
{
  "id": "final-1-001",
  "status": "approved",
  "verificationStatus": "approved",
  "type": "fact_check",
  "statement": "고구려의 제천행사는 무천이다.",
  "answer": {
    "verdict": "incorrect",
    "wrongPart": ["고구려", "무천"],
    "correction": "고구려의 제천행사는 동맹이고, 동예의 제천행사는 무천이다."
  },
  "explanation": "초기 여러 나라의 제천 행사는 자주 비교 출제된다.",
  "sourceRefs": [
    {
      "sourceId": "final-1",
      "pageNumber": 1,
      "itemNumber": 8
    }
  ],
  "evidenceRefs": [
    {
      "type": "fact",
      "factId": "fact-early-states-festival"
    }
  ],
  "chapterIds": ["ancient"],
  "tags": ["초기국가", "제천행사", "비교", "빈출표현"]
}
```

이 유형은 정답만 저장하면 안 됩니다. 틀린 부분, 올바른 문장, 해설, 근거를 함께 저장해야 합니다.

### type

- `multiple_choice`: 객관식
- `true_false`: OX
- `short_answer`: 단답형
- `fill_blank`: 빈칸
- `source_analysis`: 사료 해석
- `chronology`: 순서 배열
- `matching`: 연결형

## Keyword Card

`파이널2.pdf`처럼 문제라기보다 “키워드 - 빈출표현” 표로 구성된 자료는 `keyword_card`로 다룹니다.

키워드 카드는 바로 정답 문항으로 보지 않습니다. 먼저 표현을 보존하고, PDF 정리본 전사가 확실하면 `source_extracted` 복습 카드로 승인할 수 있습니다. 이후 빈칸, 단답, O/X, 객관식처럼 채점형 문항으로 바꿀 때는 별도 문항 변환을 거칩니다.

```json
{
  "id": "final-2-067",
  "status": "approved",
  "verificationStatus": "source_extracted",
  "type": "keyword_card",
  "keyword": "무단 통치",
  "expressions": [
    "강압적인 통치를 목적으로 헌병 경찰 제도를 시행하였다.",
    "한국인에게만 조선 태형령을 시행하였다."
  ],
  "sourceRefs": [
    {
      "sourceId": "final-2",
      "pageNumber": 7,
      "itemNumber": 67
    }
  ],
  "evidenceRefs": [],
  "chapterIds": ["japanese-colonial"],
  "tags": ["일제강점기", "1910년대", "무단통치", "빈출표현"],
  "createdFrom": "manual_pdf_review"
}
```

카드 출제 규칙:

- `approved` 카드는 암기 카드나 빈칸 문제의 재료로 사용할 수 있다.
- `draft` 또는 `needs_evidence` 카드는 사용자가 명시적으로 원할 때만 검토용으로 사용한다.
- 카드의 표현은 암기 복습에는 사용할 수 있지만 채점형 정답 문항으로 바로 단정하지 않는다. 정답 문항으로 쓰려면 별도 문항 변환과 검증 상태를 부여한다.

## Attempt

사용자가 문제를 푼 기록입니다.

```json
{
  "id": "attempt-000001",
  "questionId": "q-000001",
  "answeredAt": "2026-05-17T15:40:00+09:00",
  "selectedAnswer": {
    "choiceId": "A"
  },
  "isCorrect": false,
  "elapsedSeconds": 18,
  "mode": "final_review"
}
```

문항 데이터와 풀이 기록은 절대 섞지 않습니다. 그래야 문제 내용을 수정해도 과거 풀이 기록을 보존할 수 있습니다.

## Wrong Note

오답노트는 단순 오답 목록이 아니라 약점 분석 단위입니다.

```json
{
  "questionId": "q-000001",
  "firstWrongAt": "2026-05-17T15:40:00+09:00",
  "lastWrongAt": "2026-05-17T15:40:00+09:00",
  "wrongCount": 1,
  "lastSelectedAnswer": {
    "choiceId": "A"
  },
  "reason": "confused_with_cultural_rule",
  "memo": "문화 통치와 무단 통치 특징 구분 필요"
}
```

### reason

- `concept_unknown`: 개념 모름
- `confused_with_other`: 유사 개념과 혼동
- `chronology_error`: 시대 순서 오류
- `source_misread`: 사료 해석 오류
- `keyword_missed`: 핵심 단서 누락
- `careless`: 실수

## Review State

반복 복습 상태입니다.

```json
{
  "questionId": "q-000001",
  "stage": 1,
  "nextReviewAt": "2026-05-18",
  "lastReviewedAt": "2026-05-17",
  "correctStreak": 0,
  "wrongStreak": 1
}
```

초기 복습 간격은 `1일 -> 3일 -> 7일 -> 14일 -> 30일` 정도로 시작하고, 반복 오답은 다시 1단계로 내립니다.
