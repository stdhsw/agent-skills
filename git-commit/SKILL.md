---
name: git-commit
description: >
  staged 변경사항을 분석해 한국어 커밋 메시지로 커밋하는 스킬. 제목은 Conventional Commits 형식
  (type(scope): 한국어 설명), 본문은 Why/What, 끝에는 git 트레일러(Refs, Breaking-Change, Follow-up,
  Co-Authored-By)를 붙여 나중에 git log만 보고도 변경 이유와 후속 작업을 알 수 있게 한다.
  의도가 다른 변경이 섞여 있으면 커밋을 나누자고 먼저 제안한다.
  "커밋해줘", "커밋 진행", "commit", "변경사항 커밋", "지금까지 작업한거 커밋", "커밋 나눠서 해줘",
  "커밋 메시지 써줘", "변경사항 저장" 등을 요청할 때 사용하세요.
  코드를 고친 뒤 결과를 저장하거나 반영하려는 맥락이면 "커밋"이라는 말이 없어도 사용하세요.
  PR 생성과 머지는 git-pr 스킬이 맡는다.
---

# Git Commit

staged된 변경사항만 분석해 구조화된 한국어 커밋 메시지로 커밋하는 스킬이다.

커밋 메시지는 사람만 읽는 기록이 아니다. 다음 세션의 Claude Code가 `git log`, `git blame`, `git log --grep`으로 코드의 맥락을 되짚을 때 가장 먼저 읽는 자료다. diff는 **무엇이** 바뀌었는지 보여 주지만 **왜** 바꿨는지는 커밋 메시지에만 남는다. 그래서 이유(Why)와 변경 내용(What)을 나누고, 후속 작업 같은 메타데이터를 git 표준 트레일러로 남긴다.

## 한눈에 보기

| 항목 | 규칙 | 상세 |
| --- | --- | --- |
| 제목 | `type(scope): 한국어 설명`, 50자 이내, 명사형 끝맺음 | `references/message.md` "제목" |
| 본문 | `Why:`(변경 이유), `What:`(변경 내용), 필요할 때만 | `references/message.md` "Why 섹션", "What 섹션" |
| 트레일러 | `Refs`, `Breaking-Change`, `Follow-up`, `Co-Authored-By`, 해당할 때만 | `references/message.md` "트레일러" |
| 범위 | staged 파일만 커밋 | 아래 "작업 흐름" |

메시지를 처음 쓰거나 type·scope 고르기가 애매하면 `references/message.md`를 읽는다. 예시도 거기에 있다.

## 메시지 형식

```
type(scope): 한국어 제목

Why:
- 변경 이유

What:
- 변경 내용

Refs: #42
Follow-up: 후속 작업
Co-Authored-By: Claude <noreply@anthropic.com>
```

- 오타 수정, 포매팅, 단일 설정값 변경처럼 제목만으로 충분하면 본문을 생략한다.
- 이유가 자명하면(예: 의존성 버전 올림) `Why:`를 생략하고 `What:`만 쓴다.
- 빈 섹션을 형식상 남기지 않는다. 내용 없는 레이블은 읽는 쪽에 잡음일 뿐이다.

## 작업 흐름

### 1. 상태 파악

다음을 병렬로 실행한다.

```bash
git status                 # -uall 옵션은 쓰지 않는다
git diff --cached
git log --oneline -20      # 기존 scope와 메시지 스타일 참고
git branch --show-current  # 브랜치명에 이슈 번호가 있는지 확인
```

### 2. staged 변경 확인

- **staged 파일이 없으면**: 그 사실과 unstaged 변경 파일 목록을 보여 주고, 무엇을 staging할지 묻는다. 사용자가 지정한 파일만 `git add <파일>`로 올린다. `git add .`나 `git add -A`는 의도하지 않은 파일이 섞이므로 쓰지 않는다.
- **민감 파일이 있으면**: `.env`, `.env.*`, `credentials.json`, `*secret*`, `*.pem`, `*.key`, `id_rsa*` 등이 staged되어 있으면 경고하고, 사용자 확인 없이 커밋하지 않는다.

### 3. 분할 여부 판단

staged 변경을 의도별로 묶어 본다. 의도가 둘 이상이면(예: 기능 추가 + 무관한 문서 수정) 나누자고 제안한다. 의도별로 나눠 두면 되돌리기, `git bisect`, 이력 검색이 쉬워진다.

```
staged 변경에 의도가 다른 작업이 섞여 있어 나눠서 커밋하는 것을 제안드립니다.

1. feat(auth): 구글 OAuth 로그인 추가
   - src/auth/google.go, src/auth/handler.go
2. docs(readme): OAuth 설정 가이드 추가
   - README.md

나눠서 커밋할까요, 하나로 커밋할까요?
```

- 동의하면 **파일 단위**로 나눈다. `git restore --staged <파일>`로 내린 뒤 그룹별로 `git add <파일>` → 커밋을 반복한다.
- 한 파일에 여러 의도가 섞여 있으면 핵심 의도 쪽 커밋에 넣는다. `git add -p`는 대화형이라 쓰지 않는다.
- 하나로 하자고 하면 핵심 의도 기준으로 한 커밋을 만들고 나머지는 What에 적는다.
- 요청에 이미 "나눠서 커밋해줘"가 있거나 의도가 하나면 묻지 않고 진행한다.

### 4. 메시지 작성

1. type과 scope를 정한다.
2. 한국어 제목을 쓴다.
3. 이유가 diff로 드러나지 않으면 Why를 쓴다. 이번 대화에서 사용자가 말한 결정 배경이 있으면 반영한다. 대화는 사라지지만 커밋은 남는다.
4. 필요하면 What을 쓴다.
5. 해당하는 트레일러를 붙인다. 미룬 일, 알려진 한계, 임시 처리가 있으면 `Follow-up`을 빠뜨리지 않는다.

### 5. 커밋 실행

따옴표나 줄바꿈이 깨지지 않게 HEREDOC으로 전달한다.

```bash
git commit -m "$(cat <<'EOF'
<메시지>
EOF
)"
```

pre-commit hook이 실패하면 원인을 고친 뒤 **새 커밋**을 만든다. 실패한 커밋은 만들어지지 않았으므로 `--amend`하면 이전 커밋이 바뀐다.

## 결과 보고

`git status`와 `git log -1 --format='%h %s'`로 확인한 뒤 알린다.

- 커밋 해시와 제목 (분할했다면 커밋마다)
- 남은 unstaged 변경 유무
- 남긴 `Follow-up`이 있으면 그 내용

## 하지 않는 일

사용자가 명시적으로 요청한 경우가 아니면 하지 않는다.

- `git add .`, `git add -A`로 전체 staging
- 분할 과정 외에 staging 상태 바꾸기
- `--amend`로 이전 커밋 수정
- `--no-verify`, `--no-gpg-sign` 등 안전 장치 우회
- `git push`

어떤 경우에도 `git push --force`는 하지 않는다.
