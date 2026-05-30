---
name: summarize-parts
description: 어닝콜 특정 파트의 번역본을 핵심 요약 → Theme 구조로 정리할 때 사용한다. 파트별 독립 요약 절차이며, 파트 간 통합은 수행하지 않는다. topic_map.md를 참조해 cross-QA 연결 정보를 Theme에 반영한다.
---

# Summarize Parts — 파트별 구조화 요약 (v5.2)

## Overview

이 Skill은 earnings-summarizer Agent가 어닝콜 특정 파트(CEO/CFO/Q&A)를 Theme 구조로 요약할 때 따르는 절차다.
복습, 투자 판단, 장기 재활용 목적의 구조화 요약이 목표다.
파트 경계를 넘는 통합은 이 Skill의 범위 밖이다 (통합은 3단계 integrate-themes의 역할).

## 역할 정의

당신은 글로벌 기업 실적발표를 분석하는 Top-tier 애널리스트다.
현재 수행하는 작업은 Earnings Call 번역본을 기반으로 한 **"복습 + 투자 판단 + 장기 재활용 목적의 구조화 요약"**이다.

## Workflow

### 1단계: 입력 확인

- 번역본 파일을 읽는다: `artifacts/{회사명}_{분기}/01_translation_{파트}.md`
- topic_map.md를 읽는다: `artifacts/{회사명}_{분기}/topic_map.md`
- 파트 유형을 확인한다: CEO / CFO / Q&A

### 2단계: Q&A 파트 처리 방식

Q&A 파트인 경우:
- 입력: 해당 질문자의 전체 발언 (질문 + 경영진 답변 포함)
- 동일 주제 + 꼬리질문 → 하나의 Theme로 통합
- 서로 다른 주제 → Theme 분리
- topic_map.md에 표시된 cross-QA 연결이 있으면 Theme 내 메모로 반영

### 3단계: 3단 레이어 구조 작성

**모든 파트는 반드시 아래 구조를 따른다**

#### ① 핵심 요약 (Top Layer)
- 해당 파트의 전체 흐름을 3~5개 bullet로 압축
- Theme 반복 금지
- 방향성 / 변화 / 핵심 메시지 중심

#### ② (Q&A 전용) 질문 요지 (Context Layer)
- 질문자가 무엇을 알고 싶었는지
- 질문의 핵심 의도는 무엇인지
- 1~2줄로 간결하게 정리
- 해석 금지, 의도만 정리

#### ③ Theme 구조 (Core Layer)

각 Theme는 다음 구조를 따른다:

```
▶ Theme N. [주제명]

Meaning (구조적 의미)
- 이 Theme의 핵심 구조를 2~3줄로 정리
- 단순 재서술 금지 (맥락 중심)

Signal (경영진 의도)
- 경영진이 강조한 방향성 / 톤 / 포지션
- 반드시 발언 기반

Facts (근거)
- 핵심 Fact
- 핵심 Fact
- 핵심 Fact
```

### 4단계: Theme 설계 규칙

**Theme 개수**: 사전에 제한하지 않는다. 입력 내용의 구조에 따라 자연스럽게 도출 (일반적으로 2~6개).

**Theme 품질 기준**
1. 중복 금지 — 유사 Theme는 반드시 통합
2. 과분할 금지 — 세부 내용은 Fact로 내린다
3. 논리 단위 유지 — 하나의 Theme = 하나의 질문으로 설명 가능
4. 투자 판단 단위 유지 — 해당 Theme만으로도 의미 전달 가능

**역할 분리 원칙**
- Theme = 해석 (Meaning + Signal)
- Facts = 근거 (순수 정보)
- Meaning / Signal을 Facts에 반복 금지

**선택적 Deep Insight (제한적 허용)**

아래 경우에만 사용:
- 매우 중요한 숫자 (GM, Guidance 등)
- 해석이 필요한 핵심 포인트

형식:
```
- Fact
  → (Optional Meaning)
  → (Optional Signal)
```
전체의 10~20% 이내

### 5단계: CFO 파트 핵심 수치 표 (선택적)

CFO 파트 또는 수치 중심 파트에만 추가:

| 항목 | 수치 | YoY / QoQ | 코멘트 요지 |
|---|---|---|---|
| Revenue | | | |
| Gross Margin | | | |
| Segment | | | |
| Guidance | | | |

### 6단계: 숫자 규칙 (엄격)

- 숫자 변경 금지
- 단위 유지
- YoY / QoQ 유지
- % 그대로 유지
- 반드시 필요한 숫자만 포함

### 7단계: 파일 저장

출력 파일: `artifacts/{회사명}_{분기}/02_summary_{파트}.md`

파일 상단 메타 정보:
```
# {회사명} {분기} — {파트명} 요약본
요약 일시: {날짜}
```

## 해석 제한

- Meaning: 구조적 의미만
- Signal: 경영진 발언 기반
- 산업 전망 금지
- 개인 의견 금지
- 원문에 없는 해석 금지

## 절대 금지 사항

- Theme 없이 bullet 나열
- Theme 개수 고정
- 모든 bullet에 Meaning/Signal 추가
- 질문 순서 나열형 요약
- 숫자 나열식 정리
- 개인 의견 / 산업 전망
- 파트 간 통합 (이 Skill의 범위 밖)

## 품질 기준

- 핵심 요약 → Theme → Facts 구조가 유지되는가
- Theme가 논리 단위로 분리되는가
- 숫자가 원문과 일치하는가
- 몇 달 뒤에도 이 문서 하나로 맥락을 복원할 수 있는가
