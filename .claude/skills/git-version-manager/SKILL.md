---
name: git-version-manager
description: 어닝콜 하네스의 GitHub 버전 관리를 수행할 때 사용한다. "v[N]으로 올려줘", "깃허브에 저장해줘" 등의 Push 요청, "v[N]으로 롤백해줘", "이전 버전 가져와줘" 등의 Rollback 요청, "지금 버전 확인해줘" 등의 Status 조회 요청에 사용한다.
---

# Git Version Manager

## 개요

이 Skill은 어닝콜 하네스의 GitHub 버전 관리를 담당한다.
세 가지 작업을 처리한다: **Push**(버전 저장), **Rollback**(버전 복원), **Status**(현재 상태 조회).

```
로컬 경로: c:\Users\frisa\Documents\Earnings Call Translate
원격 저장소: https://github.com/bubbletea255/earnings-call-harness.git
기본 브랜치: main
```

---

## 트리거 분류

| 사용자 표현 | 실행 작업 |
|---|---|
| "v[N]으로 올려줘", "깃허브에 저장해줘", "지금 상태 저장해줘" | Operation A: Push |
| "v[N]으로 롤백해줘", "v[N] 가져와줘", "이전 버전으로 되돌려줘" | Operation B: Rollback |
| "지금 버전 확인해줘", "현재 상태 보여줘", "어떤 버전이야" | Operation C: Status |

---

## Operation A: Push (버전 저장)

### Step A-1: 원격 저장소 동기화

```bash
cd "c:\Users\frisa\Documents\Earnings Call Translate"
git fetch --tags
```

원격의 최신 태그 목록을 가져온다. 이 컴퓨터에서만 확인하면 다른 기기에서 올린 버전을 놓칠 수 있다.

### Step A-2: 버전 번호 교차검증

```bash
git tag --sort=version:refname
```

검증 규칙:

| 상황 | 처리 |
|---|---|
| 요청한 vN이 이미 존재 | "vN이 이미 있습니다. v[N+1]로 올릴까요?" |
| 갭 발생 (v4 없이 v5 요청) | "v4가 없습니다. v5 대신 v4로 올릴까요?" |
| 버전 미지정 ("올려줘"만) | "현재 v[최신]까지 있습니다. v[최신+1]로 올릴까요?" |
| 정상 | 다음 단계 진행 |

### Step A-3: 변경 내용 파악

```bash
git diff --unified=0 HEAD
git status --short
```

읽어야 할 대상:

- `.claude/skills/*/SKILL.md` — 어떤 규칙/절차가 추가·삭제·수정됐는지
- `.claude/agents/*.md` — 모델, 도구, 역할 설정 변경
- `CLAUDE.md` / `AGENTS.md` — 라우팅 규칙 변경

변경 파일이 없으면: "변경된 파일이 없습니다. 저장할 내용이 있는지 확인해주세요." 후 중단.

### Step A-4: 커밋 메시지 초안 작성

형식:
```
v[N]: [파일명1] — [무엇이 어떻게 바뀌었는지 핵심 1줄]
      [파일명2] — [무엇이 어떻게 바뀌었는지 핵심 1줄]
```

작성 기준:
- 파일명은 폴더 경로 생략, 스킬/에이전트 이름만
- "수정", "업데이트" 같은 모호한 표현 금지
- 추가/삭제/변경된 내용이 무엇인지 구체적으로 기술
- 변경 파일이 1개면 1줄, 여러 개면 각 줄로 나열

예시:
```
v3: translate-earnings — 볼드 처리 기준 6→8개 확장, 직역 금지 규칙 강화
    orchestrator — QA 최대 처리 파트 수 10→15개로 확장
```

### Step A-5: 사용자 확인 (게이트 — 반드시 멈추고 확인)

```
v[N]으로 저장합니다.

커밋 메시지 초안:
─────────────────────────────────
v[N]: [초안 내용]
─────────────────────────────────

이 메시지로 진행할까요?
수정이 필요하면 원하는 메시지를 알려주세요.
```

사용자 응답에 따라:
- "그대로 진행해" → 초안 그대로 실행
- 수정된 메시지 제시 → 해당 메시지로 실행
- "취소" → 중단

### Step A-6: 실행

```bash
git add .
git commit -m "[확정 메시지]"
git tag v[N]
git push origin main
git push origin --tags
```

### Step A-7: 검증

```bash
git log --oneline -1
git ls-remote --tags origin "refs/tags/v[N]"
```

성공 시:
```
✅ v[N] 저장 완료
   커밋: [hash] [메시지]
   GitHub 태그: v[N] 확인됨
```

태그 원격 확인 실패 시: "Push는 완료됐으나 태그 확인에 실패했습니다. 잠시 후 GitHub에서 직접 확인해주세요."

---

## Operation B: Rollback (버전 복원)

