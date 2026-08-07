# Spaceframe — 작업 규칙

`Spaceframe_v<버전>.html` 단일 파일로 된 전투함 설계·전투 게임. 빌드 단계가 없고,
브라우저에서 파일을 직접 열면 그대로 실행된다.

**파일 이름이 버전을 달고 있다.** 저장소 최상위에는 이 게임 파일이 항상 하나만
존재하며(현재 `Spaceframe_v1.31.27.html`), 정식 버전을 낼 때 파일 이름도 함께
새 버전으로 바꾼다. 작업을 시작할 때 실제 파일 이름을 먼저 확인할 것 —
이 문서에 적힌 이름이 최신이라는 보장은 없다.

## 검증

내부 로직은 IIFE 안에 있어 밖에서 직접 부를 수 없다. 검증은 페이지에 노출된
자체 테스트 하네스로만 가능하다.

```js
Object.keys(window).filter(k => k.startsWith('__SPACEFRAME') && k.endsWith('TEST__'))
```

각 하네스는 `{name, pass:boolean}` 형태의 단언을 담아 돌려주는데, 중첩 깊이가
제각각이다(`run.passed`, `recoveryThermal.passed` 등). **재귀적으로 훑어
`pass`가 boolean인 객체를 전부 세야** 한다. 최상위 `passed` 필드만 합산하면
수십 건을 놓친다.

Playwright로 `file://`을 열어 실행한다. 회귀 판정 기준:

- 단언 실패 0건
- pageerror 0건
- 총 단언 수가 줄지 않을 것

성능에 민감한 코드(매 틱 × 관측자 × 표적)를 건드렸다면 µs 단위로 실측한다.
계측 훅이 필요하면 원본이 아니라 **사본에 주입**한다.

## Git/GitHub 마무리 절차

목적은 클라우드 세션이나 컨테이너가 종료돼도 작업 결과가 유실되지 않게 하고,
각 정식 버전의 상태와 변경 이력을 GitHub에 지속적으로 보존하는 것이다.

### 작업 단위마다 (WIP)

의미 있는 작업 단위가 끝나면 commit하고 현재 작업 브랜치에 push한다. 작업이
여러 단계로 길어져도 **전체가 끝날 때까지 로컬 working tree에만 쌓아두지 않는다.**
안전하게 구분 가능한 단위마다 끊어서 올린다.

- **force push 금지**
- **기존 이력을 임의로 reset/rebase하거나 이미 존재하는 원격 커밋을 덮어쓰지 않는다**
- commit/push 전에 **branch, remote, working tree 상태를 확인**해 의도한 저장소와
  브랜치에 작업 중인지 확인한다
- WIP·중간 상태는 **commit/push까지만** 한다. 정식 버전으로 취급하거나 버전 tag를
  만들지 않는다

### 정식 버전이 완성됐을 때

구현과 최종 검증이 **모두** 끝났을 때만 아래를 수행한다.

1. `<title>`을 새 버전으로 올리고, **게임 파일 이름도 `Spaceframe_v<새 버전>.html`로
   함께 바꾼다**(`git mv`로 옮겨 이력을 잇는다). 저장소 최상위에 이전 버전 파일을
   남기지 않는다
2. 최종 변경사항 commit
3. 현재 branch push
4. 해당 버전의 Git tag 생성 (로컬)
5. ~~tag를 origin에 push~~ — **이 환경에서는 불가**(아래 "가능한 범위" 참조).
   시도하지 말고 대상 커밋 SHA를 보고에 실어 사용자가 웹에서 tag를 만들게 한다
6. `CHANGELOG.md` 갱신

버전 번호는 게임 파일의 `<title>` 프로토타입 번호를 따른다. 빌드 번호가
작은 폭으로 증가하는 체계다(1.31.17 → .18 → .21 → .23 → .24 → .25 → .26 → .27). 정식 버전을
낼 때는 title을 올리고, **이번 작업에서 실제로 손댄 테스트 하네스만** 새 버전으로
재스탬프한다(각 하네스의 `version:` 필드는 마지막으로 손댄 시점을 뜻한다).
tag 이름은 `v<버전>` 형식을 쓴다.

### tag 이름 충돌

동일한 이름의 tag가 이미 존재하면 **삭제하거나 덮어쓰지 말고 먼저 보고**한다.

### GitHub Release — 직접 생성하지 않는다

