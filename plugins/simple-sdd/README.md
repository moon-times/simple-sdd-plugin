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

문서는 `specs/NNN-기능명/` 폴더에 쌓입니다.

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
| `make-spec` | 요구사항(FR/NFR), 수용 기준, 비목표 |
| `make-plan` | 기술 선택과 근거, 파일 구조, Phase, 요구사항 매핑 |
| `make-tasks` | 커밋 단위 작업, 검증 가능한 완료 조건, 의존 관계 |
| `implement-task` | 작업 1개 구현 → 완료 조건 확인 → 체크 → plan 변경 기록 |

## ID 체계

`D-01`(결정) → `FR-01`/`NFR-01`(요구사항) → `P1`(Phase) → `T-01`(작업)

각 문서가 앞 문서의 ID를 인용하므로 빠진 요구사항이나 근거 없는 작업을 바로 찾을 수 있습니다. ID는 재번호하지 않습니다.

## 출처와 라이선스

- 이 플러그인은 MIT 라이선스입니다. 저장소 루트의 `LICENSE`를 참고하세요.
- `skills/grilling`은 [mattpocock/skills](https://github.com/mattpocock/skills)의 `skills/productivity/grilling`을 가져온 것입니다 (MIT, Copyright (c) 2026 Matt Pocock). 원본 라이선스는 `skills/grilling/LICENSE`에 있습니다.
- 원본에서 바꾼 점: Simple SDD 흐름에서 쓰일 때 결정 사항을 `decisions.md`로 저장하는 "Simple SDD handoff" 섹션을 끝에 추가했습니다. 나머지 내용은 원본과 같습니다.