### Step B-1: 원격 저장소 동기화

```bash
cd "c:\Users\frisa\Documents\Earnings Call Translate"
git fetch --tags
```

### Step B-2: 대상 버전 검증

```bash
git tag --sort=version:refname
git show tags/v[대상] --stat
```

| 상황 | 처리 |
|---|---|
| 대상 버전이 존재하지 않음 | "v[N]이 없습니다. 존재하는 버전: [목록]" 후 중단 |
| 현재 버전과 동일 | "이미 v[N] 상태입니다." 후 중단 |
| 정상 | 다음 단계 진행 |

### Step B-3: 미저장 변경사항 확인

```bash
git status --short
```

미저장 변경사항이 있으면 반드시 아래 3가지 선택지를 제시하고 멈춘다:

```
저장 안 된 변경사항이 있습니다:
  [변경된 파일 목록]

롤백하면 이 내용은 사라집니다. 어떻게 할까요?

  1) 현재 상태를 v[N+1]로 먼저 저장 후 롤백 (권장 — 안전)
  2) 현재 변경사항을 버리고 롤백 (되돌릴 수 없음)
  3) 취소
```

- 선택 1 → Operation A (Push) 먼저 실행 후 이 Operation B를 이어서 진행
- 선택 2 → 한 번 더 확인: "현재 변경사항이 영구적으로 삭제됩니다. 정말 진행하시겠습니까?" → 확인 시 `git checkout .` 후 진행
- 선택 3 → 완전 중단

### Step B-4: 롤백 확인 (게이트 — 반드시 멈추고 확인)

```bash
git diff tags/v[대상] --stat
```

```
현재 v[현재] → v[대상] 으로 롤백합니다.

변경될 파일:
  [git diff 결과]

롤백 완료 후:
  - 현재 상태는 v[다음번호]로 자동 태그됩니다
  - v[현재] 태그는 보존됩니다 (언제든 다시 복원 가능)

진행할까요?
```

### Step B-5: 실행

현재 버전 번호 확인 후 다음 번호 계산:
```bash
git tag --sort=version:refname | tail -1
```

```bash
git checkout tags/v[대상] -- .
git add .
git commit -m "v[다음번호]: rollback v[현재]→v[대상]"
git tag v[다음번호]
git push origin main
git push origin --tags
```

### Step B-6: 검증

```bash
git diff tags/v[대상] -- .
```

diff 결과가 비어있으면 성공.

성공 시:
```
✅ 롤백 완료
   현재 상태: v[다음번호] (내용: v[대상]과 동일)
   보존된 버전: v[현재] (태그 유지 — 필요 시 언제든 복원 가능)
```

diff 결과가 비어있지 않으면: "파일 내용 불일치가 감지됐습니다. 확인이 필요한 파일: [목록]. 재시도하거나 직접 확인해주세요."

---

## Operation C: Status (현재 상태 조회)

### 실행

```bash
cd "c:\Users\frisa\Documents\Earnings Call Translate"
git fetch --tags
git tag --sort=version:refname
git log --oneline -5
git status --short
```

### 보고 형식

```
📋 현재 버전 상태

현재 버전: v[N]
마지막 커밋: [날짜] — [메시지]

버전 히스토리 (최근순):
  v[N]   [날짜] [커밋 메시지]
  v[N-1] [날짜] [커밋 메시지]
  v[N-2] [날짜] [커밋 메시지]

저장 안 된 변경사항: [없음 / 파일 목록]
```

---

## 오류 처리

| 상황 | 처리 방식 |
|---|---|
| Push 거부 (원격이 앞서 있음) | "`git pull` 후 다시 시도해주세요. 원격 저장소에 이 컴퓨터에 없는 변경사항이 있습니다." |
| 네트워크 오류 | 1회 자동 재시도. 재시도 실패 시 "네트워크 오류입니다. 인터넷 연결을 확인 후 다시 요청해주세요." |
| 인증 오류 | "GitHub 인증이 만료됐습니다. GitHub Desktop을 열어 재로그인 후 다시 시도해주세요." |
| 태그 충돌 | 다음 사용 가능한 버전 번호를 자동 제안 |
| 변경 파일 없음 (Push 시) | "저장할 변경사항이 없습니다." 후 중단 |
| 대상 버전 없음 (Rollback 시) | "해당 버전이 존재하지 않습니다." + 존재하는 버전 목록 표시 후 중단 |

---

## 품질 기준

- 모든 작업은 Step A-5 또는 Step B-4의 사용자 확인 게이트를 반드시 통과해야 한다
- 사용자 확인 없이 `git push`를 실행하지 않는다
- 모든 Push/Rollback 후 검증(Step A-7, Step B-6)을 반드시 실행한다
- 오류 발생 시 사용자가 직접 복구할 수 있도록 구체적인 안내를 제공한다
