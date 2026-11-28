---
name: make-spec
description: >
  decisions.md(또는 정리된 요구사항)를 바탕으로 기능의 spec.md(무엇을, 왜)를 작성하는 스킬.
  This skill should be used when the `sdd` skill reaches step 2, or when the user asks
  "spec 만들어줘", "spec.md 작성해줘", "요구사항 문서 만들어줘", "명세서 써줘" for a feature in specs/.
metadata:
  version: "0.3.0"
---

# spec.md 작성

spec.md answers **what** to build and **why**. It contains no technology, library, file, or implementation choices; those belong in plan.md.

## 입력

1. Locate the feature folder in the specs root (the folder holding `update-hub.md`; `specs/` when there is none). If none is given and several exist, ask which one. The 문서 모드 is **Obsidian** when `update-hub.md` starts with YAML frontmatter, otherwise **일반**. Write the document's top and links by the `sdd` skill's "문서 머리말" section (`../sdd/SKILL.md`): in Obsidian 모드 the `> 근거 | 작성일` line below becomes frontmatter.
2. Read `decisions.md`. If it does not exist:
   - When the user supplied requirements in the conversation, offer to run `grilling` first (recommended). If they decline, write `spec.md` from what they supplied and mark every gap `[확인 필요]`.
3. If `spec.md` already exists, update it in place instead of rewriting; preserve existing IDs.

## 템플릿

```markdown
# Spec: <feature name>

> 근거: decisions.md | 작성일: YYYY-MM-DD

## 1. 배경 / 문제
<why this is needed, whose problem it is>

## 2. 목표
- G-01 <measurable outcome where possible>

## 3. 비목표
- <what this work deliberately does not do>

## 4. 사용자 시나리오
- **S-01 <title>**: <who> 가 <situation> 에서 <action> 하면 <result>

## 5. 기능 요구사항
| ID | 요구사항 | 근거 |
|----|---------|------|
| FR-01 | <one behaviour per line, "~해야 한다"> | D-02 |

## 6. 비기능 요구사항
| ID | 요구사항 | 근거 |
|----|---------|------|
| NFR-01 | <performance, security, compatibility, accessibility...> | D-05 |

## 7. 수용 기준
### FR-01
- Given <precondition> / When <action> / Then <observable result>
- (error or edge case) Given ... / When ... / Then ...

## 8. 제약 / 가정
- 제약: <already fixed, not negotiable>
- 가정: <assumed true; each must trace to a decision or confirmed fact>

## 9. 미결 사항
- [확인 필요] <question> (관련: FR-03)   ← Obsidian 모드: `> [!question] [확인 필요] <question> (관련: FR-03)`
```

## 작성 규칙

- Every FR and NFR cites the decision (`D-xx`) or confirmed fact it comes from. A requirement with no source is a guess: move it to 미결 사항 as `[확인 필요]`.
- One behaviour per FR. Split "A하고 B한다" into two FRs.
- Every FR has at least one Given/When/Then, and FRs with failure paths include the failure case. Then-clauses describe something observable (a screen, a response, a stored value), not internal state.
- 비목표 is mandatory. Pull it from decisions.md "범위에서 제외한 것"; if empty, ask the user for at least one boundary.
- No technology names unless the user fixed them as a constraint (then they go in 제약, not in FRs).
- IDs are stable: never renumber; mark removed items `~~FR-04~~ (삭제: <reason>)`.

## 자가 점검 (저장 전)

- [ ] 모든 FR/NFR에 근거가 있다
- [ ] 모든 FR에 수용 기준이 있다
- [ ] 비목표가 비어 있지 않다
- [ ] 구현 방법(기술, 파일, 라이브러리)이 섞이지 않았다
- [ ] decisions.md의 모든 결정이 spec 어딘가에 반영되었다 (반영할 필요 없는 결정은 제외 이유를 한 줄로)

## 빌더 점검 (자가 점검 후)

Your own check cannot see gaps you filled from the conversation. Dispatch a fresh sub-agent with this prompt, giving it only the two file paths:

> You will build this feature from `<spec.md path>` alone; `<decisions.md path>` holds the decisions behind it. Read both. List every question whose answer would change **what** gets built: behaviour the spec leaves unsaid, scenarios with no acceptance criterion, requirements that contradict each other or the decisions. Skip questions about technology or code structure; those are decided later. For each question, quote the line it comes from. Return only the numbered list, or "질문 없음".

Sort what comes back:
- **Answered by decisions.md or a confirmed fact**: write the answer into the spec, citing its source.
- **About how, not what**: drop it (plan.md decides it).
- **Everything else**: add it to 미결 사항 as `[확인 필요]`. Never settle it yourself.

Without a sub-agent tool, read the spec once as that builder yourself.

## 출력

Save `spec.md` and, if the hub exists, add the spec link to the feature's entry. Then report: FR/NFR count, and the 미결 사항 list as numbered questions with a recommended answer each. When there are none, say so in one line. When invoked from `sdd`, return to it.
