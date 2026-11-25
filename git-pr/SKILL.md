---
name: git-pr
description: >
  현재 브랜치의 커밋을 분석해 GitHub Pull Request를 만들고, 머지까지 진행하는 스킬.
  PR 제목은 Conventional Commits 형식의 한국어(type(scope): 한국어 설명), 본문은 한국어 템플릿으로 작성하고,
  커밋의 Follow-up·Breaking-Change 트레일러를 PR 본문에 모은다.
  머지할 때는 충돌과 CI 상태를 확인하고 커밋 이력을 분석해 일반 머지 / 스쿼시 머지 / 리베이스 머지 중 하나를 추천한 뒤
  사용자가 고른 방식으로 머지하며, 머지 커밋 메시지도 한국어로 쓴다. gh CLI가 없거나 로그인되지 않았으면 설치·로그인 방법을 단계별로 안내한다.
  사용자가 "PR 만들어줘", "PR 생성하자", "PR 해줘", "PR 올려줘", "pull request 만들어줘", "PR, push 해줘",
  "지금까지 작업한거 PR", "머지해줘", "머지하자", "merge해줘", "PR 머지", "PR 합치자", "브랜치 머지",
  "PR 제목 바꿔줘" 등 GitHub PR 생성·수정·머지를 요청할 때 이 스킬을 사용하세요.
  PR이나 머지라는 말이 나오고 작업 결과를 GitHub에 반영하려는 의도가 보이면 적극적으로 트리거하세요.
  개인 저장소(혼자 작업하는 환경)를 가정한다. 단, 커밋 작성은 git-commit 스킬이 담당한다.
---

# Git PR

현재 브랜치의 작업을 GitHub PR로 올리고 머지까지 마무리하는 스킬이다. 혼자 쓰는 저장소에서 빠르게 PR을 만들고, 커밋 이력에 맞는 머지 방식을 고르도록 돕는다.

| 항목 | 규칙 |
| --- | --- |
| PR 제목 | `type(scope): 한국어 설명` (type, scope만 영어) |
| PR 본문 | 한국어, 아래 템플릿 |
| 머지 커밋 메시지 | 한국어 |
| base 브랜치 | 저장소 기본 브랜치 (보통 `main`) |
| 머지 방식 | 커밋 분석으로 추천 → 사용자 선택 |
| 브랜치 삭제 | 머지 후에도 로컬·원격 브랜치를 지우지 않음 |

메시지를 한국어로 쓰는 이유는 `git-commit` 스킬과 같은 기준으로 커밋, PR, 머지 커밋을 한 언어로 맞추기 위해서다. `type`과 `scope`는 Conventional Commits 도구(changelog, 버전 판단)가 읽으므로 영어로 둔다.

---

## 1. 사전 준비: gh CLI

작업 전에 `gh`가 설치되어 있고 로그인되어 있는지 확인한다.

```bash
gh --version
gh auth status
```

### 1.1 설치되어 있지 않을 때

OS에 맞는 설치 명령을 보여 주고 사용자가 직접 설치하도록 기다린다.

