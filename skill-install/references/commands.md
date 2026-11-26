# 명령 모음

`SKILL.md`의 작업 흐름에서 쓰는 명령이다. `<이름>`은 스킬 이름으로 바꾼다.

## 변수

```bash
REPO=$(git -C "$(cd -P "<이 스킬 폴더>" && pwd)" rev-parse --show-toplevel)
DEST="$HOME/.claude/skills"
BACKUP="$HOME/.claude/skills-backup"
```

이 스킬은 링크를 거쳐 실행될 수 있으므로 `cd -P`로 실제 경로를 구한 뒤 저장소를 찾는다.

## 상태 확인

```bash
for f in $(find "$REPO" -mindepth 2 -maxdepth 2 -name SKILL.md -not -path '*/.*' | sort); do
  name=$(basename "$(dirname "$f")"); p="$DEST/$name"
  if [ -L "$p" ]; then
    t=$(readlink "$p")
    [ "$t" = "$REPO/$name" ] && s="설치됨" || s="다른 링크 → $t"
  elif [ -e "$p" ]; then s="충돌 (실제 폴더)"
  else s="미설치"; fi
  echo "$name | $s"
done
# 끊어진 링크: REPO를 가리키지만 대상이 없는 링크
# -H: DEST 자체가 링크(예: ~/.claude/skills → ~/.config/claude/skills)여도 그 안을 검사한다
find -H "$DEST" -maxdepth 1 -type l | while read -r l; do
  t=$(readlink "$l"); case "$t" in "$REPO"/*) [ -e "$l" ] || echo "끊어진 링크: $l → $t";; esac
done
```

`find`는 기본적으로 시작 경로의 링크를 따라가지 않는다. `DEST`가 다른 폴더를 가리키는 링크이면 `-H` 없이는 안을 검사하지 못해 끊어진 링크를 놓친다.

## 검사

```bash
f="$REPO/<이름>/SKILL.md"
head -1 "$f"                                                        # --- 이어야 함
awk 'NR>1&&/^---$/{print "닫힘:",NR;exit}' "$f"                      # frontmatter가 닫히는 줄
awk 'NR>1&&/^---$/{exit}/^name:/{print $2}' "$f"                     # 폴더 이름과 같아야 함
awk 'NR>1&&/^---$/{exit}/^description:/{print "description 있음"}' "$f"
```

## 설치

링크는 절대 경로로 만든다. 상대 경로 링크는 위치가 바뀌면 끊어진다.

```bash
mkdir -p "$DEST"
ln -s "$REPO/<이름>" "$DEST/<이름>"
```

## 교체 (백업 후)

```bash
mkdir -p "$BACKUP"
mv "$DEST/<이름>" "$BACKUP/<이름>-$(date +%Y%m%d-%H%M%S)"
ln -s "$REPO/<이름>" "$DEST/<이름>"
```

## 지우기 (링크 제거, 백업 없는 교체, 끊어진 링크 정리)

삭제 명령은 **변수 대신 절대 경로를 그대로 적는다.** `rm "$DEST/$n"`처럼 변수로 경로를 만들면 Claude Code 안전 검사가 "변수가 비면 루트를 지울 수 있다"며 막는다. 지우기 전에 `ls -l`로 대상이 맞는지 보여 준다.

```bash
ls -l /Users/<사용자>/.claude/skills/<이름>
rm /Users/<사용자>/.claude/skills/<이름>
```

- 링크는 `rm`만 쓴다. **끝에 `/`를 붙이거나 `-r`을 쓰지 않는다.** `rm -r <링크>/`는 링크가 아니라 저장소의 원본 스킬 내용을 지운다.
- 실제 폴더를 백업 없이 지울 때(`rm -rf`)도 절대 경로를 적는다.

## 백업 되돌리기

```bash
ls "$BACKUP" | grep '^<이름>-'
mv "$BACKUP/<이름>-<시각>" "$DEST/<이름>"
```
