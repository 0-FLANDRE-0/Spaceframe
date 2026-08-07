# SHIP-HTML — 작업 규칙

`Spaceframe.html` 단일 파일로 된 전투함 설계·전투 게임. 빌드 단계가 없고,
브라우저에서 파일을 직접 열면 그대로 실행된다.

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

## GitHub 마무리 절차

### 작업 단위마다 (WIP)

의미 있는 작업 단위가 끝날 때마다 commit하고 현재 작업 브랜치에 push한다.

- **force push 금지**
- **reset/rebase로 기존 이력 덮어쓰기 금지**
- push 전에 현재 branch와 remote를 확인할 것
- WIP 상태에서는 **GitHub Release를 만들지 않는다**

### 정식 버전이 완성됐을 때

해당 버전의 요구사항과 검증이 **모두** 끝났을 때만 아래를 수행한다.

1. 최종 commit
2. branch push
3. 버전 tag 생성 및 push
4. GitHub Release 생성
5. Release notes 작성
6. 완성된 실행용 HTML 파일을 Release asset으로 첨부

Release notes에는 다음을 간단히 포함한다.

- 주요 변경사항
- 수정된 문제
- 새 기능
- 중요한 내부 구조 변경
- 테스트/검증 결과
- 알려진 문제나 남은 작업

### 이름 충돌

기존 tag 또는 Release와 같은 버전명이 이미 존재하면 **덮어쓰거나 삭제하지 말고
먼저 보고**한다.

### 문서

- `README.md`는 프로젝트 자체 설명이 바뀔 때만 수정한다.
- 버전별 변경 이력은 `CHANGELOG.md`와 Release notes에 기록한다.
- `CHANGELOG.md`는 정식 버전마다 기존 형식을 유지하며 갱신한다.

### 작업 종료 보고

최소한 다음을 보고한다.

- branch
- commit SHA
- push 성공 여부
- (정식 버전이면) tag
- (정식 버전이면) GitHub Release 이름
- (정식 버전이면) Release asset 파일명

### 수행할 수 없을 때

인증이나 권한 문제로 commit/push/tag/release/asset 업로드를 못 하면 **임의로
우회하지 말고** 어느 단계에서 왜 실패했는지 보고한다.

## 현재 환경에서 가능한 범위

2026-08 기준 이 세션 환경에서 실측한 결과다. 환경이 바뀌면 다시 확인할 것.

| 단계 | 가능 | 경로 |
|---|---|---|
| commit | 가능 | git |
| branch push | 가능 | git over HTTPS |
| tag 생성·push | 가능 | git |
| Release 조회 | 가능 | MCP `list_releases` / `get_release_by_tag` |
| tag 조회 | 가능 | MCP `list_tags` / `get_tag`, `git ls-remote --tags` |
| **Release 생성** | **불가** | 생성용 MCP 도구 없음 |
| **Release asset 업로드** | **불가** | 업로드용 MCP 도구 없음 |

GitHub MCP 서버는 읽기 계열 Release/tag 도구만 노출한다(`get_latest_release`,
`get_release_by_tag`, `get_tag`, `list_releases`, `list_tags`). `create_release`,
`create_tag`, asset 업로드 도구는 없다.

`gh`/`hub` CLI도 없다. `GH_TOKEN`/`GITHUB_TOKEN`은 설정돼 있으나 에이전트
프록시가 `api.github.com` 직접 호출을 막는다:

```
403 {"message":"GitHub access is not enabled for this session.
     An org admin must connect the Claude GitHub App for this organization."}
```

따라서 정식 버전 릴리스 요청이 오면 **1~3단계(commit·push·tag)까지 수행하고,
4~6단계(Release·notes·asset)는 수행 불가 사실과 위 사유를 보고**한다. Release
notes 본문은 `CHANGELOG.md`에 남겨 두어 사용자가 그대로 붙여 넣을 수 있게 한다.

해소 방법은 조직 관리자가 https://claude.ai/admin-settings/claude-in-slack 에서
Claude GitHub App을 연결하는 것이다.
