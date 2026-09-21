# Chapter 00 - Development Environment

## 0. Git 기본 흐름

### git add
- 다음 commit에 포함할 변경사항을 Staging Area에 올리는 작업

### git commit
- Staging Area의 변경사항을 로컬 Git 히스토리에 기록하는 작업

### git push
- 로컬 commit을 원격 저장소(GitHub)에 전송하는 작업

### git merge
- 서로 다른 브랜치의 변경사항을 하나로 통합하는 작업

### 전체 흐름

Working Directory
→ Staging Area
→ Local Repository
→ Remote Repository

명령어 기준:

수정
→ git add
→ git commit
→ git push

브랜치 통합:

feature branch
→ Pull Request
→ CI
→ merge
→ master


## Lesson 01 - uv와 Python 프로젝트 환경

### uv란?
Python 프로젝트에서 Python 버전, 가상환경, dependency를 관리하기 위한 도구.

### 주요 파일

- `.python-version`
  - 프로젝트에서 사용할 Python 버전 정의

- `pyproject.toml`
  - 프로젝트 정보와 dependency 정의

- `uv.lock`
  - 실제 dependency resolution 결과와 버전 조합 기록

- `.venv/`
  - 프로젝트에서 실제 사용하는 로컬 Python 가상환경

### 핵심 이해

- `.python-version`
  - 어떤 Python 버전을 사용할지 정의

- `pyproject.toml`
  - 프로젝트가 무엇이고 어떤 dependency를 필요로 하는지 정의

- `uv.lock`
  - 실제 어떤 dependency 버전 조합을 사용할지 기록

- `.venv/`
  - 실제로 설치된 프로젝트 전용 실행 환경

- `uv`
  - 위 환경과 dependency를 관리하는 도구


## Lesson 02 - Dependency 관리

### dependency란?
- 프로젝트가 필요로 하는 외부 패키지

### dev dependency란?
- 개발 및 테스트 과정에서 사용하는 패키지

### direct dependency와 transitive dependency

- direct dependency
  - 내가 직접 추가한 패키지

- transitive dependency
  - direct dependency가 내부적으로 필요로 하는 패키지

### 주요 명령어

- `uv add --dev pytest`
  - 개발용 dependency로 pytest 추가

- `uv tree`
  - 프로젝트 dependency 관계를 트리 형태로 확인

### pyproject.toml과 uv.lock 차이

- `pyproject.toml`
  - 프로젝트의 dependency 요구사항 정의

- `uv.lock`
  - dependency resolution 결과와 실제 사용할 버전 조합 기록

### Git 관련 확인

첫 commit 전의 파일은 untracked 상태이므로 일반 `git diff`에는 나오지 않을 수 있다.

`git add` 후 staged된 변경사항은 다음으로 확인한다.

`git diff --staged`


## Lesson 03 - Ruff와 코드 품질

### Formatter란?
- 코드 스타일을 자동으로 정리하는 도구

### Linter란?
- 코드에서 잠재적인 문제나 스타일 위반을 검사하는 도구

### Ruff란?
- Python용 linter와 formatter 역할을 함께 제공하는 도구

### 주요 명령어

- `uv run ruff format .`
- `uv run ruff format --check .`
- `uv run ruff check .`
- `uv run ruff check . --fix`

### 기본 품질 검사 순서

1. Format
2. Lint
3. Test

### 실습에서 확인한 점

- `ruff format`은 코드 형식을 정리한다.
- 사용하지 않는 변수 같은 문제는 linter가 잡는다.
- `ruff check --fix`는 안전하게 수정 가능한 문제만 자동 수정한다.
- lint 통과와 test 통과는 서로 다른 의미를 가진다.
- CI에서는 파일을 수정하지 않고 `ruff format --check`처럼 검사만 수행하는 방식이 적절하다.


## Lesson 04 - CI와 GitHub Actions

### CI란?

CI(Continuous Integration)는 코드 변경이 Git 저장소에 올라왔을 때
코드 품질과 테스트를 자동으로 검증하는 개발 파이프라인이다.

### CI가 필요한 이유

사람이 반복적으로 검증하면 누락하거나 환경에 따라 다르게 수행할 수 있다.

CI를 사용하면 모든 변경사항에 동일한 품질 기준을 자동으로 적용할 수 있다.

### GitHub Actions란?
- GitHub에서 Workflow를 실행할 수 있는 자동화 시스템

### Workflow / Job / Step

- Workflow
  - 자동화 전체 정의

- Job
  - Workflow 안의 작업 단위

- Step
  - Job 안에서 실행되는 개별 작업

### Trigger란?
- Workflow를 실행시키는 사건 또는 조건
- 예: `push`, `pull_request`

