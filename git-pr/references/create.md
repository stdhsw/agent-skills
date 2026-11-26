# PR 생성

## 1. 상태 점검

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

## 2. 커밋 분석

```bash
git log BASE..HEAD --format='%h %s%n%b'
git diff BASE..HEAD --stat
git log BASE..HEAD --format='%(trailers:key=Follow-up,valueonly)'        # 남은 작업
git log BASE..HEAD --format='%(trailers:key=Breaking-Change,valueonly)'  # 호환성 깨짐
```

변경이 크면 `git diff BASE..HEAD`도 읽는다. 커밋이 `git-commit` 형식이면 각 커밋의 `Why:`가 PR 요약의 좋은 재료다.

## 3. PR 제목

형식: `type(scope): 한국어 설명`

- 50자 이내, 명사형으로 끝맺고 마침표를 찍지 않는다.
- 여러 type이 섞이면 비중이 가장 큰 것을 고른다. 새 기능이 중심이면 `feat`, 버그 수정이면 `fix`, 설정·의존성이면 `chore`.
- scope는 변경이 한 영역에 모여 있을 때만 쓰고, 여러 영역에 걸치면 생략한다.
- 커밋이 하나면 그 커밋 제목을 그대로 쓴다.
- 커밋 중 하나라도 `!:`나 `Breaking-Change` 트레일러가 있으면 PR 제목에도 `!`를 붙인다.

## 4. PR 본문

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

## 5. push

원격에 없는 커밋이 있으면 push한다(`SKILL.md`의 "승인" 기준을 따른다).

```bash
git push -u origin <현재 브랜치>
```

이미 모두 push되어 있으면 건너뛴다.

## 6. PR 생성

본문은 HEREDOC으로 전달해 줄바꿈이 깨지지 않게 한다.

```bash
gh pr create --base BASE --head <현재 브랜치> --title "<제목>" --body "$(cat <<'EOF'
## 요약
...
EOF
)"
```

이미 있는 PR의 제목·본문만 바꿀 때는 `gh pr edit <번호> --title "..." --body "..."`를 쓴다.

## 7. 결과 보고

- PR URL과 제목
- 본문에 넣은 섹션과, 트레일러가 없어 뺀 섹션
- 테스트 항목 중 아직 확인하지 않은 것
- CI 상태 (없으면 없다고)
