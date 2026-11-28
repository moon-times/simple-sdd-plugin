---
name: implement-task
description: >
  tasks.md의 작업 하나를 구현하고, 완료 조건을 확인한 뒤 체크하고, 계획과 달라진 점을 문서에 반영하는 스킬.
  This skill should be used when the user says "T-03 구현해줘", "다음 task 진행해줘", "다음 작업 해줘",
  "tasks.md 이어서 구현", or otherwise asks to implement a task from a specs/ feature folder.
metadata:
  version: "0.3.0"
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
- Follow plan.md's 기술 선택, interfaces, and the codebase's existing conventions. Use the names and signatures in the task's 받음 / 넘김 lines exactly; changing a 넘김 affects the tasks that 받음 it, so it needs the user's decision below.
- If the task cannot be done as planned (wrong assumption, missing API, better approach found), stop before diverging. Explain the problem in a few lines, propose the change, and wait for the user's decision when it affects the interface, data model, dependencies, or other tasks. Minor internal choices do not need to wait.

## 3. 검증

- Run the check in 완료 조건 exactly (tests, commands, requests). Also run the project's lint/type check if plan.md lists one.
- If the check fails, find the cause before changing code: read the whole error and stack trace, reproduce the failure, and look at what changed since it last passed. Fix the cause, not the symptom, then re-run. After two failed fix attempts, stop and report what fails and why.
- Check the box only when the 완료 조건 actually passed in this session, with the output read. Never check a box based on "should work", an earlier run, or a sub-agent's word.

## 3-1. Phase 리뷰 (Phase의 마지막 작업일 때)

When this task leaves no unchecked task in its Phase and the Phase has no `> 리뷰:` line in `tasks.md` yet, review the Phase before reporting.

1. Find the Phase's change. The base is the last pushed point, `@{upstream}`, which is the end of the previous Phase. When the branch has no upstream yet (Phase 1), the base is the parent of the earliest commit whose message starts with one of this Phase's task IDs (`HEAD` if none is committed). The change is `git diff <base>` (working tree included, since this task is not committed yet).
2. Dispatch a fresh sub-agent with this prompt:

   > Review one Phase of a feature. Base commit: `<base>`. Spec: `<spec.md path>`; this Phase covers `<FR/NFR IDs>`. Plan's edge cases for this Phase: `<rows from plan.md 경계 케이스>`.
   > 1. Run the `code-review` skill at `medium` effort on the changes since the base commit. If it cannot take that target or is not available, review `git diff <base>` yourself for correctness bugs.
   > 2. For each listed FR/NFR, check its 수용 기준 in the spec against the code, and check each edge case.
   > Return a numbered list of findings, each with file:line, what breaks for a user, and severity (중요 = wrong behaviour, data loss, security; 사소 = everything else). Or "발견 없음".

3. Handle the findings yourself; do not fix them inside this task:
   - **중요**: append each as a new task at the end of the Phase (new ID, 관련: the FR or `P<n> 리뷰`, 완료 조건 that reproduces the finding).
   - **사소**: list in the report only.
4. Add one line under the Phase heading in `tasks.md`: `> 리뷰: YYYY-MM-DD · 중요 N건 → T-12, T-13 · 사소 M건` (or `발견 없음`). This line means the Phase is reviewed; fix tasks added by it do not trigger another review.

## 4. 문서 반영

- `tasks.md`: change `- [ ]` to `- [x]` for the task. If new follow-up work was discovered, append it as a new task with a new ID (never renumber).
- `plan.md`: if the implementation differs from the plan (file names, interfaces, choices), update the affected section and add a line to 9. 변경 기록: `YYYY-MM-DD T-03: <what changed and why>`.
- `spec.md`: change it only if the user agreed to a change in behaviour. Then update the FR and its 수용 기준 too.
- 상태 (hub only): on the feature's first task, set the hub entry to `구현중`. When every task is checked, move the entry to the 완료 section as one line (`- NNN-slug — <one line> · [spec](NNN-slug/spec.md) · 코드: <paths>`).
- hub: if files or directories changed from plan.md, update the entry's `코드:` line too.

## 5. 커밋 / push

Follow the `sdd` skill's "브랜치 / 커밋 / push" section (`../sdd/SKILL.md`), including its rules for no git, no remote, and failed pushes.

- Work on the feature's branch `feat/NNN-<feature-slug>`. If another branch is checked out, switch to it first; if it does not exist, create it.
- Commit once the 완료 조건 passed and step 4 is done: the task's code plus its document changes, message `T-03: <task title>`. A task whose 완료 조건 failed is not committed.
- Push when the Phase is complete: every task in it is checked and it has a `> 리뷰:` line. If the review added fix tasks, the Phase is complete only after the last of them, so push then.

## 6. 보고

Report in a few lines: what was implemented, the verification command and its actual result (e.g. `npm test → 34/34 통과`, never "통과할 것" or "문제 없어 보임"), any document changes, the Phase 리뷰 findings if one ran, the commit (hash and message) and whether it was pushed, and the next available task(s). Do not continue to the next task unless the user asked for several ("T-03부터 T-05까지", "P1 끝까지"); in that case repeat steps 1-5 per task and stop at the first failure, decision point, or Phase 리뷰 with 중요 findings.
