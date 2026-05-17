# 5시간 전근대사 수업자료 구조 확인

- sourceId: `premodern-5h`
- pageCount: 36
- checkedAt: 2026-05-17
- overallType: 전근대사 압축 강의자료
- extractionResult: 텍스트 기반 강의/백지 테스트는 concept_note와 self_test_card 초안화 완료, 16쪽 손글씨는 전사 후 concept_note 추가, 혼합 기출 이미지 페이지는 항목별로 객관식/순서 배열 초안화 완료, 34쪽 도판 선택 문제만 스킵

## Page Map

| page | structure | main topics | status |
| --- | --- | --- | --- |
| 1 | 표지/도입 | 유튜브 햄지르 한국사, 전근대사 5시간 자료 | extracted |
| 2 | 강의 | 삼국시대 왕 업적 1 | extracted |
| 3 | 자가 테스트 | 고구려 왕 업적 | extracted |
| 4 | 혼합 기출 문제 | 삼국 왕 업적 기출 이미지 | question_approved |
| 5 | 강의 | 삼국시대 왕 업적 2 | extracted |
| 6 | 자가 테스트 | 신라 왕 업적 | extracted |
| 7 | 혼합 기출 문제 | 신라 왕 업적 기출 이미지 | question_approved |
| 8 | 강의 | 남북국시대 왕 업적 | extracted |
| 9 | 흐름 정리 | 신라 흐름 정리 | extracted |
| 10 | 혼합 기출 문제 | 남북국/통일신라/발해 관련 기출 이미지 | question_approved |
| 11 | 강의 | 고려 왕 업적 | extracted |
| 12 | 자가 테스트 | 고려 왕 업적 | extracted |
| 13 | 혼합 기출 문제 | 고려 왕 업적 기출 이미지 | question_approved |
| 14 | 강의 | 조선 왕 업적 | extracted |
| 15 | 자가 테스트 | 조선 왕 업적 | extracted |
| 16 | 손글씨 흐름도 | 조선 왕 업적, 사림, 붕당 정치 흐름 정리 | source_extracted_needs_review |
| 17 | 강의 | 불교 관련 내용, 스님 TOP 10, 대장경 | extracted |
| 18 | 자가 테스트 | 스님 TOP 10 백지 테스트 | extracted |
| 19 | 혼합 기출 문제 | 고려 불교사·불교 인물 기출 이미지 | question_approved |
| 20 | 강의 | 전근대 경제사, 고대·고려·조선 경제 | extracted |
| 21 | 비교표 | 고려·조선 전기·조선 후기 주요 경제 키워드 | extracted |
| 22 | 자가 테스트 | 전근대 경제 키워드 백지 테스트 | extracted |
| 23 | 혼합 기출 문제 | 전근대 경제사 기출 이미지 | question_approved |
| 24 | 강의 | 전쟁사, 후삼국 통일, 고려 대외 관계 | extracted |
| 25 | 강의 | 조선 대외 관계, 임진왜란 키워드 | extracted |
| 26 | 자가 테스트 | 후삼국 통일 과정 키워드 | extracted |
| 27 | 자가 테스트 | 임진왜란 키워드 | extracted |
| 28 | 혼합 기출 문제 | 전근대 전쟁사 기출 이미지 | question_approved |
| 29 | 강의 | 서적 1, 고대·고려·조선 역사서 | extracted |
| 30 | 표 | 역사서 비교 정리 | extracted |
| 31 | 혼합 기출 문제 | 역사서 기출 이미지 | question_approved |
| 32 | 강의 | 불상, 탑 | extracted |
| 33 | 표 | 문화유산 명칭 정리 | extracted |
| 34 | 이미지 문제 | 불상·탑 문화유산 기출 이미지 | skipped_image_based |
| 35 | 강의 | 실학, 중농학파·중상학파 | extracted |
| 36 | 자가 테스트 | 중농학파/중상학파 백지 테스트 | extracted |

## Processing Decision

이 자료는 페이지마다 강의, 백지 테스트, 기출 이미지가 섞인다. 수업자료에서 직접 확인한 내용은 `source_extracted`로 둔다. 손글씨는 읽을 수 있으면 전사하되 애매한 부분만 `source_extracted_needs_review`로 남긴다. 이미지 자체를 보고 답을 고르는 항목만 스킵하고, 말풍선·신문·화면 안 텍스트가 있는 기출 문제는 항목별로 추출해 초안 문항으로 만든다.

- `concept_note`: 강의 페이지의 핵심 개념, 인물, 제도, 사건을 정리
- `self_test_card`: 백지 테스트 페이지를 키워드 회상 카드로 정리
- `item_extracted_page`: 이미지와 텍스트가 섞인 기출 문제를 문항 단위로 분류하고 초안 문항 생성 완료
- `skipped_source_item`: 도판 자체를 보고 답을 고르는 문제만 스킵 기록으로 남김

항목별 추출 완료 페이지: 4, 7, 10, 13, 19, 23, 28, 31.

스킵 유지: 34쪽 불상·탑 문화유산 도판 선택 문제.
