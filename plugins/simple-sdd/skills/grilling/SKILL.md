---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent with a fresh context window to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

## Simple SDD handoff

Apply this section only when grilling runs as step 1 of the `sdd` skill (the `sdd` skill names the feature folder, e.g. `specs/003-user-login/`). When grilling is used on its own, ignore this section and stop as above.

Once the user confirms shared understanding, write `<feature folder>/decisions.md` in the user's language, then hand control back to the `sdd` skill. Record only what the user decided or confirmed; never add a decision the user did not make.

```markdown
# Decisions: <feature name>

## 개요
<2-3 sentences: what is being built and why>

## 결정 사항
- **D-01 <title>**: <decision>
  - 근거: <reason the user gave, or the recommendation they accepted>
- **D-02 ...**

## 범위에서 제외한 것
- <things explicitly decided NOT to do>

## 확인한 사실
- <facts found from the environment (existing code, tools, constraints), with where they came from>

## 남은 질문
- <should be empty; list anything the user chose to defer>
```
