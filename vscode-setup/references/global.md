# 전역 모드

설정을 프로젝트 `.vscode/`가 아니라 **VS Code 사용자 영역**에 넣어 모든 프로젝트에 적용하는 방법이다. 한 번 설치하면 새 프로젝트에서 따로 설정할 필요가 없다.

## 설치 위치

| 대상 | 위치 | 방법 |
| --- | --- | --- |
| 편집 화면 색상, `[markdown]` 편집 설정 | 사용자 `settings.json` | Edit 도구로 병합 |
| Preview 색상 | 로컬 확장 (`~/.vscode/extensions/local.markdown-preview-colors-<버전>/`) | VSIX로 묶어 `code --install-extension` |

### 사용자 settings.json 경로

| OS | 경로 |
| --- | --- |
| macOS | `~/Library/Application Support/Code/User/settings.json` |
| Linux | `~/.config/Code/User/settings.json` |
| Windows | `%APPDATA%\Code\User\settings.json` |

VS Code Insiders는 폴더 이름이 `Code` 대신 `Code - Insiders`다. 두 폴더가 모두 있으면 어느 쪽에 설치할지 묻는다.

## 왜 Preview는 확장으로 넣는가

Preview는 웹뷰라서 허용된 폴더(열린 프로젝트 폴더, 프로젝트 없이 연 파일의 폴더, 확장 폴더)의 파일만 읽는다. 그래서 사용자 설정의 `markdown.styles`에 절대 경로 CSS를 넣어도 대부분의 프로젝트에서 막힌다. `data:` 주소는 상대 경로로 처리되어 동작하지 않고, https URL은 인터넷 연결과 CDN 캐시에 의존한다.

확장이 `contributes."markdown.previewStyles"`로 선언한 CSS는 확장 폴더 안에 있어서 어느 프로젝트에서든 읽힌다. 스타일만 추가하므로 실행 코드나 활성화 이벤트가 필요 없다.

확장 CSS는 `markdown.styles` CSS보다 **먼저** 로드된다. 그래서 어떤 프로젝트에 `vscode-setup` 프로젝트 모드로 CSS를 넣으면 그 프로젝트에서는 프로젝트 CSS가 이긴다. 프로젝트별로 덮어쓸 수 있다는 뜻이다.

---

## 1. 사용자 settings.json 병합

`assets/markdown.json`의 내용을 사용자 `settings.json`에 병합한다. 병합 규칙은 `references/markdown.md`의 "기존 설정과 병합하기"와 같다. 사용자 설정은 모든 프로젝트에 영향을 주므로 기존 값과 충돌하면 반드시 사용자에게 묻는다.

전역 모드에서 달라지는 점:

- **`markdown.styles`는 넣지 않는다.** 템플릿의 `".vscode/markdown-preview.css"`는 프로젝트 기준 상대 경로라서 전역에서는 의미가 없다. Preview는 2단계의 확장이 맡는다.
- **기존 `markdown.styles` 확인**: 사용자 설정에 이미 `markdown.styles`가 있으면 각 경로를 확인한다.
  - 파일이 없는 경로, 또는 위 이유로 대부분의 프로젝트에서 로드되지 않는 절대 경로가 있으면 그 사실을 알리고 지울지 묻는다.
  - https URL처럼 동작할 수 있는 항목은 그대로 두고, 확장 CSS보다 나중에 로드되어 색을 덮을 수 있다고 알린다.

## 2. Preview 확장 설치

### 준비 확인

```bash
command -v code   # 없으면 명령 팔레트에서 "Shell Command: Install 'code' command in PATH" 안내
command -v npx    # 없으면 Node.js 설치 안내 (macOS: brew install node)
```

시스템 PATH를 바꾸거나 런타임을 설치하는 일이라 직접 하지 않고 사용자에게 안내한다.

### 빌드와 설치

확장은 임시 폴더에서 조립한다. 저장소 안에 `.vsix`나 복사본이 남지 않게 하기 위해서다.

```bash
SKILL="<이 스킬 폴더>"
BUILD=$(mktemp -d)
cp "$SKILL/assets/markdown-preview-extension/package.json" "$BUILD/"
cp "$SKILL/assets/markdown-preview.css" "$BUILD/preview.css"
(cd "$BUILD" && npx --yes @vscode/vsce package --allow-missing-repository --skip-license -o markdown-preview-colors.vsix)
code --install-extension "$BUILD/markdown-preview-colors.vsix" --force
rm -rf "$BUILD"
```

- CSS는 프로젝트 모드와 같은 `assets/markdown-preview.css`를 복사해 쓴다. 팔레트를 한 곳에서만 관리하기 위해서다.
- `--force`는 같은 버전이 이미 설치되어 있어도 다시 설치한다. 팔레트를 바꾼 뒤 재설치할 때 버전을 올리지 않아도 된다.
- `npx --yes @vscode/vsce`는 처음 실행할 때 npm에서 VS Code 공식 패키징 도구를 내려받는다. 실행 전에 이 점을 사용자에게 알린다.

### 확인

```bash
code --list-extensions --show-versions | grep markdown-preview-colors
```

`local.markdown-preview-colors@0.0.1`이 보이면 설치된 것이다. 열려 있는 VS Code 창은 `Developer: Reload Window`를 실행해야 반영된다.

## 3. 결과 보고

- 사용자 `settings.json`에 추가·병합한 키와 사용자 확인 후 바꾼 키
- 정리한 `markdown.styles` 항목
- 설치한 확장 이름과 버전
- 프로젝트 `.vscode/`에 이미 마크다운 설정이 있는 프로젝트는 그 설정이 우선한다는 안내
- 확장은 마켓플레이스 확장이 아니라서 Settings Sync로 다른 PC에 넘어가지 않으므로, 다른 PC에서는 이 스킬로 다시 설치해야 한다는 안내

---

## 팔레트를 바꾼 뒤 다시 적용하기

1. 사용자 `settings.json`의 색상 규칙을 새 값으로 고친다(Edit 도구).
2. 위 "빌드와 설치"를 다시 실행한다(`--force`로 덮어씀).
3. `Developer: Reload Window`

## 전역 설정 제거

사용자가 전역 적용을 되돌리고 싶어 하면:

1. 사용자 `settings.json`에서 이 스킬이 넣은 마크다운 색상 규칙(`[*Dark*]`, `[*Light*]` 안의 마크다운 스코프)과 `[markdown]` 설정을 지운다. 사용자가 직접 넣은 다른 규칙은 남긴다.
2. 확장을 제거한다.

```bash
code --uninstall-extension local.markdown-preview-colors
```
