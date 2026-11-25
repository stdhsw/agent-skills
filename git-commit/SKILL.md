---
name: git-commit
description: >
  Git의 staged 변경사항을 분석해 Conventional Commits 형식의 한국어 제목과 Why/What 본문(한국어),
  git 표준 트레일러(Refs, Breaking-Change, Follow-up, Co-Authored-By)로 커밋하는 스킬.
  나중에 Claude Code가 git log와 git blame만 보고도 변경 이유와 후속 작업을 파악할 수 있도록 구조화된 메시지를 작성한다.
  의도가 다른 변경이 섞여 있으면 커밋 분할을 먼저 제안한다.
  사용자가 "커밋해줘", "커밋 진행", "변경사항 커밋", "commit", "git commit", "커밋 해줘",
  "커밋 메시지 작성", "변경사항 반영", "지금까지 작업한거 커밋", "변경사항 저장", "커밋 나눠서 해줘" 등
  커밋 관련 요청을 할 때 이 스킬을 사용하세요.
  코드 변경 후 사용자가 결과를 확인하고 저장/반영을 원하는 맥락에서도 적극적으로 트리거하세요.
  단, PR 생성과 머지는 git-pr 스킬이 담당한다.
---

# Git Commit

staged된 변경사항만 분석해 구조화된 커밋 메시지로 커밋하는 스킬이다.

커밋 메시지는 사람만 읽는 기록이 아니다. 다음 세션의 Claude Code가 `git log`, `git blame`, `git log --grep`으로 코드의 맥락을 되짚을 때 가장 먼저 읽는 자료다. diff는 **무엇이** 바뀌었는지 보여 주지만 **왜** 바꿨는지는 커밋 메시지에만 남는다. 그래서 이 스킬은 이유(Why)와 변경 내용(What)을 나누고, 후속 작업 같은 메타데이터를 git 표준 트레일러로 남긴다.

---

## 커밋 메시지 형식

```
type(scope): 한국어 제목
                                   ← 빈 줄
Why:
- 변경 이유 (한국어)
                                   ← 빈 줄
What:
- 변경 내용 (한국어)
                                   ← 빈 줄
Refs: #42
Follow-up: 후속 작업
Co-Authored-By: Claude <noreply@anthropic.com>
```

| 부분 | 언어 | 필수 여부 |
| --- | --- | --- |
| 제목 | type과 scope는 영어, 설명은 한국어 | 항상 |
| `Why:` 섹션 | 한국어 | 이유가 diff만으로 드러나지 않으면 작성 |
| `What:` 섹션 | 한국어 | 변경이 두 가지 이상이거나 제목만으로 부족하면 작성 |
| 트레일러 | 키는 영어, 값은 한국어 가능 | 해당할 때만 작성 |

### 본문 생략 기준

- 오타 수정, 포매팅, 단일 설정값 변경처럼 제목만으로 충분하면 본문 전체를 생략한다.
- 이유가 자명하면(예: 의존성 버전 올림) `Why:`를 생략하고 `What:`만 쓴다.
- 빈 섹션을 형식상 남기지 않는다. 내용 없는 레이블은 읽는 쪽에 잡음일 뿐이다.

---

## 제목

```
type(scope): subject
type(scope)!: subject      ← 호환성을 깨는 변경
```

- `type`과 `scope`는 영어 소문자로 쓴다. Conventional Commits 도구(changelog 생성, 버전 판단)가 이 부분을 읽기 때문이다.
- `subject`(설명)는 **한국어**로 쓴다.
- 설명은 "~ 추가", "~ 수정"처럼 **명사형으로 끝낸다**. "~했습니다", "~함"처럼 문장으로 끝내지 않는다.
- type, scope를 포함해 **50자 이내**로 쓴다. 한글은 영문보다 넓게 보여서 영어 기준(70자)보다 짧게 잡는다.
- 끝에 마침표를 찍지 않는다.
- 코드 식별자, 파일명, 명령어는 원래 이름 그대로 쓴다: `feat(vscode-help): cmd+alt+h로 여는 전역 도움말 스킬 추가`
- 호환성을 깨는 변경이면 type(scope) 뒤에 `!`를 붙이고 `Breaking-Change` 트레일러를 함께 쓴다.

### Type

변경의 **의도**로 고른다. 커밋 하나에는 type 하나만 쓴다.

| Type | 선택 기준 |
| --- | --- |
| feat | 사용자가 체감하는 새 기능 추가 |
| fix | 의도대로 동작하지 않던 것을 고침 |
| docs | README, 가이드, 주석 등 문서만 변경 |
| refactor | 동작 변경 없이 구조 개선 |
| style | 포매팅, 공백 등 동작에 영향 없는 변경 |
| perf | 동작은 같고 성능이 향상됨 |
| test | 테스트 코드 추가·수정 |
| chore | 빌드, 설정, 의존성 등 유지보수 |
| ci | CI/CD 파이프라인 변경 |

