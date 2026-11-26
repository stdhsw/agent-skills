# 저장 시 자동 정렬

파일을 저장할 때 언어별 포매터로 코드를 자동 정렬하도록 사용자 `settings.json`을 설정한다. **포매터 확장이 설치된 언어만** 켜는 것이 핵심이다. 확장이 없는 언어에 `editor.defaultFormatter`를 지정하면 저장할 때마다 "포매터를 찾을 수 없음" 알림이 뜨고 정렬도 되지 않는다.

| 항목 | 값 |
| --- | --- |
| 템플릿 | `assets/format/settings.json` |
| 설치 위치 | 사용자 `settings.json` (경로는 `SKILL.md`의 "사용자 설정 폴더") |

## 언어별 포매터

| 언어 | 설정 블록 | 포매터 확장 | 저장할 때 함께 하는 일 | 확장 설치 명령 |
| --- | --- | --- | --- | --- |
| Go | `[go]` | `golang.go` | import 정리 (`source.organizeImports`) | `code --install-extension golang.go` |
| Python | `[python]` | `charliermarsh.ruff` | import 정리 (`source.organizeImports.ruff`) | `code --install-extension charliermarsh.ruff` |
| SQL | `[sql]` | `ms-ossdata.vscode-pgsql` | | `code --install-extension ms-ossdata.vscode-pgsql` |
| Shell | `[shellscript]` | `foxundermoon.shell-format` | | `code --install-extension foxundermoon.shell-format` |
| JSON | `[json]`, `[jsonc]` | `vscode.json-language-features` (VS Code 기본 내장) | | 설치 필요 없음 |
| YAML | `[yaml]` | `redhat.vscode-yaml` | | `code --install-extension redhat.vscode-yaml` |

- 전역 `editor.formatOnSave`는 켜지 않고 `[언어]` 블록 안에서만 켠다. 전역으로 켜면 포매터를 정하지 않은 언어까지 VS Code가 아무 포매터로 정렬해서 남의 저장소 파일이 통째로 바뀔 수 있다.
- Python은 `source.fixAll.ruff`를 넣지 않는다. 쓰지 않는 import나 변수를 지우는 등 코드 내용까지 바꾸기 때문이다. 사용자가 원하면 `[python]`의 `codeActionsOnSave`에 `"source.fixAll.ruff": "explicit"`를 추가한다.
- Shell 확장은 `shfmt`가 없으면 처음 정렬할 때 자동으로 내려받는다. `brew install shfmt`는 터미널에서도 쓰고 싶을 때만 안내한다.
- `jsonc`는 `settings.json`처럼 주석이 있는 JSON이다. `json`과 따로 언어 ID가 있으므로 두 블록을 모두 넣는다.
- 들여쓰기 폭 같은 **정렬 스타일**은 프로젝트의 `ruff.toml`·`pyproject.toml`·`.editorconfig`나 프로젝트 `.vscode/`(`vscode-project`의 몫)가 정한다. 이 기능은 "저장할 때 정렬한다"는 것만 켠다.

---

## 설치

### 1. 설치된 확장 확인

```bash
code --list-extensions
```

위 표의 포매터 확장이 목록에 있는지 언어마다 확인한다. JSON은 VS Code에 내장되어 있어 항상 적용 대상이다.

| 확장 상태 | 처리 |
| --- | --- |
| 설치됨 | 적용 대상에 넣는다 |
| 설치되지 않음 | 적용하지 않는다. 결과 보고에서 표의 설치 명령을 알려 주고, 설치한 뒤 다시 요청하면 추가한다고 안내한다 |
| 표의 확장은 없지만 같은 언어의 다른 포매터 확장이 있음 (예: SQL에 `ReneSaarsoo.sql-formatter-vsc`) | 그 확장으로 설정할지 사용자에게 묻는다 |

확장은 직접 설치하지 않는다. 어떤 확장을 쓸지는 사용자가 고르는 일이다.

### 2. 기존 settings.json 읽기

사용자 `settings.json`을 Read로 읽는다. 다음을 확인한다.

- 같은 `[언어]` 블록이 이미 있는지 (마크다운 기능의 `[markdown]`처럼 다른 기능이 만든 블록이 있을 수 있다)
- 전역 `editor.formatOnSave`가 있는지. `true`면 언어별 블록과 별개로 모든 언어가 정렬되고 있다는 뜻이므로 결과 보고에서 알린다. 지우지는 않는다.

### 3. 언어별 블록 병합

적용 대상 언어의 블록만 템플릿에서 가져와 Edit 도구로 병합한다. 파일이 비어 있거나(`{}`) 없으면 대상 블록만 담아 새로 쓴다.

| 기존 상태 | 처리 |
| --- | --- |
| `[언어]` 블록이 없음 | 템플릿 블록을 그대로 추가한다 |
| 블록은 있지만 템플릿 키가 없음 | 블록 안에 키만 추가한다. 다른 키는 그대로 둔다 |
| 같은 키에 같은 값 | 이미 설정됨. 건너뛴다 |
| `editor.defaultFormatter`가 다른 확장 | 사용자가 고른 포매터일 수 있다. 두 확장을 보여 주고 어느 쪽을 쓸지 묻는다 |
| `editor.formatOnSave`가 `false` | 일부러 끈 것일 수 있다. 켤지 묻는다 |
| `editor.codeActionsOnSave`에 다른 항목이 있음 | 객체 안에 템플릿 항목만 추가한다 |

### 4. 결과 보고

- 자동 정렬을 켠 언어와 포매터
- 확장이 없어 건너뛴 언어와 설치 명령
- 사용자 확인 후 처리한 항목
- 확인 방법: 해당 언어 파일을 열고 들여쓰기를 일부러 흐트린 뒤 저장하면 정렬된다. 정렬되지 않으면 명령 팔레트에서 `Format Document With...` → `Configure Default Formatter...`로 포매터가 잡히는지 확인한다.
- 정렬하지 않고 저장하려면 `Cmd + K → S` (`workbench.action.files.saveWithoutFormatting`)

---

## 언어 추가하기

사용자가 "Rust도 저장할 때 정렬해줘"처럼 새 언어를 요청하면:

1. **언어 ID와 포매터 확장 확인**: 언어 ID는 VS Code 오른쪽 아래 상태 표시줄의 언어 이름을 눌러 확인한다(예: Shell의 ID는 `shellscript`). 포매터 확장은 설치된 확장의 `package.json`이나 마켓플레이스에서 확인한다. 확실하지 않으면 사용자에게 확인을 부탁한다. 잘못된 확장 ID는 오류 없이 정렬만 안 된다.
2. **두 곳을 함께 고친다**:
   - `assets/format/settings.json`: 블록 추가
   - 이 문서의 "언어별 포매터" 표: 행 추가
3. **이 PC에 적용**: 위 "설치" 절차대로 확장이 설치되어 있을 때만 사용자 `settings.json`에 넣는다.

언어를 빼는 경우도 같은 두 곳과 사용자 `settings.json`에서 함께 지운다.

## 제거

사용자가 자동 정렬을 되돌리고 싶어 하면, 사용자 `settings.json`의 각 `[언어]` 블록에서 템플릿에 있는 키(`editor.defaultFormatter`, `editor.formatOnSave`, 템플릿의 `codeActionsOnSave` 항목)만 지운다. 블록에 다른 키가 남지 않으면 블록도 지운다. 사용자가 직접 넣은 키는 남긴다.