**이 환경에서는 Release를 만들 수 없는 것으로 확인됐다. 생성이나 업로드를 반복
시도하지 않는다.** 대신 정식 버전마다 사용자가 GitHub 웹에서 직접 만들 수 있도록
**Release Notes 초안을 최종 보고에 포함**한다. 초안에는 최소한 다음을 담는다.

- 버전명
- 주요 변경사항
- 수정된 문제
- 새 기능
- 중요한 내부 변경사항
- 테스트/검증 결과
- 알려진 문제

Release에 첨부할 최종 실행용 HTML 파일의 **정확한 파일명**도 함께 알려준다.
저장소의 게임 파일 이름이 곧 그 이름이다(`Spaceframe_v<버전>.html`).

### 문서

- `README.md`는 **매 버전마다 고치지 않는다.** 프로젝트 자체의 설명·사용법·구조가
  실제로 바뀐 경우에만 갱신한다.
- 버전별 변경 이력은 `CHANGELOG.md`에 기록한다. 정식 버전마다 기존 형식을 유지하며
  버전 번호, 주요 기능 추가, 주요 수정사항, 중요한 내부 구조 변경, 테스트·검증
  결과, 알려진 문제·남은 사항을 간결하게 남긴다.

### 작업 종료 보고

다음을 최종 보고에 포함한다.

- repository
- branch
- commit SHA
- push 성공 여부
- (정식 버전이면) tag
- (정식 버전이면) tag push 성공 여부
- CHANGELOG 갱신 여부
- (정식 버전이면) Release Notes 초안
- (정식 버전이면) Release asset으로 쓸 HTML 파일명

### 수행할 수 없을 때

인증이나 권한 문제로 commit/push/tag를 못 하면 **임의로 우회하지 말고** 어느
단계에서 왜 실패했는지 보고한다.

## 현재 환경에서 가능한 범위

2026-08 기준 이 세션 환경에서 실측한 결과다. 환경이 바뀌면 다시 확인할 것.

| 단계 | 가능 | 경로 |
|---|---|---|
| commit | 가능 | git |
| branch push | 가능 | git over HTTPS |
| tag 로컬 생성 | 가능 | git |
| **tag push** | **불가** | GitHub가 태그 ref 갱신을 403으로 거부 |
| Release 조회 | 가능 | MCP `list_releases` / `get_release_by_tag` |
| tag 조회 | 가능 | MCP `list_tags` / `get_tag`, `git ls-remote --tags` |
| **Release 생성** | **불가** | 생성용 MCP 도구 없음 |
| **Release asset 업로드** | **불가** | 업로드용 MCP 도구 없음 |

`git push origin refs/tags/<name>`은 annotated·lightweight 모두 실패한다:

```
error: RPC failed; HTTP 403 curl 22 The requested URL returned error: 403
```

응답에 `X-Github-Request-Id`가 실려 오므로 프록시가 아니라 GitHub까지 도달한 뒤
거부된 것이다. 같은 시각 같은 자격증명으로 브랜치 push는 정상 동작하므로, 이
세션의 GitHub App 토큰이 `refs/heads/`는 쓰되 `refs/tags/`는 쓰지 못한다는 뜻이다.
`info/refs?service=git-receive-pack` 핸드셰이크 자체는 200이다.

**태그는 로컬에만 만들고 push는 시도하지 않는다.** 정식 버전 태그가 필요하면
사용자가 GitHub 웹에서 Release를 만들 때 tag를 함께 생성하면 된다(Release 생성
화면에서 새 tag 이름을 입력하면 대상 커밋에 tag가 만들어진다). 최종 보고에 대상
커밋 SHA를 반드시 포함해 그 작업이 가능하게 한다.

GitHub MCP 서버는 읽기 계열 Release/tag 도구만 노출한다(`get_latest_release`,
`get_release_by_tag`, `get_tag`, `list_releases`, `list_tags`). `create_release`,
`create_tag`, asset 업로드 도구는 없다.

`gh`/`hub` CLI도 없다. `GH_TOKEN`/`GITHUB_TOKEN`은 설정돼 있으나 에이전트
프록시가 `api.github.com` 직접 호출을 막는다:

```
403 {"message":"GitHub access is not enabled for this session.
     An org admin must connect the Claude GitHub App for this organization."}
```

이건 확인된 사실이므로 **다시 시도하지 않는다.** 해소 방법은 조직 관리자가
https://claude.ai/admin-settings/claude-in-slack 에서 Claude GitHub App을
연결하는 것이다.
