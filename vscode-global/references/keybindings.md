# 단축키

직접 정한 단축키를 사용자 `keybindings.json`에 넣어 모든 프로젝트에서 쓰게 한다. 단축키 목록은 템플릿 한 곳에서 관리하고, 새 PC나 설정을 초기화한 VS Code에서 같은 단축키를 그대로 되살리는 것이 목적이다.

| 항목 | 값 |
| --- | --- |
| 템플릿 | `assets/keybindings/keybindings.json` |
| 설치 위치 | 사용자 `keybindings.json` (경로는 `SKILL.md`의 "사용자 설정 폴더") |

`cmd+alt+h`(도움말 열기)는 도움말 기능이 함께 관리하므로 이 목록에 넣지 않는다(`references/help.md`).

---

## 단축키 목록

템플릿에 있는 단축키와 그 이유다. 도움말에 적을 때 쓰는 표기도 함께 둔다.

| 도움말 표기 | `key` | `command` | 하는 일 | 기본 동작과의 관계 |
| --- | --- | --- | --- | --- |
| `Cmd + Shift + ]` | `shift+cmd+]` | `workbench.action.nextSideBarView` | 다음 사이드바 보기로 이동 | 기본값인 "다음 편집기 탭"을 대체 |
| `Cmd + Shift + [` | `shift+cmd+[` | `workbench.action.previousSideBarView` | 이전 사이드바 보기로 이동 | 기본값인 "이전 편집기 탭"을 대체 |
| `Cmd + K → Cmd + N` | `cmd+k cmd+n` | `workbench.action.moveEditorToNewWindow` | 현재 편집기를 새 창으로 이동 | 새로 추가 |
| `Cmd + Q → Cmd + Q` | `cmd+q cmd+q` | `workbench.action.quit` | `Cmd + Q`를 두 번 눌러야 종료 (실수로 끄는 것 방지) | 한 번 누르면 종료되던 동작을 대체 |
| `Ctrl + Shift + U` | `ctrl+shift+u` | `editor.action.transformToUppercase` | 선택한 글자를 대문자로 | 새로 추가 |
| `Ctrl + Shift + L` | `ctrl+shift+l` | `editor.action.transformToLowercase` | 선택한 글자를 소문자로 | 새로 추가 |
| `Cmd + Opt + S` | `alt+cmd+s` | `workbench.action.files.saveAll` | 열린 파일 모두 저장 | macOS 기본값과 같음 (명시해 둠) |

키 이름은 macOS 기준이다. Linux와 Windows에서는 `cmd`를 `ctrl`로 바꿔 넣는다. 바꾼 결과가 다른 단축키와 겹치면 사용자에게 묻는다.

---

## 설치

### 1. 기존 keybindings.json 읽기

사용자 `keybindings.json`을 Read로 읽는다. JSONC라서 맨 위에 주석(`// 키 바인딩을 이 파일에 넣어서...`)이 있을 수 있다. 주석과 기존 항목을 지우지 않도록 Edit 도구로 고친다. 파일이 없으면 템플릿 배열을 그대로 담아 Write로 만든다.

### 2. 항목별 병합

템플릿의 각 항목을 `key` 기준으로 기존 배열과 비교한다.

| 기존 상태 | 처리 |
| --- | --- |
| 같은 `key`와 같은 `command`가 있음 | 이미 설정됨. 건너뛴다 |
| 같은 `key`가 없음 | 배열 끝에 추가한다 |
| 같은 `key`에 다른 `command`가 있음 | 사용자가 다른 용도로 쓰는 키일 수 있다. 두 명령을 보여 주고 어느 쪽을 쓸지 묻는다 |
| 같은 `command`가 다른 `key`에 있음 | 같은 기능을 다른 키로 쓰고 있다. 템플릿 키를 추가할지, 기존 키를 유지할지 묻는다 |

`"-"`로 시작하는 `command`(기본 단축키 해제 항목)는 사용자가 일부러 넣은 것이므로 건드리지 않는다.

### 3. 결과 보고

- 추가한 단축키, 이미 있어서 건너뛴 단축키, 사용자 확인 후 처리한 단축키
- 기본 동작을 대체한 단축키(위 표의 "기본 동작과의 관계")가 있으면 무엇이 바뀌는지
- 단축키는 저장하면 바로 반영된다. 반영되지 않으면 `Developer: Reload Window`

---

## 단축키 추가하기

사용자가 "이 단축키도 전역에 넣어줘", "단축키 추가해줘"라고 하면:

1. **명령 ID 확인**: 사용자가 기능을 말로 설명하면 VS Code 명령 ID를 찾는다. 확실하지 않으면 명령 팔레트에서 `Preferences: Open Keyboard Shortcuts`를 열어 기능 이름으로 검색하고, 오른쪽 클릭 → `Copy Command ID`로 확인해 달라고 한다. 잘못된 명령 ID는 아무 오류 없이 동작만 안 한다.
2. **충돌 확인**: 원하는 키가 이미 다른 기능에 쓰이는지 확인한다. 사용자 `keybindings.json`을 보고, 기본 단축키는 같은 화면에서 키를 눌러 검색(`Record Keys`)해 달라고 한다.
3. **세 곳을 함께 고친다**. 하나만 고치면 다른 PC에 설치할 때 빠지거나 도움말과 실제가 어긋난다.
   - `assets/keybindings/keybindings.json`: 항목 추가
   - 이 문서의 "단축키 목록" 표: 행 추가
   - `assets/help/HELP.md`: 알맞은 소분류에 ⚙️ 표시와 함께 행 추가 (표기 규칙은 `references/help.md`)
4. **이 PC에 적용**: 위 "설치"의 병합 규칙대로 사용자 `keybindings.json`에 넣는다. 도움말이 설치되어 있으면(`~/.config/vscode/HELP.md`) 그 파일에도 같은 행을 추가한다.

단축키를 빼는 경우도 같은 세 곳과 사용자 `keybindings.json`에서 함께 지운다.

## 제거

사용자가 전역 단축키를 되돌리고 싶어 하면, 사용자 `keybindings.json`에서 이 템플릿에 있는 항목(같은 `key`와 `command`)만 지운다. 사용자가 직접 넣은 다른 항목과 도움말 단축키(`cmd+alt+h`)는 남긴다.
