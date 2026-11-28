# Simple SDD

grilling으로 설계 결정을 확정한 뒤 spec → plan → tasks 문서를 차례로 만들고, task 하나씩 구현하는 Spec-Driven Development 플러그인.

## 흐름

```
grilling ──> decisions.md
   │ (확인)
make-spec ──> spec.md      무엇을, 왜
   │ (자동 진행)
make-plan ──> plan.md      어떻게
   │ (확인)
make-tasks ─> tasks.md     무엇을 어떤 순서로
   │
implement-task  T-01, T-02 ... 하나씩 구현 + 문서 동기화
```

문서는 `specs/NNN-기능명/` 폴더에 쌓이고, `specs/update-hub.md`가 모든 기능의 설명·상태·코드 경로·spec 링크를 한 곳에 모읍니다.

## update-hub와 CLAUDE.md

- `update-hub.md`는 목록일 뿐, 요구사항을 옮겨 적지 않습니다. 진행 중 기능은 3줄 안팎, 완료된 기능은 1줄입니다. 스킬이 단계마다 직접 갱신합니다. 기능의 상태(설계중·구현중·완료)는 hub에만 적고 문서에는 적지 않습니다. 링크는 spec 하나만 걸고, 나머지 문서는 같은 폴더에 정해진 이름으로 있습니다.
- 처음 실행할 때 프로젝트 `CLAUDE.md`에 "명세 읽는 순서" 섹션을 추가합니다. hub를 먼저 읽고 관련 기능의 문서에서 필요한 ID(FR-03, D-02)만 찾아 읽게 해서, 매번 명세 전체를 읽지 않습니다.
- v0.1로 만든 `specs/`가 있으면 처음 실행할 때 기존 폴더로 hub를 만듭니다.

## Obsidian 모드

PC에 Obsidian이 설치돼 있으면(`obsidian.json`이 있으면) 자동으로 켜집니다.

| | 일반 모드 | Obsidian 모드 |
|--|----------|--------------|
| 위치 | `specs/` | 프로젝트 안 vault가 있으면 `<vault>/specs/`, 없으면 `docs/specs/` |
| 머리말 | `> 근거 \| 작성일` 한 줄 | frontmatter (`type`, `feature`, `tags`) |
| 미결 질문 | `[확인 필요]` 목록 | `> [!question]` 콜아웃 |
| 링크 | 상대 경로 마크다운 링크 | 같음 (기능마다 `spec.md`가 있어 위키링크는 이름이 겹침) |

한 번 정해진 모드는 hub 첫 줄(frontmatter 여부)로 유지됩니다. 플러그인은 `obsidian.json`이나 `.obsidian/`을 건드리지 않습니다.

## 사용법

| 하고 싶은 것 | 이렇게 말하기 |
|-------------|--------------|
| 처음부터 설계 | "로그인 기능 SDD로 설계해줘" 또는 `/simple-sdd:sdd` |
| 확인 없이 끝까지 | "SDD 자동으로 끝까지 해줘" (grilling 확인과 미결 질문에서는 여전히 멈춤) |
| 중단한 곳부터 | "003-user-login SDD 이어서 해줘" |
| 단계 하나만 | "plan 다시 만들어줘", "tasks만 만들어줘" |
| 구현 | "T-03 구현해줘", "다음 task 진행해줘", "P1 끝까지 구현해줘" |

## 스킬 구성

| 스킬 | 역할 |
|-----|------|
| `sdd` | 전체 순서 진행, 폴더 생성, 체크포인트, 이어하기 |
| `grilling` | 질문 라운드로 결정 확정 (원본 + decisions.md 저장 규칙). 단독으로 쓰면 원래대로 동작 |
| `make-spec` | 요구사항(FR/NFR), 수용 기준, 비목표. 저장 전 새 서브에이전트가 spec만 읽고 빈틈을 질문(빌더 점검) |
| `make-plan` | 기술 선택과 근거, 파일 구조, Phase, 요구사항 매핑, spec이 다루지 않은 경계 케이스 |
| `make-tasks` | 커밋 단위 작업, 검증 가능한 완료 조건, 의존 관계, 작업 사이에 넘기고 받는 인터페이스 |
| `implement-task` | 작업 1개 구현 → 완료 조건 확인 → 체크 → plan 변경 기록. Phase 마지막 작업에서는 서브에이전트가 `code-review`와 수용 기준 점검을 한 번 실행 |

## v0.3 변경 사항

[superpowers](https://github.com/obra/superpowers)의 아이디어 몇 가지를 simple-sdd 방식(추측하지 않고 사용자에게 묻기)에 맞게 가져왔습니다.

- **빌더 점검** (`make-spec`): 대화를 모르는 서브에이전트가 spec만 읽고 "이대로 만들 수 있나"를 묻습니다. 결정에 답이 있으면 spec에 반영하고, 없으면 `[확인 필요]`로 넘깁니다.
- **경계 케이스** (`make-plan`): 수용 기준이 다루지 않는 입력과 상황을 최대 5개까지 적고, 기대 동작은 근거가 있을 때만 씁니다. 근거가 없으면 체크포인트에서 묻습니다. 각 경계 케이스는 어떤 작업의 완료 조건에 들어갑니다.
- **넘김 / 받음** (`make-tasks`): 다른 작업이 쓰는 함수, 타입, API를 정확한 시그니처로 적습니다.
- **Phase 리뷰** (`implement-task`): Phase가 끝나면 서브에이전트가 Claude Code 내장 `code-review` 스킬과 수용 기준 점검을 실행합니다. 중요한 문제는 같은 Phase에 새 작업으로 추가하고, `tasks.md`에 `> 리뷰:` 줄을 남겨 다시 리뷰하지 않게 합니다.
- **원인 먼저, 증거로 보고** (`implement-task`): 실패하면 고치기 전에 원인을 찾고, 보고에는 실제 실행 결과(예: `34/34 통과`)를 적습니다.
- **브랜치 / 커밋 / push**: 기능 하나를 브랜치 하나에서 진행합니다.

  | 시점 | 동작 |
  |------|------|
  | 새 기능 시작 | `feat/NNN-기능명` 브랜치 생성 |
  | 설계 완료 | 설계 문서 커밋 |
  | task 완료 | task 하나당 커밋 `T-03: 작업 제목` |
  | Phase 완료 (리뷰 통과) | push |

  Phase 리뷰는 마지막 push 지점부터의 변경만 봅니다. git 저장소가 아니면 이 규칙은 건너뜁니다. 브랜치 병합과 PR은 사용자가 직접 합니다. force push는 하지 않습니다.

## ID 체계

`D-01`(결정) → `FR-01`/`NFR-01`(요구사항) → `P1`(Phase) → `T-01`(작업)

각 문서가 앞 문서의 ID를 인용하므로 빠진 요구사항이나 근거 없는 작업을 바로 찾을 수 있습니다. ID는 재번호하지 않습니다.

## 출처와 라이선스

- 이 플러그인은 MIT 라이선스입니다. 저장소 루트의 `LICENSE`를 참고하세요.
- `skills/grilling`은 [mattpocock/skills](https://github.com/mattpocock/skills)의 `skills/productivity/grilling`을 가져온 것입니다 (MIT, Copyright (c) 2026 Matt Pocock). 원본 라이선스는 `skills/grilling/LICENSE`에 있습니다.
- 원본에서 바꾼 점: Simple SDD 흐름에서 쓰일 때 결정 사항을 `decisions.md`로 저장하는 "Simple SDD handoff" 섹션을 끝에 추가했습니다. 나머지 내용은 원본과 같습니다.
