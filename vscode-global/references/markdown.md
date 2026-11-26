# 마크다운

마크다운 파일을 읽기 쉽게 만드는 설정이다. 편집 화면(에디터)과 미리보기(Preview) 화면에 같은 팔레트로 색을 입힌다.

두 화면은 색을 지정하는 방식이 다르다.
- **편집 화면**: 문법 강조(TextMate 스코프)로 칠한다 → 사용자 `settings.json`의 `editor.tokenColorCustomizations`
- **Preview**: 마크다운을 HTML로 렌더링해서 보여 준다 → 로컬 확장의 `markdown.previewStyles`로 CSS 연결

그래서 한쪽만 설정하면 다른 쪽에는 색이 나오지 않는다. 항상 둘을 함께 적용한다.

| 템플릿 | 설치 위치 |
| --- | --- |
| `assets/markdown/colors.json` | 사용자 `settings.json`에 병합 |
| `assets/markdown/preview.css` | 로컬 확장의 `preview.css`로 묶어 설치 |
| `assets/markdown/preview-extension/package.json` | 로컬 확장의 `package.json` |

---

## 적용 내용

### 1. 요소별 색상 (`editor.tokenColorCustomizations`)

VS Code 내장 마크다운 문법이 붙이는 TextMate 스코프에 색을 지정한다.

| 요소 | 스코프 | 다크 | 라이트 | 글꼴 |
| --- | --- | --- | --- | --- |
| H1 | `heading.1.markdown` | `#F7768E` 빨강 | `#F52A65` | 굵게 |
| H2 | `heading.2.markdown` | `#FF9E64` 주황 | `#B15C00` | 굵게 |
| H3 | `heading.3.markdown` | `#E0AF68` 노랑 | `#8C6C3E` | 굵게 |
| H4 | `heading.4.markdown` | `#9ECE6A` 초록 | `#587539` | 굵게 |
| H5 | `heading.5.markdown` | `#7AA2F7` 파랑 | `#2E7DE9` | 굵게 |
| H6 | `heading.6.markdown` | `#BB9AF7` 보라 | `#9854F1` | 굵게 |
| 굵게 | `markup.bold.markdown` | `#FF9E64` | `#B15C00` | 굵게 |
| 기울임 | `markup.italic.markdown` | `#BB9AF7` | `#9854F1` | 기울임 |
| 취소선 | `markup.strikethrough.markdown` | `#565F89` | `#848CB5` | 취소선 |
| 인라인 코드 | `markup.inline.raw.string.markdown` | `#73DACA` | `#118C74` | - |
| 링크 텍스트 | `string.other.link.title.markdown` | `#7AA2F7` | `#2E7DE9` | - |
| 링크 주소 | `markup.underline.link.markdown` | `#565F89` | `#848CB5` | 밑줄 |
| 인용문 | `markup.quote.markdown` | `#9AA5CE` | `#6172B0` | 기울임 |
| 목록 기호 | `punctuation.definition.list.begin.markdown` | `#7DCFFF` | `#007197` | 굵게 |

팔레트는 도쿄 나이트(다크)와 도쿄 나이트 데이(라이트) 테마의 색을 따른다. 제목은 H1부터 H6까지 빨강→보라 순서로 색을 바꿔, 문서 구조의 깊이가 색만으로 보이게 했다. 라이트 팔레트는 흰 배경에서 대비가 충분하도록 같은 계열의 진한 색을 쓴다.

Preview CSS는 위 표에 없는 색 하나를 더 쓴다. 인용문 왼쪽 세로줄 색으로, 다크는 `#3B4261`, 라이트는 `#C4C8DA`다.

### 2. 테마별 범위 지정

색상은 `"[*Dark*]"`, `"[*Light*]"` 키 안에 넣는다. VS Code는 테마 이름에 와일드카드(`*`)를 지원하므로, 이름에 `Dark`가 들어간 테마(Default Dark Modern, One Dark Pro 등)에는 다크 팔레트가, `Light`가 들어간 테마에는 라이트 팔레트가 적용된다.

이름에 `Dark`나 `Light`가 없는 테마(예: Monokai, Dracula, Solarized 계열 일부)를 쓰는 사용자라면 편집 화면에 색이 적용되지 않는다. 사용자가 그런 테마를 쓴다고 말하거나 `workbench.colorTheme` 설정에서 확인되면, 해당 테마 이름으로 키를 하나 더 추가하고 배경 밝기에 맞는 팔레트를 복사해 넣는다.

