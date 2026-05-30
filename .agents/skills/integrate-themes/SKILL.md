---
name: integrate-themes
description: 파트별 요약본 전체를 읽어 Earnings Call 전체를 관통하는 Global Theme 통합본을 생성할 때 사용한다. CEO/CFO/Q&A 경계를 제거하고 내용 기준으로 재구성하는 3단계 절차다.
---

# Integrate Themes — Global Theme 통합 요약 (Global Theme v1)

## Overview

이 Skill은 earnings-integrator Agent가 모든 파트 요약본을 통합해 Earnings Call 전체를 하나의 논리 구조로 재구성할 때 따르는 절차다.
"누가 말했는가"가 아닌 "무슨 주제가 반복되었는가"를 중심으로 재구성한다.
몇 달 뒤에도 이 문서 하나로 전체 맥락을 복원할 수 있어야 한다.

## 역할 정의

당신은 글로벌 소프트웨어·플랫폼·테크 기업을 분석하는 Top-tier 애널리스트다.
현재 수행하는 작업은 Earnings Call 전체 내용을 기반으로 한 **"통합 구조 요약 (Global Theme Summary)"**이다.

## Workflow

### 1단계: 입력 수집

```
Glob: artifacts/{회사명}_{분기}/02_summary_*.md
Read: artifacts/{회사명}_{분기}/topic_map.md
```

### 2단계: 파트 구분 완전 제거

- CEO / CFO / Q&A 구분 금지
- "누가 말했는지" 제거
- 오직 내용 기준으로 재구성

### 3단계: Global Theme 추출

**Global Theme 정의**: Earnings Call 전체에서 반복되거나 중요하게 다뤄진 주제

**규칙**
- Theme 개수 제한 없음 (자연 도출, 일반적으로 4~7개)
- 중복 Theme 금지
- 유사 내용 반드시 통합

topic_map.md의 "여러 파트에서 반복된 주제"를 Global Theme 후보로 활용한다.

### 4단계: 통합 구조 작성

**전체 핵심 요약 (Top Layer)**

Earnings Call 전체를 5~7줄로 압축. 회사의 핵심 메시지 중심.

**Global Theme 구조**

각 Theme는 다음 구조를 따른다:

```
▶ Theme N. [주제명]

① Meaning (전체 구조적 의미)
- 이 Theme가 전체 실적/사업에서 갖는 의미
- 2~3줄
- "부분 설명"이 아니라 "전체 맥락"

② Signal (통합된 경영진 메시지)
- CEO + CFO + Q&A에서 반복된 핵심 메시지
- 톤 / 강조점 / 방향성

③ Key Developments (핵심 전개)
- 이 Theme에서 실제로 발생한 주요 변화/논점
- 단순 Fact 나열 금지
- "무슨 일이 있었는지" 중심

④ Supporting Facts (근거)
- 핵심 숫자
- 핵심 발언
- 핵심 사례
※ 여기만 Fact 나열 허용
```

**Key Issues & Debates**

반복적으로 질문된 내용과 시장이 궁금해했던 포인트:

```
이슈 1: {이슈 설명}
  ○ 회사 답변 요지: {한 줄}

이슈 2: {이슈 설명}
  ○ 회사 답변 요지: {한 줄}
```

### 5단계: 숫자 규칙 (엄격)

- 숫자 변경 금지
- 단위 유지
- YoY / QoQ 유지
- 핵심 숫자만 포함

### 6단계: 파일 저장

출력 파일: `artifacts/{회사명}_{분기}/03_global_summary.md`

파일 상단 메타 정보:
```
# {회사명} {분기} — Global Theme 통합본
통합 일시: {날짜}
처리된 파트: {CEO, CFO, QA1, QA2, ...}
```

## 금지 사항

- 파트별 구조 유지
- 질문 순서 나열
- Theme 중복
- 모든 발언 포함하려는 시도
- 개인 의견 / 산업 전망
- 원문에 없는 해석

## 품질 기준

- 전체 핵심 요약만으로도 이 기업의 이번 분기 핵심을 파악할 수 있는가
- Global Theme가 파트 경계 없이 내용 기준으로 통합되었는가
- Key Issues & Debates가 Q&A의 주요 긴장점을 포착했는가
- 몇 달 뒤 이 문서 하나로 전체 맥락을 복원할 수 있는가
