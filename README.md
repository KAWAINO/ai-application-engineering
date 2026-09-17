# Chapter 00 - Development Environment

## Lesson 01 - uv와 Python 프로젝트 환경


### uv란?

Python 프로젝트에서 Python 버전, 가상환경, dependency를 관리하기 위한 도구.



### 주요 파일

- `.python-version`
  - 프로젝트가 사용할 Python 버전을 정의

- `pyproject.toml`
  - 프로젝트 정보와 dependency를 정의

- `.venv`
  - 실제 로컬 Python 실행 환경



### 사용한 명령어

- `uv 설치`
  - curl -LsSf https://astral.sh/uv/install.sh | sh

- `터미널 재시작 후 확인`
  - uv --version

- `프로젝트 디렉터리 이동`
  - cd /c/경로/ai-application-engineering

- `현재 시스템의 Python 3.13 확인`
  - python --version

- `프로젝트 Python을 3.13으로 고정`
  - uv python pin 3.13

- `uv 프로젝트 초기화`
  - uv init

- `가상환경 생성`
  - uv venv

- `uv 환경에서 Python 버전 확인`
  - uv run python --version

- `생성 파일 확인`
  - ls -la



### 내가 이해한 내용

- `.python-version`
  - 어떤 Python 버전을 사용 하는지

- `pyproject.toml`
  - 프로젝트가 무엇이고 무엇을 필요로 하는지

- `uv.lock`
  - 실제로 어떤 dependency 버전 조합을 사용 하는지

- `.venv/`
  - 프로젝트에서 사용하는 가상 환경

- `uv`
  - 위 환경들을 관리하는 도구





## Lesson 02 - Dependency 관리

### dependency란?

- 프로젝트가 필요로 하는 외부 패키지

### dev dependency란?

- 개발/테스트 시 사용하는 패키지

### direct dependency와 transitive dependency 차이

- direct dependency: 내가 직접 추가한 패키지(직접 의존성)
- transitive dependency: direct dependency가 내부적으로 필요로 하는 패키지(간접 의존성)

### uv add --dev pytest

- 이 프로젝트의 개발용 dependency로 pytest 추가

### uv tree

- 프로젝트의 dependency 관계를 트리 형태로 확인하는 명령어

### pyproject.toml과 uv.lock의 역할 차이

- pyproject.toml: 프로젝트 정보와 필요한 dependency를 정의하는 파일
- uv.lock: dependency resolution 결과와 실제 사용할 버전 조합을 기록하는 파일

### git diff가 비어 있었던 이유

- 아직 첫 commit 전이고 파일들이 Git의 추적 대상이 아닌 untracked 상태였기 때문
- `git add` 후 staged된 변경사항은 `git diff --staged`로 확인할 수 있음





## Lesson 03 - Ruff와 코드 품질

### Formatter란?
- 코드 스타일을 자동으로 정리하는 도구

### Linter란?
- 코드에서 잠재적 문제나 스타일 위반을 검사하는 도구

### Ruff란?
- Python용 linter와 formatter 역할을 함께 제공하는 도구

### 주요 명령어
- `uv run ruff format .`
- `uv run ruff check .`
- `uv run ruff check . --fix`

### 기본 품질 검사 순서
1. format
2. lint
3. test

### 현업에서 중요한 이유
- 코드 스타일 통일
- 반복적인 리뷰 감소
- 잠재적 오류 조기 발견
- CI 자동화 가능





### Ruff 실습에서 확인한 점

- `ruff format`은 코드 형식을 정리하지만 사용하지 않는 변수 같은 논리적 문제는 제거하지 않는다.
- `ruff check --fix`는 안전하게 수정 가능한 문제만 자동 수정한다.
- 일부 문제는 자동 수정 시 코드 의미가 바뀔 수 있어 직접 확인하고 수정해야 한다.
- lint 통과와 test 통과는 서로 다른 의미를 가진다.
- CI에서는 `ruff format --check`처럼 파일을 수정하지 않고 검사만 수행하는 방식이 적절하다.





## CI란?

CI(Continuous Integration)는 코드가 Git 저장소에 올라왔을 때
자동으로 코드 품질과 테스트를 검증하는 개발 파이프라인이다.

예:

- Formatter 검사
- Linter 검사
- Test 실행
- Build 확인

현재 프로젝트에서는 다음 순서로 CI를 구성할 예정이다.

1. `uv run ruff format --check .`
2. `uv run ruff check .`
3. `uv run pytest`

하나라도 실패하면 CI 전체를 실패로 처리한다.





## Lesson 04 - CI와 GitHub Actions

### CI란?
  - 코드가 Git 저장소에 올라왔을 때 자동으로 코드 품질과 테스트를 검증하는 개발 파이프라인

### CI가 필요한 이유
  - 사람이 반복적으로 수행하면 누락하거나 환경에 따라 다르게 실행할 수 있는 검증을 자동화하여, 모든 변경에 동일한 품질 기준을 적용하기 위해 사용한다.

### GitHub Actions란?
  - CI를 GitHub에서 수행하는 시스템

### Workflow / Job / Step
  - Workflow: 자동화 전체 정의
  - Job: Workflow 안의 작업 단위
  - Step: Job 안에서 실제 실행하는 하나의 단계

### Trigger란?
  - Trigger: Workflow를 실행시키는 사건 또는 조건

### 현재 CI 파이프라인

1. Repository Checkout
2. uv 설치
3. Python 설치
4. Dependency 동기화
5. Formatter 검사
6. Linter 검사
7. Test

### uv sync --locked를 사용하는 이유
  - 기존 uv.lock을 변경하지 않고, lock 파일에 기록된 dependency 상태를 기준으로 환경을 동기화하기 위해

### 로컬 검사와 CI의 차이
  - 로컬: 내 개발환경에서 개발자가 실행한다.
  - CI: 별도의 깨끗한 환경에서 자동으로 실행한다.

### Staging Area란?
  - 다음 commit에 포함할 변경사항을 임시로 선택해두는 영역