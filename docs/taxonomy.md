# Taxonomy

분류 체계는 문제 출제보다 오답 분석을 위해 설계합니다. 한국사능력검정시험에서는 시대, 주제, 사료 단서, 헷갈리는 비교 대상이 모두 중요하므로 단일 챕터만으로는 부족합니다.

## 기본 축

하나의 문항은 여러 축의 태그를 가질 수 있습니다.

```text
시대 축: 전근대사, 근현대사
시기 축: 선사, 고대, 고려, 조선전기, 조선후기, 개항기, 일제강점기, 현대
주제 축: 정치, 경제, 사회, 문화, 외교, 군사, 인물, 단체, 제도, 사상
문항 축: 사료, 연표, 순서, 지도, 비교, 키워드
자료 축: 강의자료, 파이널, 기출, 수동노트
```

## 대분류

```text
premodern       전근대사
modern          근현대사
full-range      전체범위
final           파이널
```

`final`은 실제 역사 범위가 아니라 자료 유형입니다. 따라서 파이널 문제도 반드시 시대 태그를 따로 가져야 합니다.

## 시대/시기

```text
prehistoric             선사
ancient                 고대
three-kingdoms          삼국
unified-silla-balhae    남북국
goryeo                  고려
joseon-early            조선전기
joseon-late             조선후기
opening-port            개항기
japanese-colonial       일제강점기
liberation              광복 전후
modern-korea            현대
```

## 주제

```text
politics        정치
economy         경제
society         사회
culture         문화
diplomacy       외교
military        군사
institution     제도
person          인물
organization    단체
thought         사상
religion        종교
artifact        문화재
chronology      연표
source          사료
map             지도
```

## 문제 유형 태그

```text
keyword         핵심어로 푸는 문제
comparison      비교 문제
chronology      순서 배열
source-analysis 사료 해석
concept-check   개념 확인
trap-choice     함정 선택지
final-review    파이널 복습
weakness-drill  취약점 반복
```

## 난이도

```text
1   기본 개념
2   대표 기출 수준
3   비교/사료 단서 필요
4   세부 연도, 지엽 구분 필요
5   고난도 또는 반복 오답 전용
```

## 챕터 확장 규칙

처음부터 모든 세부 챕터를 만들지 않습니다. 다음 조건 중 하나를 만족하면 새 챕터를 만듭니다.

- 같은 주제로 10문항 이상 생긴다.
- 오답률이 높아 별도 복습 단위가 필요하다.
- 다른 시대와 자주 혼동된다.
- 한국사능력검정시험에서 독립 단원처럼 출제된다.

예시:

```text
modern
  japanese-colonial
    colonial-rule-1910s
    march-first-movement
    cultural-rule
    national-movement-1920s
    wartime-mobilization
```

## 파이널 자료 처리

파이널 PDF의 문항은 다음처럼 분류합니다.

```json
{
  "sourceRefs": [
    {
      "sourceId": "final-1",
      "pageNumber": 4
    }
  ],
  "chapterIds": ["joseon-late"],
  "tags": ["final", "economy", "comparison", "daedong-law"]
}
```

즉, 출처는 파이널이지만 약점 분석은 실제 챕터 기준으로 합니다.
