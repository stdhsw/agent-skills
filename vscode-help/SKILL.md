---
name: vscode-help
description: >
  VS Code에서 어느 프로젝트를 열어도 단축키 하나(cmd+alt+h)로 여는 나만의 도움말(~/.config/vscode/HELP.md)을
  설치하고 내용을 관리하는 스킬. 설치 시 도움말 파일, 사용자 tasks.json의 "Help Guide" 작업,
  keybindings.json 단축키, settings.json의 Preview 연결(workbench.editorAssociations)을 한 번에 설정한다.
  설치 후에는 VS Code 단축키, Claude Code 명령어 등을 도움말의 알맞은 섹션에 정해진 표기 규칙으로 추가·수정·정리한다.
  사용자가 "vscode 도움말 만들어줘", "나만의 도움말 설치", "help guide 설정", "cmd+alt+h 도움말",
  "도움말에 추가해줘", "이 단축키 도움말에 적어줘", "도움말 정리해줘", "HELP.md 수정",
  "vscode 치트시트 만들어줘", "단축키 메모해 둘 곳 만들어줘" 등을 요청할 때 이 스킬을 사용하세요.
  "도움말"이라는 단어가 없어도 VS Code에서 공통으로 볼 개인 메모나 치트시트를 만들거나 고치는 맥락이면 적극적으로 트리거하세요.
  단, 프로젝트 단위(.vscode/) 설정은 vscode-setup 스킬이, Neovim 사용법은 nvim-helper 스킬이 담당한다.
---

# VS Code Help

어느 프로젝트에서든 `cmd+alt+h`로 같은 도움말 파일을 Preview로 여는 환경을 만들고, 그 도움말의 내용을 관리하는 스킬이다.

이 스킬은 두 가지 작업을 한다.

| 작업 | 언제 | 절차 |
| --- | --- | --- |
| **설치** | 도움말 환경이 아직 없을 때, 또는 "도움말 만들어줘" | 아래 "설치" |
| **내용 관리** | "도움말에 추가해줘", "도움말 정리해줘" | 아래 "내용 관리" |

요청이 내용 추가인데 도움말 파일이 없으면 설치를 먼저 한다.

## 구성 요소

| 항목 | 값 |
| --- | --- |
| 도움말 파일 | `~/.config/vscode/HELP.md` |
| 작업 이름 | `Help Guide` |
| 단축키 | `cmd+alt+h` |
| 여는 방식 | `code -r`로 현재 창에서 열고, Preview로 표시 |

## VS Code 사용자 설정 폴더

`tasks.json`, `keybindings.json`, `settings.json`은 모두 VS Code 사용자 설정 폴더에 있다. 사용자 설정이라 모든 프로젝트에 적용된다.

| OS | 경로 |
| --- | --- |
| macOS | `~/Library/Application Support/Code/User/` |
| Linux | `~/.config/Code/User/` |
| Windows | `%APPDATA%\Code\User\` |

VS Code Insiders를 쓰면 폴더 이름이 `Code` 대신 `Code - Insiders`다. 두 폴더가 모두 있으면 어느 쪽에 설치할지 사용자에게 묻는다.

세 파일은 JSONC(주석과 끝 쉼표 허용)다. 일반 JSON 파서로 읽고 다시 쓰면 주석이 사라지므로 스크립트로 변환하지 말고 Read로 읽은 뒤 Edit 도구로 필요한 부분만 고친다. 파일이 없으면 Write로 새로 만든다.

---

## 설치

### 1. 사전 확인

다음을 병렬로 확인한다.

```bash
command -v code                          # code 명령 설치 여부
ls ~/.config/vscode/HELP.md              # 도움말 파일 존재 여부
ls "$HOME/Library/Application Support/Code/User/"   # macOS 기준 사용자 설정 폴더
```

`code` 명령이 없으면 작업이 동작하지 않는다. 사용자에게 VS Code 명령 팔레트(`Cmd+Shift+P`)에서 `Shell Command: Install 'code' command in PATH`를 실행해 달라고 안내한다. 시스템 PATH를 바꾸는 일이라 직접 하지 않는다. 사용자가 원하지 않으면 작업의 `command`에 `code` 대신 전체 경로(macOS: `/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code`)를 쓴다.

### 2. 도움말 파일 만들기

- `~/.config/vscode/HELP.md`가 없으면 `mkdir -p ~/.config/vscode`로 폴더를 만들고 `assets/HELP.md` 템플릿을 복사한다.
- 이미 있으면 **건드리지 않는다**. 사용자가 쌓아 온 내용이기 때문이다.

### 3. 사용자 tasks.json에 작업 등록

사용자 설정 폴더의 `tasks.json`에 다음 작업을 넣는다.

```json
{
  "label": "Help Guide",
  "type": "shell",
  "command": "code -r ~/.config/vscode/HELP.md",
  "presentation": { "reveal": "never" },
  "problemMatcher": []
}
```

- 파일이 없으면 `{ "version": "2.0.0", "tasks": [ 위 작업 ] }`으로 새로 만든다.
- 파일이 있으면 `tasks` 배열 끝에 추가한다.
- 같은 `label`의 작업이 이미 있으면 `command`만 위 값과 같은지 확인한다. 다르면 바꾸기 전에 사용자에게 묻는다.

`code -r`은 새 창을 띄우지 않고 지금 창에서 파일을 연다. `"reveal": "never"`는 작업 실행 시 터미널 패널이 튀어나오지 않게 한다.

### 4. keybindings.json에 단축키 등록

```json
{
  "key": "cmd+alt+h",
  "command": "workbench.action.tasks.runTask",
  "args": "Help Guide"
}
```

- 파일이 없으면 `[ 위 항목 ]`으로 새로 만든다.
- 파일이 있으면 배열 끝에 추가한다.
- 같은 `key`에 다른 명령이 이미 있으면 덮어쓰기 전에 사용자에게 묻는다. 다른 키를 원하면 그 키로 등록한다.
- Linux와 Windows에서는 `cmd` 대신 `ctrl`을 쓴다(`ctrl+alt+h`).

### 5. settings.json에 Preview 연결

```json
"workbench.editorAssociations": {
  "**/.config/vscode/HELP.md": "vscode.markdown.preview.editor"
}
```

- `workbench.editorAssociations`가 없으면 위 키를 추가한다.
- 이미 있으면 그 객체 안에 패턴 한 줄만 추가한다. 다른 연결은 그대로 둔다.

패턴은 경로 구분자를 포함하므로 파일의 전체 경로와 비교된다. 도움말 파일 하나에만 적용되도록 파일명까지 지정했다. 다른 `.md` 파일은 평소처럼 편집 화면으로 열린다.

### 6. 결과 보고

- 만들거나 수정한 파일 경로 4개
- 기존 설정과 충돌해서 사용자에게 확인한 항목
- 사용 방법: 아무 프로젝트에서 `cmd+alt+h`를 누르면 도움말이 Preview로 열린다.
- 바로 동작하지 않으면 `Developer: Reload Window`를 실행한다.

---

## 내용 관리

도움말은 사용자가 VS Code를 쓰다가 "이거 어떻게 했더라?"를 빠르게 찾는 용도다. 짧고 찾기 쉬운 것이 가장 중요하다.

### 문서 구조

템플릿(`assets/HELP.md`)과 같은 구조를 유지한다.

```
# ⌨️ 단축키·명령어 가이드
> **표기 규칙** (인용문 안의 표)

