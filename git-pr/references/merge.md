# 머지

## 1. PR 확인

```bash
gh pr view --json number,title,headRefName,baseRefName,mergeable,statusCheckRollup,url
```

- PR이 없으면 먼저 만들어야 한다고 알리고, PR 생성을 제안한다.
- 여러 PR 중 고를 수 있으면 어느 PR을 머지할지 묻는다.

## 2. 머지 가능 여부

| 상태 | 처리 |
| --- | --- |
| `mergeable`이 `CONFLICTING` | 충돌이 있어 머지할 수 없다고 알리고 끝낸다. 기본 브랜치를 끌어와 충돌을 해결해야 한다고 안내한다 |
| CI 체크 중 `FAILURE`가 있음 | 실패한 체크 이름을 알리고 그래도 머지할지 묻는다. 기본은 보류 |
| CI가 진행 중 | 알리고 기다릴지 진행할지 묻는다 |
| CI 체크가 하나도 없음 | "이 저장소에는 CI가 없습니다"라고 알리고 다음 단계로 간다 |

## 3. 머지 방식 분석

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

분석 결과는 근거와 함께 보여 준다. 세 방식을 모두 선택지로 두고 추천하는 쪽에 `(추천)`을 붙인다.

```
커밋 분석 결과:
- 커밋 수: 8개 (Conventional Commits 형식 8개)
- WIP/fixup 류: 0개
- 머지 커밋: 없음

추천: C) 리베이스 머지
근거:
- 모든 커밋이 기능별로 나뉘어 있고 제목 형식이 일정함
- 각 커밋의 Why/What을 기본 브랜치에 그대로 남길 가치가 있음

A) 일반 머지
B) 스쿼시 머지
C) 리베이스 머지 (추천)
D) 취소
```

요청에서 방식을 이미 정했다면("스쿼시로 머지해줘") 분석 결과만 짧게 알리고 그 방식으로 진행한다.

## 4. 머지 실행

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
- 머지 후 원격 기본 브랜치의 최신 커밋을 조회해 메시지가 의도대로 들어갔는지 확인한다.

## 5. 머지 후

1. 머지된 커밋 SHA와 PR URL을 알린다.
2. 로컬 기본 브랜치를 최신으로 맞출지 묻는다. 동의하면 다음을 실행한다.

```bash
git switch BASE && git pull --ff-only
```

`--ff-only`를 쓰는 이유는 로컬 기본 브랜치에 의도하지 않은 머지 커밋이 생기는 것을 막기 위해서다. 실패하면 상황을 알리고 멈춘다.
