---
name: vscode-global
description: >
  이 PC의 모든 VS Code 프로젝트에 적용되는 전역 설정을 관리하는 스킬. 사용자 settings.json·tasks.json·
  keybindings.json과 로컬 확장에 설치한다. 기능은 세 가지다:
  마크다운(제목·강조 요소 색상을 편집 화면과 Preview에 적용, 마크다운 편집 설정),
  도움말(cmd+alt+h로 여는 개인 도움말 HELP.md 설치와 단축키·명령어 추가·정리),
  단축키(직접 정한 단축키를 keybindings.json에 설치하고 새 단축키 추가).
  "vscode 전역 설정", "마크다운 색상 설정", "미리보기에 색이 안 나와", "팔레트 다시 적용해줘",
  "vscode 도움말 만들어줘", "도움말에 추가해줘", "이 단축키 도움말에 적어줘", "vscode 단축키 설정해줘",
  "이 단축키 전역에 추가해줘", "새 PC에 vscode 설정 옮겨줘" 등을 요청할 때 사용하세요.
  VS Code 편집기 설정, 마크다운 색상, 개인 단축키·치트시트를 다루는 맥락이면 적극적으로 사용하세요.
  특정 프로젝트의 .vscode/ 설정은 vscode-project 스킬이, Neovim은 nvim-helper 스킬이 맡는다.
---

# VS Code Global

VS Code **사용자 영역**에 설정을 넣어 이 PC의 모든 프로젝트에 적용하는 스킬이다. 한 번 설치하면 새 프로젝트에서 따로 설정할 필요가 없고, 새 PC에서도 이 스킬로 같은 환경을 되살릴 수 있다.

프로젝트 `.vscode/`에는 아무것도 넣지 않는다. 특정 프로젝트에만 적용하거나 팀과 공유할 설정은 `vscode-project` 스킬의 몫이다.

## 한눈에 보기

| 기능 | 하는 일 | 상세 문서 | 템플릿 |
| --- | --- | --- | --- |
| 마크다운 | 제목·강조 요소 색상(편집 화면 + Preview, 다크/라이트별) + 마크다운 편집 설정 | `references/markdown.md` | `assets/markdown/` |
| 도움말 | `cmd+alt+h`로 여는 개인 도움말 설치 + 내용 추가·정리 | `references/help.md` | `assets/help/HELP.md` |
| 단축키 | 직접 정한 단축키를 사용자 `keybindings.json`에 설치 + 새 단축키 추가 | `references/keybindings.md` | `assets/keybindings/keybindings.json` |

## 작업 흐름

1. **기능 고르기**: 요청한 기능만 다룬다. "vscode 전역 설정해줘"처럼 지정이 없으면 세 기능을 모두 설치하고, 이미 설치된 기능은 상태만 확인한다.
2. **상세 문서 읽기**: 해당 기능의 상세 문서를 읽고 그 절차를 따른다. 설치, 병합 규칙, 재적용·제거 방법이 거기에 있다.
3. **사용자 설정 폴더 확인**: 아래 경로에서 고칠 파일을 찾는다.
4. **병합**: 아래 "설정 파일 병합 규칙"대로 고친다.
5. **결과 보고**: 상세 문서의 결과 보고 항목과 아래 공통 항목을 알린다.

## 사용자 설정 폴더

| OS | 경로 |
| --- | --- |
| macOS | `~/Library/Application Support/Code/User/` |
| Linux | `~/.config/Code/User/` |
| Windows | `%APPDATA%\Code\User\` |

- VS Code Insiders는 폴더 이름이 `Code` 대신 `Code - Insiders`다. 두 폴더가 모두 있으면 어느 쪽에 설치할지 묻는다.
- 로컬 확장은 `~/.vscode/extensions/`(Windows는 `%USERPROFILE%\.vscode\extensions`)에 설치된다.
- 개인 파일(도움말)은 `~/.config/vscode/`에 둔다.
- 마크다운과 도움말 기능은 `code` CLI가 필요하다(`command -v code`). 없으면 명령 팔레트에서 `Shell Command: Install 'code' command in PATH`를 실행해 달라고 안내한다. 시스템 PATH를 바꾸는 일이라 직접 하지 않는다.

## 설정 파일 병합 규칙

사용자 설정은 모든 프로젝트에 영향을 주므로 다음을 지킨다.

- `settings.json`, `tasks.json`, `keybindings.json`은 JSONC(주석과 끝 쉼표 허용)다. 일반 JSON 파서로 읽고 다시 쓰면 주석이 사라지므로, Read로 읽은 뒤 Edit 도구로 필요한 부분만 고친다. 파일이 없으면 Write로 새로 만든다.
- 템플릿에 있는 키가 기존 파일에 없으면 추가한다.
- 같은 키가 이미 있고 값이 객체면 하위 키 단위로 병합한다.
- 같은 키에 사용자가 다른 값을 넣어 두었다면 덮어쓰기 전에 어느 쪽을 쓸지 묻는다.
- 배열(`textMateRules`, `tasks`, 단축키 목록)은 기능별 상세 문서의 병합 방법을 따른다.

## 결과 보고

기능별 상세 문서의 결과 보고 항목에 더해 알린다.

- 열려 있는 VS Code 창은 `Developer: Reload Window`를 실행해야 반영된다.
- 프로젝트 `.vscode/`에 같은 설정이 따로 있는 프로젝트에서는 프로젝트 설정이 우선한다.

## 하지 않는 일

- 프로젝트 `.vscode/`에 설정 넣기 (`vscode-project`의 몫)
- 스크립트로 JSONC 설정 파일을 다시 쓰기 (주석이 사라진다)
- 확인 없이 기존 사용자 값 덮어쓰기
- `code` CLI, Node.js 같은 도구를 직접 설치하기 (안내만 한다)
- 사용자가 쌓아 온 도움말 파일(`~/.config/vscode/HELP.md`)을 템플릿으로 덮어쓰기
