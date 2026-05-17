# Korean History Quiz

한국사능력검정시험을 준비하기 위한 Codex 대화형 퀴즈 저장소입니다. 친구가 이 repo를 pull 받은 뒤 Codex Windows 앱에서 폴더를 열고 자연어로 말하면, Codex가 문제 출제, 채점, 해설, 오답 복습을 진행하도록 구성되어 있습니다.

설치, DB 설정, JSON 편집은 기본 학습에 필요하지 않습니다.

## 바로 시작

Codex에서 이 폴더를 연 뒤 아래처럼 말하면 됩니다.

```text
한국사 퀴즈 시작해줘. 한 문제씩 내고, 내가 답하면 채점해줘.
```

짧게 말해도 됩니다.

```text
문제 내줘
```

```text
전근대사 객관식으로 10문제 내줘
```

```text
파이널2 키워드 카드 복습하자
```

친구용 사용 예시는 [START_HERE.md](START_HERE.md)에 정리되어 있고, Codex가 따라야 할 운영 규칙은 [AGENTS.md](AGENTS.md)에 있습니다.

## 현재 포함된 학습 콘텐츠

현재 기본 채점형 퀴즈에는 `data/questions/approved/`의 승인 문항만 사용합니다.

| 자료 | 유형 | 범위 | 수량 | 상태 |
| --- | --- | --- | ---: | --- |
| 파이널1 | O/X 및 오류 수정형 fact check | 전체범위 | 120문항 | 승인 완료 |
| 4시간 근현대사 수업자료 | 객관식 | 현대사 정부 시기 | 5문항 | 승인 완료 |
| 5시간 전근대사 수업자료 | 객관식/순서 배열 | 고대, 고려, 조선 | 36문항 | 승인 완료 |
| 파이널2 | 키워드 TOP85 기출표현 카드 | 전체범위 | 85카드 | 복습 카드 승인 |

합계:

- 채점형 승인 문항: 161문항
- 승인 복습 카드: 85카드
- draft 자료: 개념 노트, 백지 테스트 카드, 일부 초안 문항

`파이널2`는 문제지가 아니라 빈출 키워드 정리본에 가까운 자료입니다. 따라서 정답을 맞히는 채점형 퀴즈가 아니라 키워드 암기, 빈출표현 확인, 빠른 회독용 카드로 사용합니다.

## 자연어 트리거

Codex는 다음 표현을 학습 요청으로 처리하도록 하네스가 준비되어 있습니다.

| 말하는 방식 | 동작 |
| --- | --- |
| `퀴즈 시작`, `문제 내줘`, `한국사 해보자`, `랜덤 문제` | 기본 승인 문항에서 한 문제씩 출제 |
| `전근대`, `근현대`, `고려`, `조선`, `현대사`, `파이널1` | 범위 필터링 |
| `객관식`, `O/X`, `순서`, `키워드 카드` | 문제/카드 유형 필터링 |
| `힌트`, `모르겠어`, `패스`, `해설` | 해설 또는 스킵 처리 |
| `오답`, `틀린 것`, `다시 내줘`, `취약한 부분` | 오답 복습 흐름 |
| `파이널2`, `키워드`, `빈출표현`, `암기카드` | 키워드 카드 복습 |

하네스 상세 문서는 [harness/orchestrator-guide.md](harness/orchestrator-guide.md), [harness/trigger-map.md](harness/trigger-map.md)에 있습니다.

## 프로젝트 구조

