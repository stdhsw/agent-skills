# gh CLI 준비

`gh --version`이나 `gh auth status`가 실패할 때 보는 문서다.

## 설치되어 있지 않을 때

OS에 맞는 설치 명령을 보여 주고 사용자가 직접 설치하도록 기다린다. 패키지 설치는 시스템을 바꾸는 일이라 직접 실행하지 않는다.

| OS | 명령 |
| --- | --- |
| macOS | `brew install gh` |
| Debian/Ubuntu | `sudo apt install gh` (공식 저장소 등록이 필요하면 https://github.com/cli/cli#installation 안내) |
| Fedora/RHEL | `sudo dnf install gh` |
| Arch | `sudo pacman -S github-cli` |
| Windows | `winget install --id GitHub.cli` |

## 로그인되어 있지 않을 때

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

로그인을 기다리는 동안 git만으로 할 수 있는 일(예: push)은 먼저 해 두어도 된다. 요청에 push 승인이 있는 경우에 한한다.
