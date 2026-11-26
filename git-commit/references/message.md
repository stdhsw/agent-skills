# 커밋 메시지 작성 규칙

`SKILL.md`의 "메시지 형식"을 실제로 채울 때 보는 상세 규칙과 예시다.

## 목차

- [제목](#제목): type, scope, 끝맺음
- [Why 섹션](#why-섹션)
- [What 섹션](#what-섹션)
- [트레일러](#트레일러)
- [예시](#예시)

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
- 코드 식별자, 파일명, 명령어는 원래 이름 그대로 쓴다: `feat(vscode-global): cmd+alt+h로 여는 전역 도움말 추가`
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

**버그 수정**
```
fix(api): 재시도 시 주문이 중복 생성되는 문제 수정

Why:
- 네트워크 타임아웃 후 재시도하면 같은 주문이 두 번 생성됨

What:
- CreateOrder에 Idempotency-Key 헤더 검증 추가
- 같은 키로 24시간 안에 들어온 요청은 기존 주문을 반환

Follow-up: 만료된 idempotency key 정리 배치 작업 추가
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
