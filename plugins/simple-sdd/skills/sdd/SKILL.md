---
name: sdd
description: >
  Simple SDD 전체 흐름(grilling → spec → plan → tasks)을 순서대로 진행하는 진입 스킬.
  This skill should be used when the user says "SDD로 설계해줘", "SDD 시작", "스펙부터 잡자",
  "spec plan task 만들어줘", "이 기능 설계하자", "/simple-sdd:sdd", or asks to resume an SDD flow
  ("SDD 이어서 해줘", "다음 단계 진행해줘") for a feature under specs/.
metadata:
  version: "0.2.0"
---

# Simple SDD 진행

Run the four design steps in order, keeping each step's output in a file so later steps (and later sessions) read files instead of relying on conversation memory. One hub file indexes every feature, so a later session reads the hub and opens only the feature it needs.

```
<specs root>/
├── update-hub.md          # 모든 기능의 설명, 상태, 코드 경로, spec 링크
└── NNN-<feature-slug>/
    ├── decisions.md       # 1단계 grilling
    ├── spec.md            # 2단계 make-spec  (무엇을, 왜)
    ├── plan.md            # 3단계 make-plan  (어떻게)
    └── tasks.md           # 4단계 make-tasks (무엇을 어떤 순서로)
```

## 0. 준비

1. Find the project root (git root, or the current working directory).
2. Find or choose the **specs root** (the folder that holds `update-hub.md`) and the **문서 모드**. First match wins:
   1. `update-hub.md` already exists: look in `specs/` and `docs/specs/`, then search the project (skip `.git`, `node_modules`, build output). Use its folder. The mode is already fixed: **Obsidian 모드** if the hub starts with YAML frontmatter (`---`), otherwise **일반 모드**.
   2. `specs/` has `NNN-*` folders but no hub (made by v0.1): use `specs/`, pick the mode by rules 3-4, and build the hub once from the existing folders (description from each `decisions.md` 개요 or `spec.md` 배경, status from `tasks.md` checkboxes, code paths from `plan.md` 파일 구조).
   3. Obsidian is installed, i.e. one of these files exists: Windows `%APPDATA%\obsidian\obsidian.json`, macOS `~/Library/Application Support/obsidian/obsidian.json`, Linux `~/.config/obsidian/obsidian.json`. Use **Obsidian 모드**. Read the vault paths (`vaults.*.path`) from that file:
      - a vault inside the project root (e.g. `<project>/docs`) → `<vault>/specs/`
      - the project root is itself a vault or inside one → `specs/`
      - otherwise → `docs/specs/`, and tell the user in one line: "Obsidian에서 `docs` 폴더를 vault로 열면 명세를 볼 수 있어요."

      Never edit `obsidian.json` or any `.obsidian/` file.
   4. Otherwise: **일반 모드**, `specs/`.

   When the hub does not exist yet, create it from the template under "update-hub.md" below.
3. If the project root's `CLAUDE.md` has no `## 명세 읽는 순서` section, append the section under "CLAUDE.md 섹션" below (create `CLAUDE.md` with only that section if there is none). Tell the user in one line.
4. Decide whether this is a new feature or a resume:
   - **Resume**: the user names an existing feature, or a folder or hub entry matches the request. Check which of the four files exist and start from the first missing step. Tell the user in one line which step it resumes from.
   - **New**: take the highest `NNN` prefix in the specs root + 1 (start at `001`), and create `NNN-<feature-slug>/` with a short English kebab-case slug (e.g. `003-user-login`). Add its hub entry (상태 `설계중`, a one-line description from the request). State the folder name in one line.
5. Read the 진행 방식 from the request:
   - **기본** (default): stop at the checkpoints below.
   - **자동**: the user said "자동", "auto", "끝까지 한 번에", or similar. Skip checkpoints B and C, but still stop at checkpoint A and whenever a step reports open questions.

## 1. grilling → decisions.md

Invoke the `grilling` skill on the feature. Pass the feature folder path and the 문서 모드 so grilling writes `decisions.md` at the end (see its "Simple SDD handoff" section). If the user already gave a complete, written-out design, still run grilling; its first rounds will be short.

**Checkpoint A (항상)**: grilling itself requires the user's confirmation of shared understanding before `decisions.md` is written. Do not proceed until `decisions.md` exists. Then replace the hub entry's description with 1-2 sentences from the 개요.

## 2. make-spec → spec.md

Invoke the `make-spec` skill.