---                      ← 대분류 앞마다 구분선

## 🤖 대분류              ← 도구 단위: Claude Code 명령어, VSCode 단축키 등
### 💬 소분류             ← 용도 단위: 멀티 커서, 터미널, 디버깅 등
> 💡 팁 (필요할 때만)
| 단축키 | 설명 |         ← 명령어 섹션은 | 명령어 | 설명 |
| --- | --- |
```

- 대분류는 **도구**로, 소분류는 **하려는 일**로 나눈다. 사용자는 "터미널에서 뭐였지?"처럼 하려는 일로 찾기 때문이다.
- 대분류와 소분류 제목 앞에는 내용을 떠올리기 쉬운 이모지를 하나 붙인다. 훑어볼 때 섹션을 빨리 찾게 해 준다. 같은 이모지를 두 섹션에 쓰지 않는다.
- 대분류(`##`) 앞에는 `---` 구분선을 둔다.
- 표 구분 줄은 `| --- | --- |`로 쓴다.
- 섹션에 덧붙일 설명은 `> 💡` 인용문 한두 문장으로 표 위에 둔다.
- 항목이 기존 소분류에 맞으면 그 표의 끝에 추가한다. 맞는 소분류가 없으면 대분류 안에 새 `###` 소분류를 만든다. 새 도구(예: 다른 CLI)면 새 `##` 대분류를 만든다.

### 추가할 때

1. `~/.config/vscode/HELP.md`를 Read로 읽는다.
2. 위 구조에 따라 들어갈 소분류를 고른다.
3. 같은 단축키나 명령이 이미 있으면 새로 추가하지 않고 기존 행을 고친다.
4. 단축키가 기본값인지 직접 설정한 것인지 사용자의 `keybindings.json`에서 확인한다. 직접 설정한 단축키면 ⚙️를 붙인다.
5. Edit 도구로 해당 표에만 행을 추가한다.

### 작성 규칙

문서 맨 위의 **표기 규칙**을 따른다.

- 키 이름은 맥 기준으로 쓴다: `Cmd`, `Opt`, `Ctrl`, `Shift`. VS Code 문서나 `keybindings.json`의 `alt`는 `Opt`로 바꿔 쓴다.
- 동시에 누르는 키는 ` + `(앞뒤 공백)로 잇는다: `` `Cmd + Shift + P` ``
- 순서대로 누르는 코드 키는 ` → `로 잇는다: `` `Cmd + K → Cmd + I` ``
- 방향키와 범위는 기호로 쓴다: `↑ / ↓`, `1 ~ 9`
- 직접 설정한 단축키는 백틱 앞에 ⚙️를 붙인다: `` ⚙️ `Cmd + K → N` ``
- 백틱(`` ` ``)이 들어간 키는 이중 백틱으로 감싼다: ``` `` Ctrl + ` `` ```
- 설명은 "무엇을 하는지"를 짧은 명사형이나 동사구로 쓴다. 헷갈리기 쉬운 차이는 괄호로 덧붙인다: `다음 줄 실행 (함수 안으로 들어가지 않음)`
- 단축키나 명령이 확실하지 않으면 사용자의 `keybindings.json`이나 VS Code 기본 단축키로 확인하고 쓴다. 틀린 도움말은 없는 것보다 해롭다.
- 사용자가 직접 쓴 문장은 고치지 않는다. 정리를 요청받았을 때만 섹션 순서나 중복을 정리한다.

### 정리할 때

"도움말 정리해줘" 요청이면 다음을 한다.

- 중복 항목 합치기
- 섹션 안에서 비슷한 항목끼리 모으기
- 비어 있는 섹션은 지우지 말고 그대로 두기 (나중에 채울 자리)

정리 전후로 바뀐 항목을 짧게 보고한다.
