---
name: earnings-orchestrator
description: 어닝콜 분석 전체 파이프라인(1~5단계)을 실행할 때 사용한다. "어닝콜 분석 실행해줘", "input 폴더 처리해줘", "실적발표 자동화 시작해줘", "{기업명} 분석해줘" 같은 요청뿐 아니라, "{기업명} 4~5단계 재실행해줘", "이전 결과 기반으로 다시 분석해줘", "특정 단계만 다시 실행해줘" 같은 재실행 요청에도 사용한다.
---

# Earnings Call Orchestrator

## 목적

이 Orchestrator는 어닝콜 대본 텍스트 파일을 입력으로 받아 5단계 분석 파이프라인을 자동 실행하고 기업별 산출물을 `artifacts/` 폴더에 저장한다.

여러 기업을 동시에 병렬 처리한다. 각 기업 내부는 순차 처리한다.

## 실행 모드 확인

Orchestrator 시작 시 아래 분기를 먼저 확인한다:

**신규 실행**: `input/` 폴더에 .txt 파일이 있고, 해당 기업의 artifacts 폴더가 없거나 비어 있는 경우
→ 전체 1~5단계를 실행한다

**재실행 (전체)**: 사용자가 "다시 실행", "새로 실행"을 요청한 경우
→ 기존 artifacts를 `artifacts/{회사명}_{분기}/archive/{YYYYMMDD-HHMMSS}/`에 보존하고 처음부터 실행한다

**재실행 (부분)**: 사용자가 "{기업명} 4~5단계 재실행해줘. {추가 지시}" 형식으로 요청한 경우
→ 해당 기업의 1~3단계 산출물은 유지하고, 04_analysis.md와 05_judgment.md만 재생성한다
→ 기존 파일은 `04_analysis_v{N}.md`, `05_judgment_v{N}.md`로 보존한다
→ 선택적 추가 지시를 earnings-analyst Agent에 전달한다

**의도 불분명**: 기존 산출물이 있고 요청이 모호한 경우
→ "이어 하기 / 부분 재실행 / 새 실행 중 무엇을 원하시나요?" 확인 후 진행한다

---

## 전체 실행 흐름

### Phase 0: 입력 파악

```
1. Glob: input/*.txt 로 처리할 기업 목록을 확인한다
2. 각 파일명에서 회사명과 분기를 파싱한다: {회사명}_{분기}.txt
3. artifacts/ 폴더에서 기존 산출물 여부를 확인한다
4. 실행 모드를 결정한다 (신규 / 재실행 / 부분 재실행)
5. artifacts/{회사명}_{분기}/ 폴더를 생성한다
```

input 파일이 없으면: "input/ 폴더에 처리할 .txt 파일이 없습니다. 파일을 추가하고 다시 실행해주세요." 메시지를 출력하고 종료한다.

### Phase 1: 기업별 병렬 실행

input 폴더의 모든 기업에 대해 Phase 2~7을 **병렬로** 실행한다.

각 기업 내부는 아래 순서로 **순차** 실행한다.

---

### Phase 2: 1단계 — 파트별 완전 번역 (병렬)

**목표**: 원문을 파트별로 나눠 각각 한국어로 완전 번역

```
실행 Agent: earnings-translator
사용 Skill: translate-earnings
모델: claude-haiku-4-5-20251001
```

**절차**:

1. 원문 파일을 읽는다: `input/{회사명}_{분기}.txt`

2. 파트 구조를 파악한다 (TOC 작성):
   - 원문을 훑어 발화자 전환 지점을 식별한다
   - 일반적인 구조: [Operator] [CEO] [CFO] [Operator/Q&A 시작] [Q&A: 질문자1] [Q&A: 질문자2] ...
   - 파트 목록을 확정한다: CEO, CFO, QA1(이름/소속), QA2(이름/소속), ...

3. 각 파트에 대해 **병렬로** earnings-translator Agent를 호출한다:
   - 입력: 해당 파트의 원문 텍스트
   - 출력: `artifacts/{회사명}_{분기}/01_translation_{파트}.md`
   - 파트명 규칙: CEO, CFO, QA1, QA2, QA3, ... (번호는 Q&A 순서)

