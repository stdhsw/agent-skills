# 마크다운 설정

마크다운 파일을 읽기 쉽게 만드는 설정이다. 편집 화면(에디터)과 미리보기(Preview) 화면에 같은 팔레트로 색을 입힌다.

두 화면은 색을 지정하는 방식이 다르다.
- **편집 화면**: 문법 강조(TextMate 스코프)로 칠한다 → `editor.tokenColorCustomizations`
- **Preview**: 마크다운을 HTML로 렌더링해서 보여 준다 → CSS 파일을 `markdown.styles`로 연결

그래서 한쪽만 설정하면 다른 쪽에는 색이 나오지 않는다. 항상 둘을 함께 적용한다.

| 템플릿 | 설치 위치 |
| --- | --- |
| `assets/markdown.json` | `.vscode/settings.json`에 병합 |
| `assets/markdown-preview.css` | `.vscode/markdown-preview.css`로 복사 |

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

이름에 `Dark`나 `Light`가 없는 테마(예: Monokai, Dracula, Solarized 계열 일부)를 쓰는 사용자라면 색이 적용되지 않는다. 사용자가 그런 테마를 쓴다고 말하거나 `workbench.colorTheme` 설정에서 확인되면, 해당 테마 이름으로 키를 하나 더 추가하고 배경 밝기에 맞는 팔레트를 복사해 넣는다.

```jsonc
"[Monokai]": { "textMateRules": [ /* 다크 팔레트와 같은 내용 */ ] }
```

### 3. Preview 색상 (`markdown.styles` + CSS)

`assets/markdown-preview.css`를 프로젝트의 `.vscode/markdown-preview.css`로 복사하고, `settings.json`에 `"markdown.styles": [".vscode/markdown-preview.css"]`를 넣는다. 상대 경로는 VS Code에서 연 폴더를 기준으로 해석된다.

- 색상은 편집 화면 팔레트와 같다. 요소는 HTML 태그로 지정한다: `h1`~`h6`, `strong`, `em`, `del`, 인라인 코드(`:not(pre) > code`), `a`, `blockquote`, 목록 기호(`li::marker`).
- VS Code는 Preview의 `body`에 테마 종류에 따라 `vscode-dark`, `vscode-light`, `vscode-high-contrast` 클래스를 붙인다. CSS는 이 클래스로 다크/라이트를 나눈다.
- 테마 **이름**이 아니라 **종류**로 구분하므로, 편집 화면 설정과 달리 Dracula나 Monokai처럼 이름에 Dark/Light가 없는 테마에서도 따로 처리할 필요가 없다.
- 고대비 테마(`vscode-high-contrast`)는 기본 스타일을 그대로 두기 위해 일부러 지정하지 않는다.

### 4. 마크다운 편집 설정 (`[markdown]`)

`.md` 파일에만 적용되는 언어별 설정이다.

| 설정 | 값 | 이유 |
| --- | --- | --- |
| `editor.wordWrap` | `"on"` | 문단이 한 줄로 길게 이어지는 마크다운 특성상 가로 스크롤 없이 읽게 한다. |
| `editor.quickSuggestions` | 모두 `"off"` | 문장을 쓸 때마다 단어 자동완성 팝업이 뜨는 것을 막는다. |
| `files.trimTrailingWhitespace` | `false` | 마크다운에서 줄 끝 공백 두 칸은 줄바꿈(`<br>`) 문법이다. 전역에서 끝 공백 제거를 켜 두어도 마크다운에서는 지워지지 않게 보호한다. |

## 기존 설정과 병합하기

### `editor.tokenColorCustomizations`

- 키가 없으면 템플릿의 객체를 통째로 추가한다.
- 키가 있으면 `[*Dark*]`, `[*Light*]` 하위 키 단위로 병합한다. 다른 테마 키나 `comments`, `strings` 같은 다른 하위 설정은 그대로 둔다.
- `textMateRules` 배열은 **`scope` 값을 기준으로** 병합한다.
  - 같은 `scope`의 규칙이 이미 있으면 템플릿 규칙으로 바꾸기 전에 사용자에게 확인한다. 사용자가 직접 고른 색일 수 있기 때문이다.
  - 마크다운이 아닌 스코프(예: `comment`, `keyword`)의 규칙은 건드리지 않는다.
  - 템플릿에만 있는 규칙은 배열 끝에 추가한다.

### `[markdown]`

- 키가 없으면 템플릿 객체를 추가한다.
- 키가 있으면 하위 설정 단위로 병합한다. `editor.defaultFormatter`처럼 템플릿에 없는 기존 설정은 그대로 둔다.

### `markdown.styles`

- 키가 없으면 템플릿 값을 추가한다.
- 키가 있으면 배열에 `.vscode/markdown-preview.css`가 없을 때만 **끝에 추가**한다. 기존 CSS는 그대로 둔다. 뒤에 연결된 CSS가 앞의 CSS를 덮으므로, 기존 CSS와 겹치는 색은 이 팔레트가 이긴다는 점을 결과 보고에 적는다.

### `.vscode/markdown-preview.css`

- 파일이 없으면 템플릿을 그대로 복사한다.
- 파일이 이미 있고 내용이 다르면 덮어쓰기 전에 사용자에게 확인한다. 사용자가 직접 고친 CSS일 수 있기 때문이다.

## 색상을 바꾸고 싶을 때

기본 팔레트를 바꾸려면 다음 세 곳을 함께 고친다. 하나라도 어긋나면 편집 화면과 Preview의 색이 달라지거나 결과 보고 내용이 실제와 달라진다.

- `assets/markdown.json` (편집 화면)
- `assets/markdown-preview.css` (Preview)
- 이 문서의 색상 표

## 적용 확인

1. 아무 `.md` 파일을 열어 편집 화면에서 H1~H6 제목과 강조 요소에 색이 들어갔는지 본다.
2. Preview(`Cmd+Shift+V` 또는 `Cmd+K V`)를 열어 같은 색이 나오는지 본다. 바로 바뀌지 않으면 Preview를 닫았다 다시 연다.
3. 편집 화면에서만 색이 안 보이면 현재 테마 이름에 `Dark`/`Light`가 들어 있는지 확인한다(명령 팔레트 → `Preferences: Color Theme`).
4. Preview에서만 색이 안 보이면 Preview 오른쪽 위에 보안 알림이 떠서 CSS를 막고 있는지 확인한다.
5. 편집 화면에서 어떤 스코프가 붙는지 확인하려면 명령 팔레트에서 `Developer: Inspect Editor Tokens and Scopes`를 실행하고 해당 글자에 커서를 둔다.