| OS | 명령 |
| --- | --- |
| macOS | `brew install gh` |
| Debian/Ubuntu | `sudo apt install gh` (공식 저장소 등록이 필요하면 https://github.com/cli/cli#installation 안내) |
| Fedora/RHEL | `sudo dnf install gh` |
| Arch | `sudo pacman -S github-cli` |
| Windows | `winget install --id GitHub.cli` |

### 1.2 로그인되어 있지 않을 때

`gh auth login`은 브라우저 인증을 거치는 대화형 명령이라 직접 실행하지 않는다. 사용자가 막히지 않도록 질문마다 무엇을 고를지 알려 준다.

먼저 원격 주소 형식을 확인해 Git 프로토콜 답을 정한다.

```bash
git remote get-url origin   # git@github.com:... → SSH, https://github.com/... → HTTPS
```

그다음 아래 순서로 안내한다.

1. `gh auth login` 실행
2. `Where do you use GitHub?` → **GitHub.com**
3. `How would you like to authenticate GitHub CLI?` → **Login with a web browser**
4. `What is your preferred protocol for Git operations on this host?` → 위에서 확인한 **SSH** 또는 **HTTPS** (기본값이 HTTPS이므로 SSH면 ↓를 한 번 누른다)
5. SSH 키를 올릴지 물으면 이미 SSH로 push가 되는 경우 **Skip**, 아니면 기존 공개 키 선택
6. 터미널에 나온 일회용 코드를 복사 → Enter로 브라우저 열기 → 코드 입력 → **Authorize**
7. `gh auth status`에 `Logged in to github.com account <이름>`이 나오면 완료

사용자가 완료를 알리면 `gh auth status`로 다시 확인하고 하던 작업을 이어 간다.

---

## 2. PR 생성

### 2.1 상태 점검

다음을 병렬로 확인한다.

```bash
git rev-parse --abbrev-ref HEAD                                   # 현재 브랜치
git status --short                                                # 미커밋 변경
gh repo view --json defaultBranchRef -q .defaultBranchRef.name    # 기본 브랜치 (BASE)
git log BASE..HEAD --oneline                                      # 올릴 커밋
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null  # 원격 추적 여부
gh pr view --json number,title,url 2>/dev/null                    # 이미 열린 PR
```

| 상황 | 처리 |
| --- | --- |
| 현재 브랜치가 기본 브랜치 | PR을 만들 수 없다고 알리고, 작업 브랜치를 만들어야 한다고 안내한다 |
| 미커밋 변경이 있음 | 알리고 처리 방법을 묻는다(커밋 후 진행 / 무시하고 진행 / 중단). 임의로 커밋하거나 stash하지 않는다. 커밋이 필요하면 `git-commit` 스킬을 권한다 |
| `BASE..HEAD` 커밋이 0개 | 올릴 내용이 없다고 알리고 끝낸다 |
| 이미 열린 PR이 있음 | 새로 만들지, 기존 PR에 push만 할지(필요하면 제목·본문 갱신) 묻는다 |

### 2.2 커밋 분석

```bash
git log BASE..HEAD --format='%h %s%n%b'
git diff BASE..HEAD --stat
git log BASE..HEAD --format='%(trailers:key=Follow-up,valueonly)'        # 남은 작업
git log BASE..HEAD --format='%(trailers:key=Breaking-Change,valueonly)'  # 호환성 깨짐
```

변경이 크면 `git diff BASE..HEAD`도 읽는다. 커밋이 `git-commit` 형식이면 각 커밋의 `Why:`가 PR 요약의 좋은 재료다.

### 2.3 PR 제목

형식: `type(scope): 한국어 설명`

- 50자 이내, 명사형으로 끝맺고 마침표를 찍지 않는다.
- 여러 type이 섞이면 비중이 가장 큰 것을 고른다. 새 기능이 중심이면 `feat`, 버그 수정이면 `fix`, 설정·의존성이면 `chore`.
- scope는 변경이 한 영역에 모여 있을 때만 쓰고, 여러 영역에 걸치면 생략한다.
- 커밋이 하나면 그 커밋 제목을 그대로 쓴다.
- 커밋 중 하나라도 `!:`나 `Breaking-Change` 트레일러가 있으면 PR 제목에도 `!`를 붙인다.

### 2.4 PR 본문

```markdown
## 요약

<왜 이 변경이 필요한지, 어떤 가치를 주는지 1~3줄>

## 변경 내용

- <영역/주제별로 묶은 주요 변경>

## ⚠️ 호환성 변경

- <Breaking-Change 트레일러 내용>

## 남은 작업

- <Follow-up 트레일러 내용>

## 테스트

- [x] <실제로 확인한 동작>
- [ ] <아직 확인하지 않은 동작>
```

- **요약**은 "무엇을 했는지"보다 "왜 필요했는지"에 가깝게 쓰고, 변경 내용과 겹치지 않게 한다.
- **변경 내용**은 커밋 단위가 아니라 영역이나 주제별로 묶는다.
- **호환성 변경**과 **남은 작업**은 해당 트레일러가 있을 때만 넣는다. 빈 섹션은 남기지 않는다.
- **테스트**는 이번 작업에서 실제로 확인한 것만 `[x]`로 체크한다. 확인하지 않은 것은 `[ ]`로 두고 사용자에게 알린다. 확인하지 않은 것을 체크하면 PR을 읽는 사람이 잘못 믿게 된다.
- 세션 지침에 PR 본문 끝에 붙일 서명 줄이 있으면 본문 마지막에 붙인다.

### 2.5 승인

요청에 진행 승인이 이미 들어 있는지에 따라 다르게 한다. 같은 확인을 두 번 묻지 않기 위해서다.

| 요청 | 처리 |
| --- | --- |
| "PR, push 해줘", "바로 PR 올려줘"처럼 생성·push를 명시적으로 요청 | 초안을 보여 주고 바로 push·생성한다 |
| "PR 만들자", "PR 초안 써줘"처럼 확인 여지가 있는 요청 | 초안을 보여 주고 A) 이대로 생성 / B) 수정 / C) 취소 중에서 고르게 한다 |

### 2.6 push

원격에 없는 커밋이 있으면 push한다(2.5에서 승인을 받은 경우).

```bash
git push -u origin <현재 브랜치>
```

이미 모두 push되어 있으면 건너뛴다. 강제 push는 하지 않는다.

### 2.7 PR 생성

본문은 HEREDOC으로 전달해 줄바꿈이 깨지지 않게 한다.

```bash
gh pr create --base BASE --head <현재 브랜치> --title "<제목>" --body "$(cat <<'EOF'
## 요약
...
EOF
)"
```

만든 뒤 PR URL과 CI 상태를 알린다. 이미 있는 PR의 제목·본문만 바꿀 때는 `gh pr edit <번호> --title "..." --body "..."`를 쓴다.

---

## 3. 머지

### 3.1 PR 확인

```bash
gh pr view --json number,title,headRefName,baseRefName,mergeable,statusCheckRollup,url
```