```jsonc
"[Monokai]": { "textMateRules": [ /* 다크 팔레트와 같은 내용 */ ] }
```

### 3. Preview 색상 (로컬 확장)

`assets/markdown/preview.css`를 로컬 확장의 `preview.css`로 묶어 설치한다.

- 요소는 HTML 태그로 지정한다: `h1`~`h6`, `strong`, `em`, `del`, 인라인 코드(`:not(pre) > code`), `a`, `blockquote`, 목록 기호(`li::marker`).
- VS Code는 Preview의 `body`에 테마 종류에 따라 `vscode-dark`, `vscode-light`, `vscode-high-contrast` 클래스를 붙인다. CSS는 이 클래스로 다크/라이트를 나눈다.
- 테마 **이름**이 아니라 **종류**로 구분하므로, 편집 화면과 달리 Dracula나 Monokai에서도 따로 처리할 필요가 없다.
- 고대비 테마(`vscode-high-contrast`)는 기본 스타일을 그대로 두기 위해 일부러 지정하지 않는다.

**왜 확장으로 넣는가**: Preview는 웹뷰라서 허용된 폴더(열린 프로젝트 폴더, 프로젝트 없이 연 파일의 폴더, 확장 폴더)의 파일만 읽는다. 그래서 사용자 설정의 `markdown.styles`에 절대 경로 CSS를 넣어도 대부분의 프로젝트에서 막힌다. `data:` 주소는 상대 경로로 처리되어 동작하지 않고, https URL은 인터넷 연결과 CDN 캐시에 의존한다. 확장이 `contributes."markdown.previewStyles"`로 선언한 CSS는 확장 폴더 안에 있어서 어느 프로젝트에서든 읽힌다. 스타일만 추가하므로 실행 코드나 활성화 이벤트가 필요 없다.

확장 CSS는 `markdown.styles` CSS보다 **먼저** 로드된다. 그래서 어떤 프로젝트의 `.vscode/settings.json`에 `markdown.styles`가 따로 있으면 그 프로젝트에서는 프로젝트 CSS가 이긴다.

### 4. 마크다운 편집 설정 (`[markdown]`)

`.md` 파일에만 적용되는 언어별 설정이다.

| 설정 | 값 | 이유 |
| --- | --- | --- |
| `editor.wordWrap` | `"on"` | 문단이 한 줄로 길게 이어지는 마크다운 특성상 가로 스크롤 없이 읽게 한다. |
| `editor.quickSuggestions` | 모두 `"off"` | 문장을 쓸 때마다 단어 자동완성 팝업이 뜨는 것을 막는다. |
| `files.trimTrailingWhitespace` | `false` | 마크다운에서 줄 끝 공백 두 칸은 줄바꿈(`<br>`) 문법이다. 전역에서 끝 공백 제거를 켜 두어도 마크다운에서는 지워지지 않게 보호한다. |

---

## 설치

### 1. 사용자 settings.json 병합

`assets/markdown/colors.json`의 내용을 사용자 `settings.json`에 병합한다.

**`editor.tokenColorCustomizations`**
- 키가 없으면 템플릿의 객체를 통째로 추가한다.
- 키가 있으면 `[*Dark*]`, `[*Light*]` 하위 키 단위로 병합한다. 다른 테마 키나 `comments`, `strings` 같은 다른 하위 설정은 그대로 둔다.
- `textMateRules` 배열은 **`scope` 값을 기준으로** 병합한다.
  - 같은 `scope`의 규칙이 이미 있으면 템플릿 규칙으로 바꾸기 전에 사용자에게 확인한다. 사용자가 직접 고른 색일 수 있기 때문이다.
  - 마크다운이 아닌 스코프(예: `comment`, `keyword`)의 규칙은 건드리지 않는다.
  - 템플릿에만 있는 규칙은 배열 끝에 추가한다.

**`[markdown]`**
- 키가 없으면 템플릿 객체를 추가한다.
- 키가 있으면 하위 설정 단위로 병합한다. `editor.defaultFormatter`처럼 템플릿에 없는 기존 설정은 그대로 둔다.

**기존 `markdown.styles` 확인**
- 사용자 설정에 `markdown.styles`가 있으면 각 경로를 확인한다.
- 파일이 없는 경로, 또는 위 이유로 대부분의 프로젝트에서 로드되지 않는 절대 경로가 있으면 그 사실을 알리고 지울지 묻는다.
- https URL처럼 동작할 수 있는 항목은 그대로 두고, 확장 CSS보다 나중에 로드되어 색을 덮을 수 있다고 알린다.

