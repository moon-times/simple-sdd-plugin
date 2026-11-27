# Simple SDD

Claude Code용 Spec-Driven Development 플러그인입니다. 질문 라운드(grilling)로 설계 결정을 확정한 뒤 spec → plan → tasks 문서를 차례로 만들고, task를 하나씩 구현합니다.

## 설치

Claude Code 안에서 아래 두 줄을 입력하세요.

```
/plugin marketplace add moon-times/simple-ssd-plugin
/plugin install simple-sdd@moon-times
```

설치 범위를 묻는 화면이 나오면 기본값으로 진행하면 됩니다.

### 업데이트

```
/plugin marketplace update moon-times
```

## 사용법

| 하고 싶은 것 | 이렇게 말하기 |
|-------------|--------------|
| 처음부터 설계 | "인벤토리 정렬 기능 SDD로 설계해줘" 또는 `/simple-sdd:sdd` |
| 확인 없이 끝까지 | "SDD 자동으로 끝까지 해줘" |
| 중단한 곳부터 | "001-inventory-sort SDD 이어서 해줘" |
| 구현 | "T-01 구현해줘", "다음 task 진행해줘" |

문서는 프로젝트 안 `specs/NNN-기능명/` 폴더에 `decisions.md`, `spec.md`, `plan.md`, `tasks.md`로 쌓입니다. `specs/update-hub.md`가 모든 기능의 설명·상태·코드 경로를 한곳에 모으고, 처음 실행할 때 프로젝트 `CLAUDE.md`에 "hub부터 읽고 관련 명세만 읽기" 규칙을 추가합니다.

PC에 Obsidian이 설치돼 있으면 Obsidian 모드로 만듭니다. 프로젝트 안에 vault가 있으면 그 안의 `specs/`에, 없으면 `docs/specs/`에 frontmatter를 붙인 문서를 만듭니다.

### 팁

- 질문마다 추천 답이 붙어 나옵니다. 잘 모르겠으면 "추천대로"라고 답하면 됩니다.
- 기술 선택을 확인하는 단계에서도 판단이 어려우면 "추천대로 해줘"로 충분합니다.
- 오타 수정, 수치 조정 같은 작은 변경은 SDD 없이 그냥 요청하는 게 빠릅니다. 새 기능을 추가할 때 쓰세요.

자세한 스킬 구성은 [plugins/simple-sdd/README.md](plugins/simple-sdd/README.md)를 참고하세요.

## 출처와 라이선스

- MIT 라이선스 ([LICENSE](LICENSE))
- `grilling` 스킬은 [mattpocock/skills](https://github.com/mattpocock/skills)에서 가져와 수정했습니다 (MIT, Copyright (c) 2026 Matt Pocock). 원본 라이선스: [plugins/simple-sdd/skills/grilling/LICENSE](plugins/simple-sdd/skills/grilling/LICENSE)