- PR이 없으면 먼저 만들어야 한다고 알리고, PR 생성을 제안한다.
- 여러 PR 중 고를 수 있으면 어느 PR을 머지할지 묻는다.

### 3.2 머지 가능 여부

| 상태 | 처리 |
| --- | --- |
| `mergeable`이 `CONFLICTING` | 충돌이 있어 머지할 수 없다고 알리고 끝낸다. 기본 브랜치를 끌어와 충돌을 해결해야 한다고 안내한다 |
| CI 체크 중 `FAILURE`가 있음 | 실패한 체크 이름을 알리고 그래도 머지할지 묻는다. 기본은 보류 |
| CI가 진행 중 | 알리고 기다릴지 진행할지 묻는다 |
| CI 체크가 하나도 없음 | "이 저장소에는 CI가 없습니다"라고 알리고 다음 단계로 간다 |

### 3.3 머지 방식 분석

```bash
git log BASE..HEAD --format='%h %s'
git log BASE..HEAD --merges --format='%h %s'
git log BASE..HEAD --format='%s' | grep -ciE '^(wip|fixup|squash|temp|tmp|typo|oops)|^(fixup|squash)!'
git log BASE..HEAD --format='%s' | grep -cE '^[a-z]+(\([a-z0-9-]+\))?!?: '   # Conventional Commits 형식 커밋 수
```

| 추천 | 조건 | 결과 |
| --- | --- | --- |
| **리베이스 머지** | 모든 커밋이 Conventional Commits 형식이고, WIP/fixup 류가 없고, 머지 커밋이 없음 | 커밋이 그대로 선형으로 들어간다. `git-commit`으로 정리한 브랜치의 기본 추천 |
| **일반 머지** | 커밋은 잘 정리되어 있지만 브랜치 안에 머지 커밋이 있거나, PR 단위로 묶어 보이는 게 중요할 때 | 머지 커밋이 생겨 PR 범위가 이력에 드러난다 |
| **스쿼시 머지** | WIP/fixup/typo 류 커밋이 하나라도 있거나, 형식이 제각각이거나, 사실상 하나의 작은 변경을 여러 번 나눠 커밋했을 때 | 커밋이 하나로 합쳐진다 |

분석 결과는 다음처럼 근거와 함께 보여 준다. 세 방식을 모두 선택지로 두고 추천하는 쪽에 `(추천)`을 붙인다.

```
커밋 분석 결과:
- 커밋 수: 8개 (Conventional Commits 형식 8개)
- WIP/fixup 류: 0개
- 머지 커밋: 없음

추천: C) 리베이스 머지
근거:
- 모든 커밋이 스킬별로 나뉘어 있고 제목 형식이 일정함
- 각 커밋의 Why/What을 기본 브랜치에 그대로 남길 가치가 있음

A) 일반 머지
B) 스쿼시 머지
C) 리베이스 머지 (추천)
D) 취소
```

사용자가 요청에서 방식을 이미 정했다면("스쿼시로 머지해줘") 분석 결과만 짧게 알리고 그 방식으로 진행한다.

### 3.4 머지 실행

머지 커밋 메시지는 한국어로 지정한다. 지정하지 않으면 GitHub가 영어 기본 메시지를 만든다.

```bash
# A) 일반 머지
gh pr merge <번호> --merge --subject "Merge pull request #<번호>: <PR 제목>" --body "<PR 요약 한두 줄>"

# B) 스쿼시 머지
gh pr merge <번호> --squash --subject "<PR 제목> (#<번호>)" --body "$(cat <<'EOF'
Why:
- ...

What:
- ...

Follow-up: ...
EOF
)"

# C) 리베이스 머지 (새 메시지가 생기지 않음)
gh pr merge <번호> --rebase
```

- 스쿼시 머지는 실행 전에 합쳐질 메시지를 보여 주고 고칠 부분이 있는지 묻는다. 본문은 `git-commit` 형식(Why/What, 트레일러)으로 각 커밋의 내용을 모아 쓴다. 트레일러가 사라지지 않게 하기 위해서다.
- `--delete-branch`는 쓰지 않는다.

### 3.5 머지 후

1. 머지된 커밋 SHA와 PR URL을 알린다.
2. 로컬 기본 브랜치를 최신으로 맞출지 묻는다. 동의하면 다음을 실행한다.

```bash
git switch BASE && git pull --ff-only
```

`--ff-only`를 쓰는 이유는 로컬 기본 브랜치에 의도하지 않은 머지 커밋이 생기는 것을 막기 위해서다. 실패하면 상황을 알리고 멈춘다.

---

## 4. 하지 않는 일

- 승인 없이 push, PR 생성, 머지 (요청에 명시적 승인이 있는 경우는 2.5 기준을 따른다)
- 강제 push (`--force`, `--force-with-lease`)
- `--delete-branch`로 브랜치 삭제
- 기본 브랜치에서 PR 생성 시도
- `gh auth login` 같은 대화형 인증 명령 자동 실행
- CI 실패를 알리지 않고 머지
- 미커밋 변경을 임의로 커밋하거나 stash
