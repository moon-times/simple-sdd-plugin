---
name: make-plan
description: >
  확정된 spec.md를 바탕으로 기술 설계와 단계(Phase) 계획인 plan.md(어떻게)를 작성하는 스킬.
  This skill should be used when the `sdd` skill reaches step 3, or when the user asks
  "plan 만들어줘", "plan.md 작성해줘", "구현 계획 세워줘", "설계 문서 써줘" for a feature in specs/.
metadata:
  version: "0.2.0"
---

# plan.md 작성

plan.md answers **how** to build what spec.md defines. It adds no new features.

## 입력

1. Locate the feature folder in the specs root (the folder holding `update-hub.md`; `specs/` when there is none) and read `spec.md` and `decisions.md`. The 문서 모드 is **Obsidian** when `update-hub.md` starts with YAML frontmatter, otherwise **일반**. Write the document's top and links by the `sdd` skill's "문서 머리말" section (`../sdd/SKILL.md`): in Obsidian 모드 the `> 근거 | 작성일` line below becomes frontmatter.
2. If `spec.md` is missing, stop and tell the user to run `make-spec` (or `sdd`) first.
3. If `spec.md` still has `[확인 필요]` items, ask them first (numbered, with a recommended answer each) and update spec.md. Do not plan around open questions.
4. Explore the existing codebase before choosing anything: directory layout, language and framework versions, existing patterns for similar features, test setup, lint/build commands. Use a sub-agent for wide searches. Prefer the project's existing conventions over new ones; a new dependency needs a reason in 기술 선택.

## 템플릿

```markdown
# Plan: <feature name>

> 근거: spec.md | 작성일: YYYY-MM-DD

## 1. 아키텍처 개요
<components and data flow; a mermaid diagram if it helps>

## 2. 기술 선택
| 항목 | 선택 | 이유 | 검토한 대안 |
|------|------|------|------------|
| <e.g. 상태 관리> | <choice> | <reason, ideally tied to FR/NFR/제약> | <alternative: why not> |

## 3. 파일 / 모듈 구조
- 신규: `path/to/new_file` — <role>
- 수정: `path/to/existing` — <what changes>

## 4. 데이터 모델 / 인터페이스
<schemas, API contracts (method, path, request, response, errors), key function signatures>

## 5. 단계(Phase)
### P1 <name>
- 목표: <what works at the end of this phase>
- 포함 요구사항: FR-01, FR-02, NFR-01
- 완료 조건: <verifiable: tests, commands, observable behaviour>

### P2 ...

## 6. 요구사항 매핑
| 요구사항 | Phase | 비고 |
|---------|-------|------|
| FR-01 | P1 | |

## 7. 테스트 전략
- 단위: <what>  / 통합: <what> / 수동 확인: <what>
- 실행 명령: `<test command found in the project>`

## 8. 리스크
| 리스크 | 영향 | 대응 |
|-------|------|------|

## 9. 변경 기록
- YYYY-MM-DD: 초안 작성
```

## 작성 규칙

- Do not add features, endpoints, or screens that no FR/NFR requires. If something seems missing from the spec, raise it as a question instead of adding it.
- Each Phase ends in a working, verifiable state. Order Phases so each builds on the previous one (usually data → core logic → interface → polish).
- Keep Phases to 2-5 for a typical feature. Too many Phases usually means tasks are leaking in.
- Every FR and NFR appears in the 요구사항 매핑 table exactly once as primary owner. An NFR that spans phases goes to the phase where it is verified.
- Choices the user should weigh (new dependency, data model shape, public API shape, anything costly to reverse) are flagged in 기술 선택 so the checkpoint can surface them.
- Use real paths and commands found in the codebase, not placeholders.

## 자가 점검 (저장 전)

- [ ] 모든 FR/NFR이 매핑 표에 있다
- [ ] spec에 없는 기능이 추가되지 않았다
- [ ] 모든 기술 선택에 이유와 대안이 있다
- [ ] 모든 Phase에 검증 가능한 완료 조건이 있다
- [ ] 파일 경로와 명령어가 실제 프로젝트 기준이다

## 출력

Save `plan.md`. If the hub exists, set the feature's `코드:` line from 3. 파일 / 모듈 구조 (directories or globs, not every file). Then report: the 기술 선택 table, the Phase list (one line each), and any FR/NFR that could not be mapped. When invoked from `sdd`, return to it for checkpoint C.