```text
korean-history-quiz/
  AGENTS.md                  # Codex가 이 repo에서 따라야 할 최상위 운영 규칙
  START_HERE.md              # 친구용 빠른 시작 문서
  apps/
    web/                     # 향후 UI 앱 자리
  packages/
    core/                    # 향후 채점, 복습 스케줄, 세션 로직
    content/                 # 향후 데이터 로더와 스키마
    ingest/                  # 향후 PDF/OCR 추출 파이프라인
  sources/
    raw/                     # 원본 PDF 보관 위치, Git에는 기본 제외
    extracted/               # PDF 페이지별 1차 추출본
    normalized/              # 사람이 읽기 좋게 정리한 페이지별 내용
  data/
    catalog/                 # 출처, manifest, 챕터, 태그, 처리 상태
    facts/approved/          # 답안 검증에 사용한 승인 Fact
    questions/approved/      # 기본 퀴즈에 사용하는 승인 문항
    questions/draft/         # 검증 또는 정리 전 문항
    concepts/draft/          # 수업자료 기반 개념 노트
    cards/approved/          # 승인된 복습 카드
    cards/draft/             # 초안 카드
    source-analysis/draft/   # 스킵, 리뷰, 추출 판단 기록
    review/                  # 개인 오답 기록, Git에는 기본 제외
    runtime/                 # 생성 가능한 캐시, Git에는 기본 제외
  docs/                      # 설계와 작업 규칙
  harness/                   # 자연어 학습 하네스와 repo-local skill
  scripts/                   # 검증 스크립트
  tests/                     # 향후 자동 테스트 자리
```

## 데이터 원칙

- 원본 PDF는 `sources/raw/`에 두지만 Git에는 올리지 않습니다. 파일이 크거나 라이선스가 있을 수 있기 때문입니다.
- 모든 문항과 카드는 출처 PDF, 페이지, 챕터, 태그, 검증 상태를 남깁니다.
- 기본 학습에는 `approved` 상태만 사용합니다.
- `draft`, `needs_evidence`, `source_extracted_needs_review`는 기본 채점형 퀴즈에 섞지 않습니다.
- 답안지가 없는 O/X나 오류 수정형 문항은 Fact 검증을 거쳐야 합니다.
- 단순 수업자료 정리본, 백지 테스트, 키워드 표처럼 PDF 내용을 그대로 구조화한 자료는 웹 검증 없이 `source_extracted`로 관리합니다.
- 이미지 자체를 보고 답을 골라야 하는 문제는 기본 텍스트 퀴즈에서 제외하고 `skipped_image_based`로 기록합니다.
- 손글씨나 이미지 안 텍스트는 가능한 범위에서 전사하고, 불확실한 부분만 리뷰 대상으로 남깁니다.

## 콘텐츠 검증

문항, 카드, manifest를 수정한 뒤에는 아래 스크립트를 실행합니다.

```text
scripts/validate-content
```

검증 항목:

- JSON/JSONL 문법
- 문항 ID 중복
- 객관식 정답 선택지 참조
- `sourceRefs` 존재 여부
- 승인 문항의 근거 Fact 연결
- manifest 수량과 실제 파일 수량 일치

## 문서

- [프로젝트 구조](docs/project-structure.md)
- [데이터 모델](docs/data-model.md)
- [PDF 수집 및 정제 계획](docs/pdf-ingestion-plan.md)
- [분류 체계](docs/taxonomy.md)
- [무설치 Codex 학습 모드](docs/zero-setup-codex-mode.md)
- [오답 기록 형식](docs/review-log-format.md)
- [진행 플랜](docs/roadmap.md)

## 다음 작업 후보

현재 친구가 자연어로 문제를 푸는 기본 흐름은 가능한 상태입니다. 다음 개선은 선택 작업입니다.

1. `data/review/`에 실제 오답 기록 예시를 추가해 복습 세션 품질을 높입니다.
2. `파이널2` 키워드 카드를 채점형 빈칸 문제로 변환합니다.
3. draft 상태의 백지 테스트 카드를 승인 복습 카드로 승격합니다.
4. 추가 PDF를 넣을 때 `sources/extracted/`, `sources/normalized/`, `data/catalog/*manifest.json` 흐름을 반복합니다.
5. 문항 수가 더 커지면 검색용 SQLite 캐시를 생성 가능한 내부 파일로 추가하되, 원본은 계속 JSONL로 유지합니다.