### 현재 CI 파이프라인

1. Repository Checkout
2. uv 설치
3. Python 설치
4. Dependency 동기화
5. Formatter 검사
6. Linter 검사
7. Test

### `uv sync --locked`를 사용하는 이유

기존 `uv.lock`을 변경하지 않고
lock 파일에 기록된 dependency 상태를 기준으로 환경을 재현하기 위해 사용한다.

### 로컬 검사와 CI 차이

- 로컬
  - 내 개발환경에서 개발자가 직접 실행
  - 필요하면 코드 자동 수정 가능

- CI
  - 별도의 깨끗한 환경에서 자동 실행
  - 일반적으로 코드를 수정하지 않고 검증만 수행

### Staging Area란?
- 다음 commit에 포함할 변경사항을 임시로 선택해두는 영역


## Lesson 05 - CI 실패와 Quality Gate

### Quality Gate란?
- 다음 단계로 넘어가기 전에 반드시 만족해야 하는 품질 조건

현재 프로젝트에서는 다음이 모두 통과해야 한다.

- Formatter
- Linter
- Test

### CI 실패 실험

- `test_smoke.py`를 의도적으로 실패하도록 수정
- 로컬에서 `pytest` 실패 확인
- 실패 상태를 GitHub에 push
- GitHub Actions에서 Test 단계 실패 확인
- 테스트 복구 후 CI 성공 확인

### CI 실패와 Git Push의 관계

기본 CI만 사용할 경우:

문제 있는 코드
→ Push 가능
→ CI 실패

즉 CI는 문제를 발견하지만,
기본 설정만으로는 master 진입 자체를 막지 않는다.

이 문제를 해결하기 위해 Branch Protection과 Pull Request 기반 개발 방식이 필요하다.


## Lesson 06 - Branch와 Pull Request

### Branch를 사용하는 이유

- 중요한 브랜치에서 직접 작업하지 않기 위해
- 독립된 작업 공간에서 기능을 개발하기 위해
- 검증된 변경사항만 master에 반영하기 위해

### Feature Branch란?
- 특정 기능이나 작업을 수행하기 위해 기준 브랜치에서 분리한 작업용 브랜치

### Pull Request란?
- 특정 브랜치의 변경사항을 다른 브랜치에 반영하기 전에 검토하는 과정

PR에서는 다음을 확인할 수 있다.

- 코드 변경사항
- CI 결과
- 리뷰
- 코멘트

### PR과 Merge 차이

- Pull Request
  - 변경사항을 합치기 위한 검토 과정

- Merge
  - 실제로 두 브랜치의 변경사항을 통합하는 작업

### 기본 작업 흐름

master
→ feature branch
→ 수정
→ add
→ commit
→ push
→ Pull Request
→ CI
→ Merge
→ master

### Squash and Merge

- feature branch의 여러 commit을 하나의 새로운 commit으로 합쳐 master에 반영
- master history를 비교적 깔끔하게 유지할 수 있음
- feature branch의 원래 commit과 master에 생성되는 squash commit은 서로 다른 commit


## Lesson 07 - Branch Protection

### Branch Protection이란?
- 중요한 브랜치에 변경사항이 들어오는 방식을 제한하는 규칙

### CI만으로 부족한 이유

CI는 코드 문제를 검증할 수 있지만,
기본 설정에서는 문제가 있는 commit이 이미 master에 들어갈 수 있다.

### CI와 Branch Protection 역할 차이

- CI
  - 코드 품질 검사

- Branch Protection
  - 검사 결과를 기준으로 merge 가능 여부 통제

### 적용한 Ruleset

- Ruleset 이름: `protect-master`
- Target branch: `master`
- Require Pull Request: ON
- Required approvals: 0
- Required Status Check: `quality`
- Force Push 차단
- Enforcement: Active

### 검증 결과

- CI Workflow의 job 이름인 `quality`를 required status check로 지정
- `quality` 실패 시 Pull Request의 Merge 버튼이 비활성화되는 것을 확인
- 테스트 복구 후 CI 통과 시 Merge 가능 상태로 변경되는 것을 확인

### 최종 개발 흐름

feature branch
→ Pull Request
→ CI
→ Required Status Check
→ Merge 허용
→ master


## Chapter 00 최종 정리

이번 챕터에서 구성한 기본 개발 파이프라인:

코드 수정
→ Ruff Format
→ Ruff Lint
→ pytest
→ git add
→ git commit
→ feature branch push
→ Pull Request
→ GitHub Actions CI
→ Required Status Check
→ Squash and Merge
→ master

핵심 목표는 사람의 기억에 의존하지 않고,
자동화된 검사와 Git 정책을 이용해 일관된 품질 기준을 유지하는 것이다.