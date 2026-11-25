# agent-skills

자주 쓰는 Claude Code 스킬을 직접 만들어 모아 두는 저장소입니다.
여기서 만든 스킬은 `skill-install`로 `~/.claude/skills`에 심볼릭 링크로 설치해 모든 프로젝트에서 씁니다.

| 항목 | 값 |
| --- | --- |
| GitHub | https://github.com/stdhsw/agent-skills |
| 원격(origin) | `git@github.com:stdhsw/agent-skills.git` |
| 기본 브랜치 | `main` |

## 저장소 구조

```
agent-skills/
├── <스킬 이름>/                  # 스킬 하나 = 루트의 폴더 하나
│   ├── SKILL.md                  # frontmatter(name, description) + 작업 흐름
│   ├── references/<기능>.md      # 기능별 상세 설명 (필요할 때만 읽힘)
│   └── assets/<기능>.*           # 설치·복사할 템플릿 파일
├── .claude/skills/skill-install  # → ../../skill-install (이 저장소 전용 상대 링크)
├── SKILL_IDEAS.md                # 만든 스킬 현황과 다음 스킬 후보
└── CLAUDE.md
```

| 스킬 | 역할 |
| --- | --- |
| `git-commit` | Conventional Commits 형식의 한국어 제목 + 한국어 Why/What 본문 + git 트레일러로 커밋 |
| `git-pr` | 한국어 PR 생성, 커밋 분석 기반 머지 방식 추천과 머지, gh 설치·로그인 안내 |
| `skill-install` | 이 저장소의 스킬을 `~/.claude/skills`에 링크로 설치·제거, 상태 확인 |
| `vscode-setup` | 프로젝트 `.vscode/`에 마크다운 색상(편집 화면 + Preview)과 편집 설정 적용 |
| `vscode-help` | `cmd+alt+h`로 여는 개인 도움말(`~/.config/vscode/HELP.md`) 설치와 내용 관리 |

## 스킬 작성 규칙

- **위치와 이름**: 스킬은 저장소 루트 바로 아래 폴더로 만듭니다. 이름은 `<도구>-<동작>` 형식의 소문자 하이픈 표기입니다(`git-commit`, `vscode-setup`). 폴더 이름과 frontmatter의 `name`이 같아야 `skill-install` 검사를 통과합니다.
- **언어**: `SKILL.md`, references, 템플릿 모두 한국어로 씁니다. 명령어, 설정 키, 커밋의 type·scope처럼 원래 영어인 것만 영어로 둡니다.
- **description**: 무엇을 하는지와 트리거 예시 문구를 함께 적습니다. 비슷한 스킬이 있으면 역할 구분을 적습니다(예: "프로젝트 설정은 vscode-setup이, 개인 도움말은 vscode-help가 담당"). 스킬이 덜 실행되는 경향이 있으므로 트리거 조건은 넉넉하게 씁니다.
- **구조**: 기능이 여러 개인 스킬은 `SKILL.md`에 공통 흐름만 두고, 기능별 내용은 `references/`, 템플릿은 `assets/`에 둡니다.
- **본문 문체**: 대문자 MUST 같은 강한 지시보다 "왜 그렇게 해야 하는지"를 설명합니다.
- **설정 파일 수정 방식**: 스킬이 사용자 설정 파일(settings.json 등 JSONC)을 고칠 때는 덮어쓰지 않고 Edit 도구로 병합하도록 지시합니다. 스크립트로 JSON을 다시 쓰면 주석이 사라지고 검토도 어렵기 때문입니다. 기존 값과 충돌하면 사용자에게 확인하도록 합니다.

## 템플릿과 문서를 함께 고치기

같은 값이 여러 파일에 있는 경우가 있습니다. 한쪽만 고치면 스킬 결과와 설명이 어긋나므로 함께 고칩니다.

- `vscode-setup` 마크다운 팔레트(현재 도쿄 나이트):
  - `assets/markdown.json`: 편집 화면
  - `assets/markdown-preview.css`: Preview
  - `references/markdown.md`: 색상 표
- 이 저장소의 `.vscode/settings.json`, `.vscode/markdown-preview.css`는 `vscode-setup` 템플릿을 적용한 결과입니다. 팔레트를 바꾸면 이쪽도 맞춥니다. (`.vscode/`는 gitignore 대상이라 커밋되지 않습니다.)
- 스킬을 추가하거나 상태가 바뀌면 `SKILL_IDEAS.md`의 "만든 스킬" 표와 후보 목록을 갱신합니다.

## 설치와 반영

- 스킬 설치·상태 확인은 `skill-install` 스킬로 합니다. 설치는 링크라서 저장소에서 스킬을 고치면 바로 반영됩니다. 새로 추가한 스킬만 설치가 필요합니다.
- 새 스킬이나 크게 바뀐 description은 새로 시작한 세션부터 확실히 반영됩니다.
- `~/.claude/skills`는 별도의 git 저장소이고, 이 저장소와 관계없는 스킬(`nvim-helper`, `go-*`)과 Claude 앱이 관리하는 `synced/`가 함께 있습니다. 이 항목들은 건드리지 않고, 그 저장소에 커밋하지도 않습니다.

## 테스트

- skill-creator로 테스트하면 `<스킬>-workspace/` 폴더가 저장소 루트에 생깁니다. 이 폴더는 gitignore 대상이 아니므로 확인이 끝나면 지웁니다.
- skill-creator의 뷰어 스크립트는 Python 3.10 이상 문법을 씁니다. 이 PC의 기본 `python3`은 3.9라서 그대로는 실행되지 않습니다.
- 뷰어는 숨김 폴더(`.vscode/` 등)의 파일을 보여 주지 않습니다. 확인할 결과 파일이 숨김 폴더에 있으면 보이는 위치에 복사해서 봅니다.

## Git

- 커밋은 `git-commit` 스킬 형식을 따릅니다. 커밋 메시지는 한국어로 씁니다. 제목은 `type(scope): 한국어 설명` 형식이고 type과 scope만 영어입니다. 본문은 한국어 Why/What입니다. 미룬 작업은 `Follow-up` 트레일러로 남깁니다.
- scope는 스킬 이름을 씁니다(`feat(vscode-help): ...`). 저장소 전체에 걸친 변경이면 scope를 생략합니다.
- 기능(스킬)별로 커밋을 나눕니다.
- `main`에 바로 커밋하지 않고 작업 브랜치에서 커밋합니다. PR과 머지는 `git-pr` 스킬로 합니다. PR 제목과 머지 커밋 메시지도 커밋과 같은 기준(`type(scope): 한국어 설명`)으로 씁니다.
