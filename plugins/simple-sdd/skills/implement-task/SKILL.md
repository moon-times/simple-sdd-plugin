---
name: implement-task
description: >
  tasks.md의 작업 하나를 구현하고, 완료 조건을 확인한 뒤 체크하고, 계획과 달라진 점을 문서에 반영하는 스킬.
  This skill should be used when the user says "T-03 구현해줘", "다음 task 진행해줘", "다음 작업 해줘",
  "tasks.md 이어서 구현", or otherwise asks to implement a task from a specs/ feature folder.
metadata:
  version: "0.2.0"
---

# task 구현

Implement exactly one task from `tasks.md`, verify it, and keep the documents in sync with the code.

## 1. 작업 고르기

1. Locate the feature folder: read `update-hub.md` in the specs root (`specs/` when there is no hub) and pick the feature from it. If several are 진행 중 with unchecked tasks and the user did not say which, ask. The 문서 모드 is **Obsidian** when the hub starts with YAML frontmatter, otherwise **일반**.
2. Read `tasks.md`, then only what the task needs: the FR/NFR it cites with their 수용 기준 in `spec.md`, the decisions they cite, and the `plan.md` sections for its files and interfaces. Read a whole document only when the task cannot be understood otherwise.
3. Pick the task:
   - A named ID (e.g. "T-03"): use it.
   - "다음": the first unchecked task whose dependencies are all checked.
4. If any dependency is unchecked, stop and say which task should come first. Do not implement it silently.

## 2. 구현

- Do only what the task's 내용 says, in the files it lists. A small change to an unlisted file is fine when required; note it in the report.
- Follow plan.md's 기술 선택, interfaces, and the codebase's existing conventions.
- If the task cannot be done as planned (wrong assumption, missing API, better approach found), stop before diverging. Explain the problem in a few lines, propose the change, and wait for the user's decision when it affects the interface, data model, dependencies, or other tasks. Minor internal choices do not need to wait.

## 3. 검증

- Run the check in 완료 조건 exactly (tests, commands, requests). Also run the project's lint/type check if plan.md lists one.
- If the check fails, fix and re-run. After two failed fix attempts, stop and report what fails and why.
- Check the box only when the 완료 조건 actually passed. Never check a box based on "should work".

## 4. 문서 반영

- `tasks.md`: change `- [ ]` to `- [x]` for the task. If new follow-up work was discovered, append it as a new task with a new ID (never renumber).
- `plan.md`: if the implementation differs from the plan (file names, interfaces, choices), update the affected section and add a line to 9. 변경 기록: `YYYY-MM-DD T-03: <what changed and why>`.
- `spec.md`: change it only if the user agreed to a change in behaviour. Then update the FR and its 수용 기준 too.
- 상태 (hub only): on the feature's first task, set the hub entry to `구현중`. When every task is checked, move the entry to the 완료 section as one line (`- NNN-slug — <one line> · [spec](NNN-slug/spec.md) · 코드: <paths>`).
- hub: if files or directories changed from plan.md, update the entry's `코드:` line too.

## 5. 보고

Report in a few lines: what was implemented, the verification command and its result, any document changes, and the next available task(s). Do not continue to the next task unless the user asked for several ("T-03부터 T-05까지", "P1 끝까지"); in that case repeat steps 1-4 per task and stop at the first failure or decision point.

Do not commit unless the user asks. When they do, use one commit per task with the task ID in the message (e.g. `T-03: 로그인 API 추가`).