- If make-spec reports `[확인 필요]` items, ask them as one numbered round (with a recommended answer each, as in grilling), update `spec.md` and `decisions.md` with the answers, then continue.
- **Checkpoint B (기본만)**: no stop by default after spec, because grilling already settled the decisions. Show a 3-5 line summary (FR 개수, 핵심 목표, 비목표) and continue straight to step 3.

## 3. make-plan → plan.md

Invoke the `make-plan` skill.

- **Checkpoint C (기본)**: show the technology choices, the Phase list, and the FR→Phase mapping table, then ask: "이 계획으로 tasks를 만들까요?" Wait for the answer. Apply requested changes to `plan.md` before step 4.

## 4. make-tasks → tasks.md

Invoke the `make-tasks` skill.

## 5. 마무리

Report briefly:
- the feature folder path and the four files
- task count by Phase, and which tasks can start right away (no unmet dependencies)
- the next action: "`implement-task`로 T-01부터 구현할 수 있어요 (예: 'T-01 구현해줘')."

Do not start implementing in this skill.

## 문서 머리말

Every skill writes the top of a document according to the 문서 모드. A feature's 상태 lives only in the hub, never in the documents.

- **일반 모드**: the line under the title, `> 근거: <previous doc> | 작성일: YYYY-MM-DD`.
- **Obsidian 모드**: YAML frontmatter before the title, and the 근거 line keeps only the link:

  ```markdown
  ---
  type: spec            # decisions | spec | plan | tasks
  feature: 003-user-login
  tags: [sdd]
  ---
  # Spec: <feature name>

  > 근거: [decisions](decisions.md)
  ```

  Write each `[확인 필요]` item as a callout: `> [!question] [확인 필요] <question>`.
- **Both modes**: link with relative Markdown links (`[spec](spec.md)`), never `[[wikilinks]]`. Every feature folder has a `spec.md`, so a wikilink by name points at the wrong feature.

## update-hub.md

The hub is an index, not a summary: never copy requirements or decisions into it. Keep each 진행 중 entry to about three lines and each 완료 entry to one line, so the hub stays small as features pile up.

```markdown
---
type: hub
tags: [sdd]
---
# Update Hub

> 기능 폴더마다 decisions.md · spec.md · plan.md · tasks.md가 있다. simple-sdd 스킬이 갱신한다.

## 진행 중

### 004-cloud-sync · 구현중
여러 PC와 폰의 일정을 동기화한다. 충돌하면 마지막 수정이 이긴다. · [spec](004-cloud-sync/spec.md)
- 코드: `src/Infrastructure/Sync/`, `src/App/Settings/SyncPage.*`

## 완료

- 003-timetable — 시간표 보기 추가 · [spec](003-timetable/spec.md) · 코드: `src/App/Timetable/`
```

- The frontmatter is written only in Obsidian 모드; in 일반 모드 the file starts at `# Update Hub`.
- 상태: `설계중` → `구현중` → `완료`.
- 코드: directories or file globs from plan.md "파일 / 모듈 구조", not every file. Before plan.md exists, write `코드: (plan 작성 후)`.
- 링크: only `spec.md`, added by make-spec once it exists. The other documents sit next to it under fixed names, so they are not linked.

| 시점 | 스킬 | hub 변경 |
|------|------|---------|
| 기능 폴더 생성 | sdd | 항목 추가 (`설계중`) |
| decisions.md 작성 | sdd | 설명을 개요 1-2문장으로 교체 |
| spec.md 작성 | make-spec | spec 링크 추가 |
| plan.md 작성·수정 | make-plan, implement-task | 코드 경로 갱신 |
| 첫 작업 시작 | implement-task | `구현중` |
| 마지막 작업 완료 | implement-task | 완료 섹션으로 옮겨 한 줄로 |

## CLAUDE.md 섹션

Write `<hub path>` relative to the project root.

```markdown
## 명세 읽는 순서
1. 작업 전에 `<hub path>`를 읽는다.
2. 설명이나 코드 경로가 작업과 겹치는 기능의 문서만 연다. 겹치는 기능이 없으면 명세는 읽지 않는다.
3. 기능 폴더 안에서는 필요한 ID(FR-03, D-02, T-05)를 찾아 그 부분만 읽는다. 문서 전체는 설계를 바꿀 때만 읽는다.
```

## 규칙

- Each step reads the previous step's file from disk, not the conversation.
- Never fill a gap with a guess. Anything the user did not decide becomes a question to the user, not an assumption in a document.
- Finding facts in the codebase or environment is Claude's job; deciding is the user's.
- Write all documents in the user's language (Korean by default). Keep code identifiers, file paths, and IDs (D-01, FR-01, NFR-01, P1, T-01) as they are.
