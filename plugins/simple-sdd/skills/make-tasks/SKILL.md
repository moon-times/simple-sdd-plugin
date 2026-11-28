---
name: make-tasks
description: >
  plan.md의 각 Phase를 커밋 단위 작업으로 쪼개 tasks.md를 작성하는 스킬.
  This skill should be used when the `sdd` skill reaches step 4, or when the user asks
  "tasks 만들어줘", "tasks.md 작성해줘", "작업 목록으로 쪼개줘", "태스크 분해해줘" for a feature in specs/.
metadata:
  version: "0.3.0"
---

# tasks.md 작성

tasks.md lists **what to do, in what order**. Each task is small enough for one commit and one working session, and has a completion condition that can be checked.

## 입력

1. Locate the feature folder in the specs root (the folder holding `update-hub.md`; `specs/` when there is none) and read `plan.md` and `spec.md`. The 문서 모드 is **Obsidian** when `update-hub.md` starts with YAML frontmatter, otherwise **일반**. Write the document's top and links by the `sdd` skill's "문서 머리말" section (`../sdd/SKILL.md`): in Obsidian 모드 the `> 근거 | 작성일` line below becomes frontmatter.
2. If `plan.md` is missing, stop and tell the user to run `make-plan` (or `sdd`) first.
3. If `tasks.md` already exists, update it in place: keep checked tasks and existing IDs, append new tasks with new IDs.

## 템플릿

```markdown
# Tasks: <feature name>

> 근거: plan.md | 작성일: YYYY-MM-DD
> 표기: [P] = 같은 Phase 안에서 다른 [P] 작업과 병렬 가능

## P1 <phase name>

- [ ] T-01 [P] <imperative title>
  - 관련: FR-01, FR-02
  - 파일: `src/models/user.ts`, `src/models/user.test.ts`
  - 내용: <what to do, 1-3 lines>
  - 완료 조건: <verifiable check>
  - 의존: 없음
  - 넘김: `findUser(email: string): User | null`

- [ ] T-02 <title>
  - 관련: FR-03
  - 파일: ...
  - 내용: ...
  - 완료 조건: ...
  - 의존: T-01
  - 받음: `findUser(email: string): User | null` (T-01)

## P2 <phase name>
...

## 요구사항 커버리지
| 요구사항 | 작업 |
|---------|------|
| FR-01 | T-01, T-04 |
```

## 작성 규칙

- **크기**: one commit, roughly under ~300 changed lines, touching a few related files. Split anything larger. Do not split so fine that a task cannot be verified on its own.
- **완료 조건** must be checkable by running or observing something: "`npm test -- user.test.ts` 통과", "`POST /login`에 잘못된 비밀번호 → 401 응답", "빌드 경고 없음". Never "잘 동작한다" or "구현 완료".
- Prefer pairing implementation with its tests in the same task, or put the test task immediately before (test-first) when the plan's test strategy says so.
- **순서**: tasks within a Phase are listed in dependency order. A task depends only on earlier task IDs. The last task of each Phase verifies the Phase 완료 조건 from plan.md.
- **[P]** only when the task shares no files and no unmet dependency with other [P] tasks in the same Phase.
- **관련** cites FR/NFR IDs. Setup/refactor tasks with no direct requirement cite the Phase instead (관련: P1 준비).
- **넘김 / 받음**: when a later task uses something this task creates (a function, type, API, table, event), write it in this task's 넘김 and the later task's 받음 (with the source task ID), with the exact name and signature from plan.md "4. 데이터 모델 / 인터페이스". Omit both lines when a task shares nothing with others.
- **경계 케이스**: every row of plan.md's 경계 케이스 table appears in the 완료 조건 of a task in its Phase.
- Paths and commands come from plan.md and the real codebase.
- Introduce nothing that plan.md does not contain. If a needed step is missing from the plan, add it to plan.md's 변경 기록 and the relevant section first, then create the task.

## 자가 점검 (저장 전)

- [ ] 모든 FR/NFR이 커버리지 표에서 하나 이상의 작업에 연결된다
- [ ] 모든 작업에 검증 가능한 완료 조건이 있다
- [ ] 의존 관계에 순환이 없고, 앞선 ID만 참조한다
- [ ] 각 Phase 마지막 작업이 plan의 Phase 완료 조건을 확인한다
- [ ] 모든 받음에 같은 이름과 시그니처의 넘김이 앞선 작업에 있다
- [ ] plan의 모든 경계 케이스가 어떤 작업의 완료 조건에 들어 있다

## 출력

Save `tasks.md`, then report: task count per Phase, tasks that can start now (의존: 없음), and any requirement left uncovered. When invoked from `sdd`, return to it.