4. 모든 파트 번역이 완료되면 Phase 3으로 진행한다.

**완료 기준**: 파트 수만큼 01_translation_*.md 파일이 생성되었는가

---

### Phase 3: Topic Pre-scan (순차)

**목표**: 전체 번역본을 훑어 Q&A 간 연결 관계를 파악

```
실행 Agent: earnings-topic-scanner
사용 Skill: topic-prescan
모델: claude-sonnet-4-6
```

**절차**:

1. Glob으로 모든 번역 파일을 수집한다: `artifacts/{회사명}_{분기}/01_translation_*.md`
2. topic-prescan 스킬을 따라 Cross-QA 연결을 파악한다
3. 저장: `artifacts/{회사명}_{분기}/topic_map.md`

**완료 기준**: topic_map.md 파일이 생성되었는가

---

### Phase 4: 2단계 — 파트별 구조화 요약 (병렬)

**목표**: 각 번역본을 Theme 구조로 요약 (topic_map 참조)

```
실행 Agent: earnings-summarizer
사용 Skill: summarize-parts
모델: claude-haiku-4-5-20251001
```

**절차**:

1. 각 파트에 대해 **병렬로** earnings-summarizer Agent를 호출한다:
   - 입력: `artifacts/{회사명}_{분기}/01_translation_{파트}.md` + `topic_map.md`
   - 출력: `artifacts/{회사명}_{분기}/02_summary_{파트}.md`

2. 모든 파트 요약이 완료되면 Phase 5로 진행한다.

**완료 기준**: 번역 파일 수만큼 02_summary_*.md 파일이 생성되었는가

---

### Phase 5: 3단계 — Global Theme 통합 (순차)

**목표**: 모든 요약본을 통합해 Global Theme 통합본 생성

```
실행 Agent: earnings-integrator
사용 Skill: integrate-themes
모델: claude-sonnet-4-6
```

**절차**:

1. earnings-integrator Agent를 호출한다:
   - 입력: `artifacts/{회사명}_{분기}/02_summary_*.md` 전체 + `topic_map.md`
   - 출력: `artifacts/{회사명}_{분기}/03_global_summary.md`

**완료 기준**: 03_global_summary.md 파일이 생성되었는가

---

### Phase 6: 4단계 — 구조 분석 + 쟁점 도출 (순차)

**목표**: 통합본 기반 구조적 분석 및 분기 특화 쟁점 도출

```
실행 Agent: earnings-analyst
사용 Skill: analyze-structure
모델: claude-opus-4-7
```

**절차**:

1. 선택적 추가 지시 확인 (재실행 시 사용자가 제공한 추가 지시)
2. earnings-analyst Agent를 호출한다:
   - 입력: `artifacts/{회사명}_{분기}/03_global_summary.md` + `02_summary_*.md` 전체
   - 선택적 추가 지시: 있으면 Agent에 전달, 없으면 기본 구조로 진행
   - 출력: `artifacts/{회사명}_{분기}/04_analysis.md`

**완료 기준**: 04_analysis.md 파일이 생성되었는가

---

### Phase 7: 5단계 — 최종 투자 판단 (순차)

**목표**: 4단계 분석 결과만을 기반으로 투자 판단

```
실행 Agent: earnings-judge
사용 Skill: judge-investment
모델: claude-opus-4-7
```

**절차**:

1. earnings-judge Agent를 호출한다:
   - 입력: `artifacts/{회사명}_{분기}/04_analysis.md` (이것만)
   - 출력: `artifacts/{회사명}_{분기}/05_judgment.md`

**완료 기준**: 05_judgment.md 파일이 생성되었는가

---

### Phase 8: 완료 처리

각 기업에 대해:

1. `artifacts/{회사명}_{분기}/README.md`를 생성한다:

```md
# {회사명} {분기} 분석 완료

실행 일시: {날짜 및 시간}
처리된 파트: {CEO, CFO, QA1, QA2, ...}
추가 지시: {있으면 내용 / 없으면 "없음"}

## 파일 목록

| 파일 | 설명 |
|---|---|
| topic_map.md | Cross-QA 연결 지도 |
| 01_translation_CEO.md | CEO 번역본 |
| 01_translation_CFO.md | CFO 번역본 |
| 01_translation_QA*.md | Q&A 번역본 |
| 02_summary_CEO.md | CEO 요약본 |
| 02_summary_CFO.md | CFO 요약본 |
| 02_summary_QA*.md | Q&A 요약본 |
| 03_global_summary.md | Global Theme 통합본 ← 스크리닝 시작점 |
| 04_analysis.md | 구조 분석 + 쟁점 도출 |
| 05_judgment.md | 최종 투자 판단 |

## 읽기 순서 추천

1. 03_global_summary.md (스크리닝)
2. 02_summary_*.md (깊은 이해)
3. 01_translation_*.md (정밀 검토)
```

2. 모든 기업 처리가 완료되면 최종 완료 메시지를 출력한다:

```
✅ {N}개 기업 어닝콜 분석 완료

처리된 기업:
- {회사명1}_{분기1}: artifacts/{회사명1}_{분기1}/
- {회사명2}_{분기2}: artifacts/{회사명2}_{분기2}/
...

읽기 시작점: 각 폴더의 03_global_summary.md
심층 분석 재실행: "{기업명} 4~5단계 재실행해줘. {추가 지시}"
```

---

## 오류 처리

| 상황 | 처리 방식 |
|---|---|
| input/*.txt 없음 | 안내 메시지 출력 후 종료 |
| 특정 파트 번역 실패 | 1회 재시도, 재시도 실패 시 누락으로 표시하고 계속 진행 |
| topic_map 생성 실패 | 빈 topic_map.md로 대체하고 다음 단계 진행 |
| 4단계 또는 5단계 실패 | 오류 내용 보고, 재실행 여부를 사용자에게 확인 |
| 기존 파일 충돌 | 자동으로 archive 폴더에 보존 후 새 파일 생성 |

---

## Task 등록 계약

| Task | 담당 Agent | 입력 | 출력 | 의존 | 모델 |
|---|---|---|---|---|---|
| T01-parse | Orchestrator | input/*.txt | 파트 목록 | 없음 | - |
| T02-translate | earnings-translator (병렬) | 파트 원문 | 01_translation_*.md | T01 | haiku |
| T03-prescan | earnings-topic-scanner | 01_translation_*.md | topic_map.md | T02 | sonnet |
| T04-summarize | earnings-summarizer (병렬) | 01_translation_*.md + topic_map | 02_summary_*.md | T03 | haiku |
| T05-integrate | earnings-integrator | 02_summary_*.md | 03_global_summary.md | T04 | sonnet |
| T06-analyze | earnings-analyst | 03_global_summary.md + 02_summary_*.md | 04_analysis.md | T05 | opus |
| T07-judge | earnings-judge | 04_analysis.md | 05_judgment.md | T06 | opus |
| T08-wrap | Orchestrator | 모든 산출물 | README.md | T07 | - |

---

## 부분 재실행 상세

사용자가 "{기업명}_{분기} 4~5단계 재실행해줘. {추가 지시}" 요청 시:

1. 해당 기업의 `artifacts/{회사명}_{분기}/` 폴더를 확인한다
2. 03_global_summary.md와 02_summary_*.md 파일이 있는지 확인한다
3. 기존 04_analysis.md가 있으면 `04_analysis_v{N}.md`로 이름을 변경해 보존한다
4. 기존 05_judgment.md가 있으면 `05_judgment_v{N}.md`로 이름을 변경해 보존한다
5. 추가 지시를 전달해 T06-analyze부터 다시 실행한다
6. T07-judge를 실행한다
7. README.md를 업데이트한다

추가 지시 예시:
- "AI 수익화 발언에 집중해서 분석해줘"
- "마진 구조에 초점을 맞춰라"
- "가이던스 신뢰도만 집중 평가해줘"
- "부채 구조 및 유동성 리스크에 초점을 맞춰라"