### 2. Preview 확장 설치

`npx`가 필요하다. 없으면 Node.js 설치를 안내한다(macOS: `brew install node`). 런타임 설치는 직접 하지 않는다.

확장은 임시 폴더에서 조립한다. 저장소 안에 `.vsix`나 복사본이 남지 않게 하기 위해서다.

```bash
SKILL="<이 스킬 폴더>"
BUILD=$(mktemp -d)
cp "$SKILL/assets/markdown/preview-extension/package.json" "$BUILD/"
cp "$SKILL/assets/markdown/preview.css" "$BUILD/preview.css"
(cd "$BUILD" && npx --yes @vscode/vsce package --allow-missing-repository --skip-license -o markdown-preview-colors.vsix)
code --install-extension "$BUILD/markdown-preview-colors.vsix" --force
rm -rf "$BUILD"
```

- CSS는 `assets/markdown/preview.css`를 복사해 쓴다. 팔레트를 한 곳에서만 관리하기 위해서다.
- `--force`는 같은 버전이 이미 설치되어 있어도 다시 설치한다. 팔레트를 바꾼 뒤 재설치할 때 버전을 올리지 않아도 된다.
- `npx --yes @vscode/vsce`는 처음 실행할 때 npm에서 VS Code 공식 패키징 도구를 내려받는다. 실행 전에 이 점을 사용자에게 알린다.

설치 확인:

```bash
code --list-extensions --show-versions | grep markdown-preview-colors
```

`local.markdown-preview-colors@0.0.1`이 보이면 설치된 것이다.

### 3. 결과 보고

- 사용자 `settings.json`에 추가·병합한 키와 사용자 확인 후 바꾼 키
- 정리한 `markdown.styles` 항목
- 설치한 확장 이름과 버전
- 확장은 마켓플레이스 확장이 아니라서 Settings Sync로 다른 PC에 넘어가지 않으므로, 다른 PC에서는 이 스킬로 다시 설치해야 한다는 안내

---

## 팔레트 바꾸기

기본 팔레트를 바꾸려면 다음 세 곳을 함께 고친다. 하나라도 어긋나면 편집 화면과 Preview의 색이 달라지거나 결과 보고 내용이 실제와 달라진다.

- `assets/markdown/colors.json` (편집 화면)
- `assets/markdown/preview.css` (Preview)
- 이 문서의 색상 표

이미 설치했다면 다음으로 다시 적용한다.

1. 사용자 `settings.json`의 색상 규칙을 새 값으로 고친다(Edit 도구).
2. 위 "Preview 확장 설치"를 다시 실행한다(`--force`로 덮어씀).
3. `Developer: Reload Window`

## 제거

사용자가 전역 마크다운 설정을 되돌리고 싶어 하면:

1. 사용자 `settings.json`에서 이 스킬이 넣은 마크다운 색상 규칙(`[*Dark*]`, `[*Light*]` 안의 마크다운 스코프)과 `[markdown]` 설정을 지운다. 사용자가 직접 넣은 다른 규칙은 남긴다.
2. 확장을 제거한다.

```bash
code --uninstall-extension local.markdown-preview-colors
```

## 적용 확인

1. 아무 `.md` 파일을 열어 편집 화면에서 H1~H6 제목과 강조 요소에 색이 들어갔는지 본다.
2. Preview(`Cmd+Shift+V` 또는 `Cmd+K V`)를 열어 같은 색이 나오는지 본다. 바로 바뀌지 않으면 Preview를 닫았다 다시 연다.
3. 편집 화면에서만 색이 안 보이면 현재 테마 이름에 `Dark`/`Light`가 들어 있는지 확인한다(명령 팔레트 → `Preferences: Color Theme`).
4. Preview에서만 색이 안 보이면 `code --list-extensions | grep markdown-preview-colors`로 확장이 설치되어 있는지 확인한다.
5. 특정 프로젝트에서만 색이 다르면 그 프로젝트의 `.vscode/settings.json`에 색상이나 `markdown.styles`가 따로 있는지 확인한다. 같은 설정은 프로젝트 값이 우선하고, `markdown.styles` CSS는 확장 CSS보다 나중에 로드되어 덮어쓴다.
6. 편집 화면에서 어떤 스코프가 붙는지 확인하려면 명령 팔레트에서 `Developer: Inspect Editor Tokens and Scopes`를 실행하고 해당 글자에 커서를 둔다.
