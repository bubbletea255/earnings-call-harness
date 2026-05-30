# Earnings Call Translate

이 프로젝트는 어닝콜 대본을 자동으로 번역·요약·분석하는 하네스입니다.

## 자연어 라우팅

아래 요청이 들어오면 `earnings-orchestrator`를 먼저 사용합니다:

- "어닝콜 분석 실행해줘"
- "input 폴더 처리해줘"
- "실적발표 자동화 시작해줘"
- "{기업명} {분기} 어닝콜 분석해줘"
- "{기업명} 4~5단계 재실행해줘. {추가 지시}"
- "이전 결과 기반으로 {기업명} 다시 분석해줘"
- "특정 단계만 다시 실행해줘"

## 주요 위치

| 목적 | 경로 |
|---|---|
| 입력 파일 | `input/{회사명}_{분기}.txt` |
| Orchestrator | `.claude/skills/earnings-orchestrator/SKILL.md` |
| Agent 정의 | `.claude/agents/` |
| 산출물 | `artifacts/{회사명}_{분기}/` |
| 버전 관리 스킬 | `~/.claude/skills/git-version-manager/SKILL.md` (전역) |
| GitHub 저장소 | `https://github.com/bubbletea255/earnings-call-harness` |

## 사람의 작업

1. `input/` 폴더에 `.txt` 파일 넣기 (파일명: `unity_2025Q1.txt` 형식)
2. "어닝콜 분석 실행해줘" 한 마디
3. 완료 후 `artifacts/` 에서 결과 확인

### 추천 읽기 순서

1. `03_global_summary.md` — 전체 스크리닝 (모든 기업)
2. `02_summary_{파트}.md` — 깊은 이해 (관심 기업)
3. `01_translation_{파트}.md` — 정밀 검토 (투자 후보 기업)

### 심층 재분석 요청 예시

```
"unity_2025Q1 4~5단계 재실행해줘. AI 수익화 발언에 집중해서 분석해줘."
"datadog_2025Q2 4~5단계 재실행. 가이던스 신뢰도만 집중 평가해줘."
```

## 산출물 구조

```
artifacts/{회사명}_{분기}/
├── README.md                        실행 요약 및 파일 목록
├── topic_map.md                     Cross-QA 연결 지도 (Pre-scan 결과)
├── 01_translation_CEO.md            1단계: CEO 번역본
├── 01_translation_CFO.md            1단계: CFO 번역본
├── 01_translation_QA1.md            1단계: Q&A1 번역본
├── 01_translation_QA{N}.md          (파트 수에 따라 자동 생성)
├── 02_summary_CEO.md                2단계: CEO 요약본
├── 02_summary_CFO.md                2단계: CFO 요약본
├── 02_summary_QA1.md                2단계: Q&A1 요약본
├── 02_summary_QA{N}.md              (파트 수에 따라 자동 생성)
├── 03_global_summary.md             3단계: Global Theme 통합본
├── 04_analysis.md                   4단계: 구조 분석 + 쟁점 도출
└── 05_judgment.md                   5단계: 투자 판단
```

## 하네스 변경 이력

| 날짜 | 변경 내용 | 대상 | 사유 |
|---|---|---|---|
| 2026-05-26 | 초기 구성 | 전체 | 어닝콜 반복 분석 자동화 하네스 생성 |
| 2026-05-30 | git-version-manager 스킬 추가 | CLAUDE.md, .claude/skills/ | GitHub 버전 관리 자동화 |
| 2026-05-30 | git-version-manager 전역 이동 | ~/.claude/skills/ | 모든 프로젝트에서 재사용 가능하도록 전역화 |
| 2026-05-31 | .gitignore에 docs/ 추가 | .gitignore | /저장 명령 대화 기록 GitHub 노출 방지 |
