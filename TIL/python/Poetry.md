## About Poetry
Poetry [공식 문서](https://python-poetry.org/docs/)의 설명은 다음과 같다.
> Poetry is a tool for **dependency management** and **packaging** in Python. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you. Poetry offers a lockfile to ensure repeatable installs, and can build your project for distribution.

종속성 관리와 패키징을 위한 도구이다. 프로젝트에서 필요한 라이브러리를 선언하면 Poetry가 이를 설치하거나 업데이트하여 관리하여 준다. 또한 반복 가능성 설치를 보장하는 lockfile을 제공하며 프로젝트를 배포할 수 있도록 빌드하는 기능도 제공한다.

### pip와의 차이점
pip는 기본적인 패키지 설치 도구로, 의존성 해결에 한계가 있다. 반면 Peotry는 고급 의존성 해결 알고리즘, 복잡한 의존성 트리를 자동으로 처리, 패키지 간 충돌을 더 효과적으로 방지하는 장점을 가지고 있다.
pip는 requirements.txt를 사용하지만 Poetry는 pyproject.toml 파일을 사용한다.

### 가상환경
Poetry는 자동으로 가상 환경을 생성해서 수동 가상 환경 설정이 불필요하다. 

### Lockfile
Poetry의 'poetry.lock' 파일은 프로젝트의 정확한 의존성 상태를 기록하는 중요한 파일이다.

Lockfile이 없을 때 (`poetry install`)는 pyproject.toml에 명시된 패키지들의 최신 버전을 다운로드, 모든 의존성을 해결하고 정확한 버전을 기록한다. 프로젝트를 특정 버전에 고정한다.(locking)
Lockfile이 있을 때 (`poetry install`)는 최신 버전이 아니더라도 lock 파일에 기록된 정확한 버전을 설치하여 의존성의 예상치 못한 변경을 방지한다. 이를 통해 모든 개발자가 동일한 패키지 환경을 유지할 수 있다.


## 프로젝트 생성하기
```
$ curl -sSL https://install.python-poetry.org | python3
```

설치 후, 터미널에서 `$ poetry`를 입력하면 다음과 같은 명령어를 확인할 수 있다.

```
Poetry (version 1.8.5)

Usage:
  command [options] [arguments]

Options:
  -h, --help                 Display help for the given command. When no command is given display help for the list command.
  -q, --quiet                Do not output any message.
  -V, --version              Display this application version.
      --ansi                 Force ANSI output.
      --no-ansi              Disable ANSI output.
  -n, --no-interaction       Do not ask any interactive question.
      --no-plugins           Disables plugins.
      --no-cache             Disables Poetry source caches.
  -C, --directory=DIRECTORY  The working directory for the Poetry command (defaults to the current working directory).
  -v|vv|vvv, --verbose       Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug.

Available commands:
  about              Shows information about Poetry.
  add                Adds a new dependency to pyproject.toml and installs it.
  build              Builds a package, as a tarball and a wheel by default.
  check              Validates the content of the pyproject.toml file and its consistency with the poetry.lock file.
  config             Manages configuration settings.
  export             Exports the lock file to alternative formats.
  help               Displays help for a command.
  init               Creates a basic pyproject.toml file in the current directory.
  install            Installs the project dependencies.
  list               Lists commands.
  lock               Locks the project dependencies.
  new                Creates a new Python project at %3Cpath%3E.
  publish            Publishes a package to a remote repository.
  remove             Removes a package from the project dependencies.
  run                Runs a command in the appropriate environment.
  search             Searches for packages on remote repositories.
  shell              Spawns a shell within the virtual environment.
  show               Shows information about packages.
  update             Update the dependencies as according to the pyproject.toml file.
  version            Shows the version of the project or bumps it when a valid bump rule is provided.

 cache
  cache clear        Clears a Poetry cache by name.
  cache list         List Poetry's caches.

 debug
  debug info         Shows debug information.
  debug resolve      Debugs dependency resolution.

 env
  env info           Displays information about the current environment.
  env list           Lists all virtualenvs associated with the current project.
  env remove         Remove virtual environments associated with the project.
  env use            Activates or creates a new virtualenv for the current project.

 self
  self add           Add additional packages to Poetry's runtime environment.
  self install       Install locked packages (incl. addons) required by this Poetry installation.
  self lock          Lock the Poetry installation's system requirements.
  self remove        Remove additional packages from Poetry's runtime environment.
  self show          Show packages from Poetry's runtime environment.
  self show plugins  Shows information about the currently installed plugins.
  self update        Updates Poetry to the latest version.

 source
  source add         Add source configuration for project.
  source remove      Remove source configured for the project.
  source show        Show information about sources configured for the project.>)
```

파이썬 프로젝트로 사용할 디렉토리를 생성한 뒤, 이동한다.

```
$ mkdir my-python-project
$ cd my-python-project
```

먼저, poetry를 init(초기화)한다.

```
$ poetry init

This command will guide you through creating your pyproject.toml config.

Package name [my-python-project]:
Version [0.1.0]:
Description []:
Author [Sujinkim-625 <hidong1015@dgu.ac.kr>, n to skip]:
License []:
Compatible Python versions [^3.13]:

Would you like to define your main dependencies interactively? (yes/no) [yes]
You can specify a package in the following forms:
  - A single name (requests): this will search for matches on PyPI
  - A name and a constraint (requests@^2.23.0)
  - A git url (git+https://github.com/python-poetry/poetry.git)
  - A git url with a revision (git+https://github.com/python-poetry/poetry.git#develop)
  - A file path (../my-package/my-package.whl)
  - A directory (../my-package/)
  - A url (https://example.com/packages/my-package-0.1.0.tar.gz)

Package to add or search for (leave blank to skip):

Would you like to define your development dependencies interactively? (yes/no) [yes]
Package to add or search for (leave blank to skip):

Generated file

[tool.poetry]
name = "my-python-project"
version = "0.1.0"
description = ""
authors = ["Sujinkim-625 <hidong1015@dgu.ac.kr>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.13"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"


Do you confirm generation? (yes/no) [yes]
```

init을 하고 나면, pyproject.toml 파일이 생성된다.

```
$ ls

pyproject.toml
```

### 의존성 라이브러리 설치하기

```
$ poetry install

Creating virtualenv my-python-project-uk3Gi9X5-py3.13 in /Users/sujinkim/Library/Caches/pypoetry/virtualenvs
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

Installing the current project: my-python-project (0.1.0)
Warning: The current project could not be installed: [Errno 2] No such file or directory: '/Users/sujinkim/python/my-python-project/README.md'
If you do not want to install the current project use --no-root.
If you want to use Poetry only for dependency management but not for packaging, you can disable package mode by setting package-mode = false in your pyproject.toml file.
In a future version of Poetry this warning will become an error!
```

lock file이 생성되었다.

```
$ ls
poetry.lock    pyproject.toml
```

아래의 명령어를 사용해도 패키지를 설치할 수 있다.
```
$ poetry add requests
```

패키지를 추가로 설치하고 싶다면 아래의 명령어를 사용하면 된다.

```
$ poetry add pandas

Using version ^2.2.3 for pandas

Updating dependencies
Resolving dependencies... (0.8s)

Package operations: 6 installs, 0 updates, 0 removals

  - Installing six (1.17.0)
  - Installing numpy (2.2.0)
  - Installing pytz (2024.2)
  - Installing python-dateutil (2.9.0.post0)
  - Installing tzdata (2024.2)
  - Installing pandas (2.2.3)

Writing lock file
```

`poetry show`명령어로 설치된 패키지들을 확인할 수 있다.

```
$ poetry show --tree

pandas 2.2.3 Powerful data structures for data analysis, time series, and statistics
├── numpy >=1.26.0
├── python-dateutil >=2.8.2
│   └── six >=1.5
├── pytz >=2020.1
└── tzdata >=2022.7
requests 2.32.3 Python HTTP for Humans.
├── certifi >=2017.4.17
├── charset-normalizer >=2,<4
├── idna >=2.5,<4
└── urllib3 >=1.21.1,<3
```

패키지 삭제는 `poetry remove` 명령어를 사용한다.
```
$ poetry remove pandas
```

### 가상환경 들어가기
가상환경에 진입한다.
```
$ poetry shell

Spawning shell within /Users/sujinkim/Library/Caches/pypoetry/virtualenvs/my-python-project-uk3Gi9X5-py3.13

emulate bash -c '. /Users/sujinkim/Library/Caches/pypoetry/virtualenvs/my-python-project-uk3Gi9X5-py3.13/bin/activate'
```

[참고 블로그 "# 나의 파이썬 환경 구축기 2 - pyenv + poetry"](https://dailyheumsi.tistory.com/244)