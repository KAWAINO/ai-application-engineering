# GitHub CI / Branch Protection 설정 가이드

이 문서는 Python 프로젝트에서 GitHub Actions 기반 CI와 Branch Protection(Ruleset)을 설정하는 과정을 정리한 실전 가이드다.

목표는 다음 개발 흐름을 구성하는 것이다.

```text
feature branch
→ Pull Request
→ CI
→ Required Status Check
→ Merge 허용
→ master
```

---

## 1. CI Workflow 파일 생성

Repository 루트에 아래 경로로 파일을 생성한다.

```text
.github/
└── workflows/
    └── ci.yml
```

`ci.yml` 내용:

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  quality:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v6

      - name: Set up Python
        run: uv python install

      - name: Install dependencies
        run: uv sync --locked

      - name: Check formatting
        run: uv run ruff format --check .

      - name: Run lint
        run: uv run ruff check .

      - name: Run tests
        run: uv run pytest
```

정상 결과:

```text
push 또는 Pull Request 발생
→ GitHub Actions에서 CI Workflow 실행
```

---

## 2. GitHub Actions 실행 확인

GitHub Repository에서 아래 메뉴로 이동한다.

`Repository → Actions → CI`

Workflow 실행 결과를 확인한다.

정상일 경우:

```text
CI
→ quality
→ 성공
```

초록색 체크 표시가 나타나면 성공이다.

---

## 3. Feature Branch 생성

작업 전 `master`를 최신 상태로 만든다.

```bash
git switch master
git pull
git switch -c feature/<작업명>
```

예:

```bash
git switch -c feature/ch00-branch-protection
```

---

## 4. 코드 수정 후 로컬 검사

Push 전에 로컬에서 품질 검사를 실행한다.

```bash
uv run ruff format --check .
uv run ruff check .
uv run pytest
```

세 단계가 모두 통과하는지 확인한다.

```text
Format ✅
Lint   ✅
Test   ✅
```

---

## 5. 변경사항 확인 및 Commit

변경사항을 확인한다.

```bash
git status
git diff
```

Commit에 포함할 파일을 Staging Area에 올린다.

```bash
git add <변경파일>
```

Staged 변경사항을 확인한다.

```bash
git diff --staged
```

문제가 없으면 commit한다.

```bash
git commit -m "<commit message>"
```

예:

```bash
git commit -m "docs: add branch protection notes"
```

---

## 6. Feature Branch Push

처음 원격에 올리는 branch라면:

```bash
git push -u origin feature/<작업명>
```

이후 같은 branch에서는:

```bash
git push
```

만 사용해도 된다.

---

## 7. Pull Request 생성

GitHub Repository에서 아래 메뉴로 이동한다.

`Repository → Pull requests → New pull request`

PR 방향은 다음과 같이 설정한다.

```text
base
→ master

compare
→ feature/<작업명>
```

그 다음 `Create pull request` 버튼을 클릭한다.

---

## 8. Pull Request에서 CI 확인

PR 화면에서 CI 결과를 확인한다.

정상:

```text
All checks have passed
```

실패:

```text
CI / quality
→ Failed
```

현재 Workflow는 `push`, `pull_request` 둘 다 Trigger로 사용하므로 동일 변경에 CI가 두 번 실행될 수 있다.

---

## 9. Ruleset 생성

GitHub Repository에서 아래 메뉴로 이동한다.

`Repository → Settings → Rules → Rulesets`

그 다음:

`New ruleset → New branch ruleset`

을 선택한다.

---

## 10. Ruleset 기본 설정

Ruleset 이름:

```text
protect-master
```

Enforcement Status:

```text
Active
```

`Disabled` 상태에서는 규칙이 실제로 적용되지 않으므로 반드시 `Active`인지 확인한다.

---

## 11. Target Branch 지정

Ruleset 화면에서:

`Target branches → Add target`

을 선택한다.

대상 branch:

```text
master
```

---

## 12. Pull Request 필수 설정

아래 옵션을 활성화한다.

`Require a pull request before merging → ON`

개인 학습 Repository에서는 승인 리뷰를 강제하지 않는다.

```text
Required approvals
→ 0
```

---

## 13. Required Status Check 설정

다음 옵션을 활성화한다.

`Require status checks to pass → ON`

그 다음 `Add checks`를 클릭한다.

검색창에 다음을 입력한다.

```text
quality
```

`quality`를 선택한다.

`quality`는 `.github/workflows/ci.yml`에 정의한 Job 이름이다.

```yaml
jobs:
  quality:
```

최종적으로 다음과 같이 표시되어야 한다.

```text
Required checks
→ quality
```

---

## 14. Force Push 차단

Force Push를 허용하지 않도록 설정한다.

GitHub Ruleset에서는 non-fast-forward 변경 차단 관련 규칙으로 적용될 수 있다.

목표:

```text
Force Push
→ 차단
```

---

## 15. Ruleset 저장 및 상태 확인

설정 완료 후 `Save changes` 또는 `Create` 버튼을 클릭한다.

최종적으로 Ruleset 상태가 다음과 같은지 확인한다.

```text
Ruleset: protect-master
Target: master
Enforcement: Active
Require Pull Request: ON
Required Status Check: quality
Force Push: 차단
```

---

## 16. Branch Protection 동작 테스트

Branch Protection이 실제로 동작하는지 확인하기 위해 새 feature branch를 만든다.

```bash
git switch master
git pull
git switch -c feature/protection-test
```

테스트를 의도적으로 실패하도록 수정한다.

예:

```python
def test_smoke():
    assert 1 + 1 == 3
