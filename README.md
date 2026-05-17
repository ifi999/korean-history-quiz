# Korean History Quiz

한국사능력검정시험 합격을 목표로, PDF 강의자료와 파이널 문제를 페이지 단위로 정리하고 퀴즈, 오답노트, 반복 복습까지 연결하는 학습 프로그램입니다.

이 프로젝트는 처음부터 여러 개의 큰 PDF를 추가할 수 있는 구조를 전제로 합니다. 원본 자료는 보존하고, 페이지별 추출본과 정제 데이터, 최종 문항 데이터를 분리해 추적 가능성을 유지합니다.

## 목표

- PDF 자료를 한 페이지씩 읽고 정리한다.
- 자료 출처, 페이지, 챕터, 시대, 주제, 문제 유형을 잃지 않는다.
- 전근대사, 근현대사, 파이널/전체범위 자료를 함께 관리한다.
- 객관식, OX, 빈칸, 사료 해석, 순서 배열 등 다양한 문제 유형을 지원한다.
- 오답, 반복 오답, 취약 챕터, 복습 주기를 추적한다.
- 추후 PDF와 챕터가 늘어나도 데이터 구조를 크게 바꾸지 않는다.
- 친구가 Windows Codex 앱에서 폴더를 열고 바로 대화형으로 문제를 풀 수 있게 한다.

## 친구용 빠른 시작

이 저장소를 받은 사용자는 DB, JSON, 빌드 도구를 몰라도 됩니다. Codex에서 이 폴더를 열고 아래처럼 말하면 됩니다.

```text
한국사 퀴즈 시작해줘. 한 문제씩 내고, 내가 답하면 채점해줘.
```

자세한 사용 예시는 [START_HERE.md](START_HERE.md)를 봅니다. Codex가 따라야 할 학습 진행 규칙은 [AGENTS.md](AGENTS.md)에 둡니다.

자연어 기반 학습 흐름은 [harness/orchestrator-guide.md](harness/orchestrator-guide.md)와 [harness/trigger-map.md](harness/trigger-map.md)에 정리되어 있습니다. 친구는 하네스 파일을 직접 볼 필요 없이 Codex에 말만 하면 됩니다.

## 초기 대상 자료

원본 PDF는 `sources/raw/`에 보관합니다.

| 파일 | 임시 분류 | 역할 |
| --- | --- | --- |
| `4시간 근현대사 수업자료.pdf` | 근현대사 | 개념, 연표, 사료, 문제 후보 추출 |
| `5시간 전근대사 수업자료.pdf` | 전근대사 | 개념, 연표, 사료, 문제 후보 추출 |
| `파이널1.pdf` | 파이널/전체범위 | 실전형 문제와 약점 태그 추출 |
| `파이널2.pdf` | 파이널/전체범위 | 키워드 TOP85 기출표현 카드 추출 |

파이널 자료는 파일 단위로는 `전체범위`에 넣되, 각 문항에는 실제 시대와 주제 태그를 별도로 붙입니다.

## 권장 구조

```text
korean-history-quiz/
  apps/
    web/                     # 퀴즈 UI, 오답노트, 통계 화면
  packages/
    core/                    # 채점, 복습 스케줄, 퀴즈 세션 로직
    content/                 # 데이터 스키마, 검증기, 콘텐츠 로더
    ingest/                  # PDF 추출, OCR, 페이지 정제 파이프라인
  sources/
    raw/                     # 원본 PDF
    extracted/               # 페이지별 1차 추출본
    normalized/              # 사람이 검수한 정제본
  data/
    catalog/                 # 출처, 챕터, 태그, 분류 체계
    questions/
      draft/                 # 검수 전 문항
      approved/              # 앱에서 사용할 승인 문항
    concepts/
      draft/                 # 문항 생성 전 개념 초안
    cards/
      draft/                 # 검증 전 키워드 카드
      approved/              # 검증 완료 키워드 카드
    source-analysis/
      draft/                 # 사료형 후보, 항목별 추출 기록, 이미지 기반 스킵 기록
    review/                  # 풀이 기록, 오답 기록, 복습 상태
  docs/                      # 설계 문서
  scripts/                   # 반복 작업 실행 스크립트
  tests/                     # 데이터 검증 및 핵심 로직 테스트
  harness/                   # Codex 자연어 학습 하네스와 트리거
```

## 핵심 설계 원칙

1. 원본 보존: PDF는 수정하지 않고 `sources/raw/`에 그대로 둡니다.
2. 페이지 추적: 모든 개념과 문제는 원본 파일명과 페이지 번호를 가집니다.
3. 초안과 승인 분리: 자료에서 확실히 추출한 `source_extracted`와 채점형 출제 가능한 `approved`를 분리합니다.
4. 분류는 다중 태그: 하나의 문제는 `조선후기`, `경제`, `대동법`, `사료`처럼 여러 태그를 가질 수 있습니다.
5. 복습 데이터 분리: 문제 데이터와 사용자의 풀이 기록은 섞지 않습니다.

## 진행 문서

- [프로젝트 구조](docs/project-structure.md)
- [데이터 모델](docs/data-model.md)
- [PDF 수집 및 정제 계획](docs/pdf-ingestion-plan.md)
- [분류 체계](docs/taxonomy.md)
- [무설치 Codex 학습 모드](docs/zero-setup-codex-mode.md)
- [오답 기록 형식](docs/review-log-format.md)
- [진행 플랜](docs/roadmap.md)

## 콘텐츠 검증

콘텐츠를 추가한 뒤에는 Codex가 다음 스크립트로 JSON/JSONL 문법, ID 중복, 선택지 정답 참조, manifest 카운트를 확인합니다.

```text
scripts/validate-content
```

## 다음 단계

1. `sources/raw/`에 초기 4개 PDF를 복사합니다.
2. 각 PDF의 페이지 목록과 처리 상태를 `data/catalog/sources.json`으로 등록합니다.
3. 페이지별 추출본을 `sources/extracted/{source-id}/page-001.md` 형태로 생성합니다.
4. 페이지별로 개념, 연표, 사료, 문제 후보를 정제합니다.
5. `data/concepts/draft/`, `data/questions/draft/`, `data/cards/draft/`에 초안을 만들고, 자료 유형에 따라 `source_extracted`, `source_extracted_needs_review`, `needs_evidence`, `approved` 상태로 관리합니다.

이미지나 사진 자체를 보고 답을 고르는 문제는 현재 기본 파이프라인에서 제외하고 `skipped_image_based`로 기록합니다. 손글씨, 회전된 인쇄 텍스트, 화면·말풍선·신문 안의 텍스트는 먼저 전사를 시도하고, 확실하지 않은 부분만 리뷰 대상으로 남깁니다.

현재 기본 출제 가능한 승인 문항은 파이널1 `fact_check` 120개, 4시간 근현대사 객관식 5개, 5시간 전근대사 객관식/순서 배열 36개입니다. 4시간/5시간 수업자료의 혼합 기출 페이지는 항목별로 분리한 뒤 정답 번호를 근거 Fact로 검증해 `approved`로 승격했습니다. 도판 자체를 보고 고르는 문제만 스킵 상태로 남깁니다.
