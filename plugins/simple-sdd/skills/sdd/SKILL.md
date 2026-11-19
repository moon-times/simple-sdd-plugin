---
name: sdd
description: >
  Simple SDD 전체 흐름(grilling → spec → plan → tasks)을 순서대로 진행하는 진입 스킬.
  This skill should be used when the user says "SDD로 설계해줘", "SDD 시작", "스펙부터 잡자",
  "spec plan task 만들어줘", "이 기능 설계하자", "/simple-sdd:sdd", or asks to resume an SDD flow
  ("SDD 이어서 해줘", "다음 단계 진행해줘") for a feature under specs/.
metadata:
  version: "0.1.0"
---

# Simple SDD 진행

Run the four design steps in order, keeping each step's output in a file so later steps (and later sessions) read files instead of relying on conversation memory.

```
specs/NNN-<feature-slug>/
├── decisions.md   # 1단계 grilling
├── spec.md        # 2단계 make-spec  (무엇을, 왜)
├── plan.md        # 3단계 make-plan  (어떻게)
└── tasks.md       # 4단계 make-tasks (무엇을 어떤 순서로)
```

## 0. 준비

1. Find the project root (git root, or the current working directory).
2. Decide whether this is a new feature or a resume:
   - **Resume**: the user names an existing feature, or a folder under `specs/` matches the request. Check which of the four files exist and start from the first missing step. Tell the user in one line which step it resumes from.
   - **New**: list `specs/`, take the highest `NNN` prefix + 1 (start at `001`), and create `specs/NNN-<feature-slug>/`. Use a short English kebab-case slug derived from the feature (e.g. `003-user-login`). State the folder name in one line.
3. Read the mode from the request:
   - **기본 모드** (default): stop at the checkpoints below.
   - **자동 모드**: the user said "자동", "auto", "끝까지 한 번에", or similar. Skip checkpoints B and C, but still stop at checkpoint A and whenever a step reports open questions.

## 1. grilling → decisions.md

Invoke the `grilling` skill on the feature. Pass the feature folder path so grilling writes `decisions.md` at the end (see its "Simple SDD handoff" section). If the user already gave a complete, written-out design, still run grilling; its first rounds will be short.

**Checkpoint A (항상)**: grilling itself requires the user's confirmation of shared understanding before `decisions.md` is written. Do not proceed until `decisions.md` exists.

## 2. make-spec → spec.md

Invoke the `make-spec` skill.

- If make-spec reports `[확인 필요]` items, ask them as one numbered round (with a recommended answer each, as in grilling), update `spec.md` and `decisions.md` with the answers, then continue.
- **Checkpoint B (기본 모드만)**: no stop by default after spec, because grilling already settled the decisions. Show a 3-5 line summary (FR 개수, 핵심 목표, 비목표) and continue straight to step 3.

## 3. make-plan → plan.md

Invoke the `make-plan` skill.

- **Checkpoint C (기본 모드)**: show the technology choices, the Phase list, and the FR→Phase mapping table, then ask: "이 계획으로 tasks를 만들까요?" Wait for the answer. Apply requested changes to `plan.md` before step 4.

## 4. make-tasks → tasks.md

Invoke the `make-tasks` skill.

## 5. 마무리

Report briefly:
- the feature folder path and the four files
- task count by Phase, and which tasks can start right away (no unmet dependencies)
- the next action: "`implement-task`로 T-01부터 구현할 수 있어요 (예: 'T-01 구현해줘')."

Do not start implementing in this skill.

## 규칙

- Each step reads the previous step's file from disk, not the conversation.
- Never fill a gap with a guess. Anything the user did not decide becomes a question to the user, not an assumption in a document.
- Finding facts in the codebase or environment is Claude's job; deciding is the user's.
- Write all documents in the user's language (Korean by default). Keep code identifiers, file paths, and IDs (D-01, FR-01, NFR-01, P1, T-01) as they are.