```

그 다음 commit 및 push:

```bash
git add tests/test_smoke.py
git commit -m "test: verify branch protection with failing CI"
git push -u origin feature/protection-test
```

GitHub에서 Pull Request를 생성한다.

---

## 17. CI 실패 시 Merge 차단 확인

PR의 CI 결과를 확인한다.

예상 결과:

```text
CI / quality
→ Failed
```

그리고 Merge 버튼이 비활성화되어야 한다.

```text
Merge
→ 비활성화
```

이 상태라면 Required Status Check가 정상적으로 동작하고 있는 것이다.

---

## 18. 테스트 정상 복구

실패시킨 테스트를 다시 정상화한다.

```python
def test_smoke():
    assert 1 + 1 == 2
```

README 등 필요한 문서 수정도 같은 feature branch에서 진행할 수 있다.

로컬 검사를 다시 실행한다.

```bash
uv run ruff format --check .
uv run ruff check .
uv run pytest
```

정상 결과:

```text
Format ✅
Lint   ✅
Test   ✅
```

---

## 19. 복구 변경사항 Commit 및 Push

```bash
git status
git diff

git add tests/test_smoke.py README.md

git diff --staged

git commit -m "docs: complete branch protection lesson"

git push
```

기존 Pull Request에 새 commit이 자동으로 추가되고 CI가 다시 실행된다.

---

## 20. CI 성공 후 Merge 가능 여부 확인

PR 화면에서:

```text
CI / quality
→ Passed
```

를 확인한다.

Required Status Check가 모두 통과하면 Merge 버튼이 다시 활성화된다.

---

## 21. Squash and Merge

PR 화면에서 Merge 방식을 선택한다.

`Merge → Squash and merge`

Squash and Merge는 feature branch의 여러 commit을 하나의 새로운 commit으로 합쳐 `master`에 반영한다.

장점:

```text
PR 하나
→ master commit 하나
```

형태로 history를 비교적 깔끔하게 유지할 수 있다.

---

## 22. Remote Feature Branch 삭제

Merge 완료 후 GitHub PR 화면에서 `Delete branch` 버튼을 눌러 원격 feature branch를 삭제한다.

완료된 작업용 branch는 계속 남겨두지 않는 편이 관리하기 쉽다.

---

## 23. 로컬 master 최신화

로컬에서:

```bash
git switch master
git pull
```

실행.

GitHub에서 Merge된 최신 `master` 상태를 로컬로 가져온다.

---

## 24. 로컬 Feature Branch 삭제

작업이 끝난 feature branch를 삭제한다.

```bash
git branch -d feature/<작업명>
```

Squash and Merge를 사용한 경우 다음과 같은 warning이 발생할 수 있다.

```text
not yet merged to HEAD
```

이유는 feature branch의 기존 commit과 `master`에 생성된 squash commit의 SHA가 서로 다르기 때문이다.

코드 변경사항이 `master`에 정상 반영된 것을 확인했다면 삭제해도 된다.

---

## 25. 원격 Branch 정보 정리

GitHub에서 삭제된 원격 branch 참조를 로컬에서도 정리한다.

```bash
git fetch --prune
```

---

## 26. 전체 작업 흐름

```text
master 최신화
↓
feature branch 생성
↓
코드 수정
↓
로컬 검사
↓
git add
↓
git commit
↓
git push
↓
Pull Request
↓
GitHub Actions CI
↓
Required Status Check
↓
CI 성공
↓
Squash and Merge
↓
Remote Branch 삭제
↓
git switch master
↓
git pull
↓
Local Branch 삭제
↓
git fetch --prune
```

---

## 27. 현재 CI 품질 검사 항목

현재 프로젝트의 CI에서는 아래 세 가지를 검사한다.

### Formatter

```bash
uv run ruff format --check .
```

파일을 직접 수정하지 않고 formatting 규칙 위반 여부만 검사한다.

### Linter

```bash
uv run ruff check .
```

사용하지 않는 import, 잠재적인 코드 문제 등을 검사한다.

### Test

```bash
uv run pytest
```

pytest 기반 테스트를 실행한다.

셋 중 하나라도 실패하면 `quality` Job이 실패하고 CI 전체가 실패한다.

---

## 28. 현재 Branch Protection 정책

현재 학습 Repository에서 사용한 정책:

```text
Target Branch
→ master

Pull Request
→ 필수

Required Approval
→ 0

Required Status Check
→ quality

Force Push
→ 차단

Enforcement
→ Active
```

따라서 기본 개발 흐름은 다음과 같다.

```text
feature branch
→ Pull Request
→ CI
→ quality 통과
→ Merge 가능
→ master
```

---

## 29. 핵심 기억 포인트

### CI

```text
코드가 기준을 만족하는지 자동으로 검사
```

### Branch Protection

```text
CI 결과를 이용해 중요한 branch의 진입을 통제
```

### Pull Request

```text
변경사항을 master에 합치기 전 검토하는 과정
```

### Required Status Check

```text
지정한 CI 검사가 성공해야 Merge 가능
```

### Squash and Merge

```text
feature branch의 여러 commit을 하나로 합쳐 master에 반영
```

---

## 30. 문제 발생 시 확인 순서

CI가 실패하면 아래 순서로 확인한다.

1. Pull Request
2. Checks
3. `CI / quality`
4. 실패한 Step 확인
5. 로컬에서 동일 명령 실행

예:

```bash
uv run ruff format --check .
uv run ruff check .
uv run pytest
```

Ruleset이 동작하지 않는다면 아래 메뉴로 이동한다.

`Settings → Rules → Rulesets → protect-master`

확인 항목:

```text
Enforcement
→ Active

Target branch
→ master

Require status checks to pass
→ ON

Required check
→ quality
```