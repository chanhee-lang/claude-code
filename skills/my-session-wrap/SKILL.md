---
name: my-session-wrap
description: 세션 종료 시 작업 정리, 문서 업데이트, 학습 기록을 하는 스킬. "/wrap", "세션 정리", "마무리" 요청에 사용.
---

# My Session Wrap

세션이 끝날 때 multi-agent로 작업을 자동 정리하는 스킬.

## Execution Flow

```
┌─────────────────────────────────────────────────┐
│  1. Git 상태 확인 (오늘 뭘 했나?)                  │
├─────────────────────────────────────────────────┤
│  2. Phase 1: 4개 분석 에이전트 병렬 실행           │
│     ┌────────────────┬────────────────┐          │
│     │  doc-updater   │  automation-   │          │
│     │  (문서 업데이트) │  scout(자동화)  │          │
│     ├────────────────┼────────────────┤          │
│     │  learning-     │  followup-     │          │
│     │  extractor     │  suggester     │          │
│     └────────────────┴────────────────┘          │
├─────────────────────────────────────────────────┤
│  3. Phase 2: 검증 에이전트 순차 실행              │
│     ┌─────────────────────────────────┐          │
│     │       duplicate-checker         │          │
│     └─────────────────────────────────┘          │
├─────────────────────────────────────────────────┤
│  4. 결과 통합                                     │
├─────────────────────────────────────────────────┤
│  5. 사용자 선택 + 실행                            │
└─────────────────────────────────────────────────┘
```

## Step 1: Git 상태 확인

```bash
git status --short
git diff --stat HEAD~3 2>/dev/null || git diff --stat
```

변경 내용을 바탕으로 세션 요약을 작성한다:

```
세션 요약:
- 작업: [이번 세션에서 한 주요 작업]
- 파일: [생성/수정된 파일 목록]
- 결정: [주요 결정 사항]
```

## Step 2: Phase 1 — 4개 분석 에이전트 병렬 실행

4개 에이전트를 동시에(한 메시지에 4개 Task 호출) 실행한다.

```
Task(doc-updater):
  "세션 요약을 바탕으로 CLAUDE.md, MEMORY.md 업데이트가 필요한 부분을 찾아줘."

Task(automation-scout):
  "세션 요약을 바탕으로 반복 패턴이나 자동화할 수 있는 작업을 찾아줘."

Task(learning-extractor):
  "세션 요약을 바탕으로 오늘 배운 것, 실수, 새로운 발견을 TIL 형식으로 정리해줘."

Task(followup-suggester):
  "세션 요약을 바탕으로 미완료 작업과 다음 세션 우선순위를 제안해줘."
```

### 에이전트 역할

| 에이전트 | 역할 | 출력 |
|---------|------|------|
| **doc-updater** | CLAUDE.md/MEMORY.md 업데이트 분석 | 추가할 구체적 내용 |
| **automation-scout** | 자동화 패턴 탐지 | 스킬/커맨드/에이전트 제안 |
| **learning-extractor** | 학습 포인트 추출 | TIL 형식 요약 |
| **followup-suggester** | 후속 작업 제안 | 우선순위 목록 |

## Step 3: Phase 2 — 검증 에이전트 순차 실행

Phase 1 완료 후 실행한다.

```
Task(duplicate-checker):
  "Phase 1 결과를 검증해줘.

  doc-updater 제안: [결과]
  automation-scout 제안: [결과]
  learning-extractor 결과: [결과]
  followup-suggester 제안: [결과]

  중복 확인:
  1. 완전 중복 → 제거 권고
  2. 부분 중복 → 병합 방법 제안
  3. 중복 없음 → 추가 승인"
```

## Step 4: 결과 통합

```markdown
## 세션 랩 분석 결과

### 문서 업데이트
[doc-updater 요약]
- 중복 체크: [duplicate-checker 피드백]

### 자동화 제안
[automation-scout 요약]
- 중복 체크: [duplicate-checker 피드백]

### 오늘 배운 것
[learning-extractor 요약]

### 다음 할 일
[followup-suggester 요약]
```

## Step 5: 사용자 선택 + 실행

```
AskUserQuestion(
  question: "어떤 작업을 진행할까요?",
  multiSelect: true,
  options: [
    "커밋하기 (Recommended)",
    "MEMORY.md 업데이트",
    "자동화 생성",
    "Skip"
  ]
)
```

선택된 작업만 실행한다.

---

## Quick Reference

### 언제 사용하면 좋은가
- 긴 작업 세션이 끝날 때
- 다른 프로젝트로 전환하기 전
- 기능 완성이나 버그 수정 후

### 언제 건너뛰어도 되는가
- 짧은 질문만 한 경우
- 코드만 읽은 경우
- 간단한 일회성 작업