애매할 때:
- 기능 추가와 리팩토링이 섞임 → 핵심 의도가 기능이면 `feat`, 구조 개선이면 `refactor`
- 버그 수정 중 코드 정리도 함 → `fix`
- 코드와 README를 함께 수정 → 코드 기준 type (`docs`는 문서만 바뀔 때)

### Scope

- 변경 파일들의 공통 영역(디렉토리, 모듈, 기능명)에서 소문자 한 단어로 고른다.
- 여러 영역에 걸치면 가장 핵심적인 영역 하나를 고른다.
- 프로젝트 전체에 걸친 변경이면 생략한다.
- `git log --oneline -20`에서 이 프로젝트가 이미 쓰는 scope가 있으면 그것을 따른다. scope가 일관되면 `git log --grep "(auth)"`처럼 영역별 이력을 찾기 쉽다.

### 자주 쓰는 끝맺음

추가, 삭제, 수정, 변경, 개선, 구현, 분리, 이동, 이름 변경, 단순화, 교체, 전환, 설정, 정리

---

## Why 섹션

변경의 **동기**를 쓴다. 미래의 독자가 "이 코드를 되돌려도 되나?"를 판단할 근거가 된다.

- 문제 상황, 요구사항, 제약, 대안을 고르지 않은 이유 등을 쓴다.
- diff를 다시 서술하지 않는다. "함수 A를 수정함"은 Why가 아니라 What이다.
- 사용자와 대화에서 나온 결정 배경이 있으면 여기에 남긴다. 대화는 사라지지만 커밋은 남는다.

좋은 예:
```
Why:
- 네트워크 타임아웃 후 재시도하면 같은 주문이 두 번 생성됨
- 결제 게이트웨이가 idempotency key를 지원해서 서버 측 중복 검사보다 단순함
```

나쁜 예:
```
Why:
- order.go 수정
```

## What 섹션

변경 내용을 **의미 단위**로 쓴다.

- 파일명 나열이 아니라 "무엇이 어떻게 달라졌는지"를 쓴다.
- 나중에 검색될 만한 **식별자**(함수명, 설정 키, 엔드포인트, 환경 변수)는 백틱 없이 원래 이름 그대로 적는다. `git log --grep`으로 찾을 수 있게 하기 위해서다.
- 동작 변화, 기본값 변화, 마이그레이션 필요 여부처럼 읽는 쪽이 놓치면 안 되는 내용을 우선한다.

---

## 트레일러

본문 맨 끝에 빈 줄 하나를 두고 `Key: value` 형식으로 쓴다. git이 트레일러로 인식하려면 다음을 지켜야 한다.

- 트레일러 블록은 메시지의 **마지막 문단**이어야 한다.
- 트레일러 사이에 빈 줄을 넣지 않는다.
- 키에는 공백 없이 하이픈을 쓴다(`Follow-up`, `Breaking-Change`).

이렇게 쓰면 `git log --format='%h %s%n%(trailers)'`나 `git log --grep '^Follow-up:'`로 바로 추출할 수 있다.

| 트레일러 | 언제 쓰나 | 예시 |
| --- | --- | --- |
| `Refs` | 관련 이슈·PR·문서가 있을 때. 사용자가 언급했거나 브랜치명에 이슈 번호가 있을 때만 쓰고 추측하지 않는다. | `Refs: #42` |
| `Breaking-Change` | API, 설정 형식, 기본 동작 등 기존 사용처가 깨질 때. 제목에도 `!`를 붙인다. | `Breaking-Change: config.yaml의 port 키가 server.port로 이동` |
| `Follow-up` | 이번 커밋에서 의도적으로 남긴 후속 작업이 있을 때. 여러 개면 줄마다 하나씩 쓴다. | `Follow-up: 카카오 로그인 추가` |
| `Co-Authored-By` | Claude가 작업에 참여한 커밋. 세션 지침에 지정된 서명 줄이 있으면 그대로 쓰고, 없으면 `Co-Authored-By: Claude <noreply@anthropic.com>`을 쓴다. | `Co-Authored-By: Claude <noreply@anthropic.com>` |

`Follow-up`은 다음 세션의 Claude가 "이어서 할 일"을 찾는 단서다. 작업 중 미룬 일, 알려진 한계, 임시 처리(workaround)가 있으면 빠뜨리지 않는다.

---

## 실행 절차

### 1단계: 상태 파악

다음 명령을 병렬로 실행한다.

```bash
git status                 # -uall 옵션은 쓰지 않는다
git diff --cached
git log --oneline -20      # 기존 scope와 메시지 스타일 참고
git branch --show-current  # 브랜치명에 이슈 번호가 있는지 확인
```

### 2단계: staged 변경 확인

**staged 파일이 없으면:**
- "staged된 파일이 없습니다"라고 알리고 unstaged 변경 파일 목록을 보여 준다.
- 어떤 파일을 staging할지 사용자에게 묻는다.
- 사용자가 지정한 파일만 `git add <파일>`로 staging한다. `git add .`, `git add -A`는 쓰지 않는다. 의도하지 않은 파일이 섞이는 것을 막기 위해서다.

**민감 파일이 staged되어 있으면:**
- `.env`, `.env.*`, `credentials.json`, `*secret*`, `*.pem`, `*.key`, `id_rsa*` 등이 보이면 경고하고 사용자 확인 없이 커밋하지 않는다.

### 3단계: 분할 여부 판단

staged 변경을 의도별로 묶어 본다. 의도가 둘 이상이면(예: 기능 추가 + 무관한 문서 수정, 버그 수정 + 무관한 리팩토링) 커밋을 나누자고 제안한다. 의도별로 나눠 두면 되돌리기, `git bisect`, 이력 검색이 쉬워진다.

제안 형식:
```
staged 변경에 의도가 다른 작업이 섞여 있어 나눠서 커밋하는 것을 제안드립니다.

1. feat(auth): 구글 OAuth 로그인 추가
   - src/auth/google.go, src/auth/handler.go
2. docs(readme): OAuth 설정 가이드 추가
   - README.md

나눠서 커밋할까요, 하나로 커밋할까요?
```

- 사용자가 동의하면 **파일 단위**로 나눈다. `git restore --staged <파일>`로 staging을 풀고 그룹별로 `git add <파일>` → 커밋을 반복한다.
- 한 파일 안에 여러 의도가 섞여 있으면 그 파일은 핵심 의도 쪽 커밋에 함께 넣는다. hunk 단위 분할(`git add -p`)은 대화형이라 쓰지 않는다.
- 사용자가 하나로 하자고 하면 핵심 의도 기준으로 한 커밋을 만들고, 나머지 의도는 What에 적는다.
- 의도가 하나면 묻지 않고 바로 4단계로 간다.

### 4단계: 메시지 작성

1. type과 scope를 정한다.
2. 한국어 제목을 쓴다.
3. 이유가 diff로 드러나지 않으면 Why를 쓴다. 이번 대화에서 사용자가 말한 배경이 있으면 반영한다.
4. 필요하면 What을 쓴다.
5. 해당하는 트레일러를 붙인다.

### 5단계: 커밋 실행

메시지는 HEREDOC으로 전달한다. 따옴표나 줄바꿈이 깨지지 않게 하기 위해서다.

```bash
git commit -m "$(cat <<'EOF'
fix(api): 재시도 시 주문이 중복 생성되는 문제 수정

Why:
- 네트워크 타임아웃 후 재시도하면 같은 주문이 두 번 생성됨

What:
- CreateOrder에 Idempotency-Key 헤더 검증 추가
- 같은 키로 24시간 안에 들어온 요청은 기존 주문을 반환

Follow-up: 만료된 idempotency key 정리 배치 작업 추가
Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 6단계: 결과 보고

`git status`와 `git log -1 --format='%h %s'`로 확인한 뒤 다음을 알린다.

- 커밋 해시와 제목 (분할했다면 커밋마다)
- 남은 unstaged 변경 유무
- 남긴 `Follow-up`이 있으면 그 내용

---

## 예시

**기능 추가**
```
feat(auth): 구글 OAuth 로그인 추가

Why:
- 이메일 가입 단계에서 이탈이 많아 소셜 로그인이 필요함

What:
- Google OAuth 2.0 로그인 추가
- 같은 이메일의 기존 계정이 있으면 자동으로 연결

Refs: #42
Follow-up: 카카오 로그인 추가
Co-Authored-By: Claude <noreply@anthropic.com>
```

**호환성을 깨는 변경**
```
refactor(config)!: port 설정을 server 섹션 아래로 이동

Why:
- 서버 관련 설정이 최상위에 흩어져 있어 설정 파일 구조를 정리함

What:
- port 키를 server.port로 이동
- 이전 port 키는 더 이상 읽지 않음

Breaking-Change: 기존 config.yaml의 port 키를 server.port로 옮겨야 함
```

**What만 있는 변경**
```
chore: Go 의존성 업데이트

What:
- golang.org/x/net v0.20.0 → v0.25.0
- go.sum 갱신
```

**제목만 있는 변경**
```
style(api): gofmt 적용
```

---

## 주의사항

### 지켜야 할 것
- staged 파일만 커밋한다. 분할 과정 외에는 staging 상태를 바꾸지 않는다.
- pre-commit hook이 실패하면 원인을 고친 뒤 **새 커밋**을 만든다. 실패한 커밋은 만들어지지 않았으므로 `--amend`하면 이전 커밋이 바뀐다.
- 메시지는 HEREDOC으로 전달한다.

### 하지 말 것 (사용자가 명시적으로 요청한 경우 제외)
- `git add .`, `git add -A`로 전체 staging
- `--amend`로 이전 커밋 수정
- `--no-verify`, `--no-gpg-sign` 등 안전 장치 우회
- `git push`

### 어떤 경우에도 하지 말 것
- `git push --force`
