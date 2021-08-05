## [Python Virtual Environments: A Primer](https://realpython.com/python-virtual-environments-a-primer/)

Python 가상 환경(Python Virtual Environments)을 사용하여 Python 프로젝트를 위한 개별 환경을 만들고 관리하는 방법과 각각 다른 버전의 Python을 실행하는 방법에 대해 설명한다. 또한 Python 종속성이 어떻게 저장되고 해결되는지 살펴본다.

### 목차

* Why the Need for Virtual Environments?
* What Is a Virtual Environment?
* Using Virtual Environments
* How Does a Virtual Environment Work?
* Managing Virtual Environments With virtualenvwrapper
* Using Different Versions of Python
* Conclusion

### 가상 환경이 필요한 이유?

최근 대부분 프로그래밍 언어처럼 Python 또한 고유한 방법으로 패키지(또는 모듈)를 다운로드하고 저장하여 사용하고 있다. 이 방법에는 장점이 있지만 패키지 저장과 사용에 대해 몇 가지 흥미로운 방법을 채택하고 있으며, 이로 인해 패키지를 저장하는 방법 및 위치와 같은 문제들이 발생했다.

이러한 패키지들이 시스템에 다른 위치에 설치될 수 있다. 예를 들어 대부분의 시스템 패키지는 sys.prefix가 표시하는 경로의 하위 디렉터리에 저장된다.

Mac OS X에서는 Python shell을 사용하여 sys.prefix가 가르키는 위치를 쉽게 찾을 수 있다.

``` python
>>> import sys
>>> sys.prefix
'/System/Library/Frameworks/Python.framework/Versions/3.5'
```

가상환경과 관련이 있는 것으로, [easy\_install](https://pythonhosted.org/setuptools/easy_install.html) 또는 [pip](https://en.wikipedia.org/wiki/Pip_%28package_manager%29)을 사용하여 설치된 서드 파티 패키지는 일반적으로 [site.getsitepackages](https://docs.python.org/3/library/site.html#site.getsitepackages)가 가리키는 디렉토리 중 하나에 위치한다.

``` python
>>> import site
>>> site.getsitepackages()
[
  '/System/Library/Frameworks/Python.framework/Versions/3.5/Extras/lib/python',
  '/Library/Python/3.5/site-packages'
]
```

그럼, 왜 이 모든 사소한 것들이 문제가 될까요?

기본적으로 시스템의 모든 프로젝트는 동일한 디렉토리를 사용하여 **site packages**(서드 파티 library)를 저장하고 검색하므로 이 사실을 알아야 합니다. 언뜻 보기에 이것은 별 것 같지 않을 수도 있지만, **system packages**(표준 Python library의 일부인 패키지)의 경우에는 문제가 없지만,**site packages** 경우에는 문제가 된다.

*프로젝트A*와 *프로젝트B*의 두 프로젝트가 있는 경우 두 프로젝트 모두 동일한 library *프로젝트C*에 종속되어 있다 하고 *프로젝트A*와 *프로젝트B*에서 다른 버전의 *프로젝트C*를 필요로 하면, 문제가 명백해 진다. 즉 *프로젝트A*는 v1.0.0이 필요하고 *프로젝트B*는 최신 v2.0.0이 필요할 수 있다.

site-packages 디렉토리에서 버전을 구분할 수 없기 때문에 Python에서 발생하는 실제 문제이다. 따라서 v1.0.0과 v2.0.0은 이름이 동일한 디렉토리에 있다.

```
/System/Library/Frameworks/Python.framework/Versions/3.5/Extras/lib/python/ProjectC
```

프로젝트 이름만 보고 저장하기 때문에 버전에 차이를 가질 수 없다. 따라서, *프로젝트A*와 *프로젝트B* 모두에 동일한 버전이 사용될 수 있으며, 많은 경우에 수용할 수 없다.

이때 가상 환경과 [virtualenv](https://virtualenv.readthedocs.org/en/latest/)/[venv](https://docs.python.org/3/library/venv.html) 툴을 사용한다.

### 가상 환경이란?

기본적으로 Python 가상 환경의 주된 목적은 [Python project](https://realpython.com/intermediate-python-project-ideas/)를 위한 격리된 환경을 만드는 것이다. 즉, 다른 프로젝트가 갖는 종속성과 상관없이 각 프로젝트마다 자신의 종속성을 가질 수 있다.

위의 예에서 *프로젝트A*와 *프로젝트B* 각각을 위한 별도의 가상 환경을 만들어 수행하면 됩니다. 각 환경은 다른 환경과는 별개로 자신이 선택한 *프로젝트C* 버전에 따라 달라질 수 있다.

The great thing about this is that there are no limits to the number of environments you can have since they’re just directories containing a few scripts. Plus, they’re easily created using the virtualenv or pyenv command line tools.

이는 몇 개의 스크립트로 구성된 디렉토리에 불과하므로 환경 수에 제한이 없다는 것이 가장 큰 장점이다. 더우기 가상 <code>virtualenv</code> 또는 <code>[pyenv](https://realpython.com/intro-to-pyenv/)</code> 명령어 도구를 사용하여 쉽게 만들 수 있다.

### 가상 환경 사용하기

To get started, if you’re not using Python 3, you’ll want to install the virtualenv tool with pip:

시작하기 위하여 Python 3을 사용하지 않을 경우 다음과 같이 <code>pip</code>으로 <code>virtualenv</code> 도구를 설치해야 한다.

``` bash
$ pip install virtualenv
```

Python 3을 사용하는 경우, 표준 라이브러리의 [venv](https://docs.python.org/3/library/venv.html) 모듈이 이미 설치되어 있어야 한다.

**Note**: 이제부터는 실제 명령과 관련하여 새로운 <code>venv</code> 툴과 <code>virtualenv</code>거의 차이가 없기 때문에 <code>venv</code> 툴을 사용하는 것으로 가정한다. 그러나 실제로는 매우 다른 도구이다.

새로운 프로젝트에 사용할 새 디렉토리를 만드는 것부터 시작한다.

``` bash
$ mkdir python-virtual-environments && cd python-virtual-environments
```

디렉터리 내에 새 가상 환경을 생성한다.

``` bash
# Python 2:
$ virtualenv env

# Python 3
$ python3 -m venv env
```

**Note**: 기본적으로 기존 사이트 패키지는 포함되지 않는다.

Python 3 venv 접근 방식을 사용하면 가상 환경을 만드는 데 사용할 Python 3 인터프리터의 특정 버전을 선택할 수 있는 이점이 있다. 따라서 새 환경의 기반이 되는 Python 설치에 대한 혼동을 방지할 수 있다.

From Python 3.3 to 3.4, the recommended way to create a virtual environment was to use the pyvenv command-line tool that also comes included with your Python 3 installation by default. But on 3.6 and above, python3 -m venv is the way to go.

Python 3.3에서 3.4까지를 위한 권장하는 가상 환경 생성 방법은 Python 3 설치에 기본적으로 포함되어 있는 <code>pyvenv</code> 명령어를 사용하는 것이며, 3.6 이상에서는 <code>python3 -m venv</code>를 사용하는 것이 가장 좋다.

In the above example, this command creates a directory called env, which contains a directory structure similar to this:

위의 예에서 이 명령은 <code>env</code>라는 디렉토리를 만듭다. 이 디렉토리는 아래와 유사하게 디렉토리 구조를 구성한다.

``` shell
├── bin
│   ├── activate
│   ├── activate.csh
│   ├── activate.fish
│   ├── easy_install
│   ├── easy_install-3.5
│   ├── pip
│   ├── pip3
│   ├── pip3.5
│   ├── python -> python3.5
│   ├── python3 -> python3.5
│   └── python3.5 -> /Library/Frameworks/Python.framework/Versions/3.5/bin/python3.5
├── include
├── lib
│   └── python3.5
│       └── site-packages
└── pyvenv.cfg
```

Here’s what each folder contains:

bin: files that interact with the virtual environment
include: C headers that compile the Python packages
lib: a copy of the Python version along with a site-packages folder where each dependency is installed

아래는 각 폴더에 있는 것들이다.

* <code>bin</code>: 가상 환경과 상호 작용하는 파일
* <code>include</code>: Python 패키지를 컴파일하는 C 헤더
* <code>lib</code>: 각 종속성이 설치된 <code>site-package</code> 폴더에 따른 Python 버전의 복사본

또한, Python 실행 파일 자체뿐만 아니라 몇 몇 다른 Python 도구의 복사본 또는 [symlink](https://en.wikipedia.org/wiki/Symbolic_link)가 있다. 이 파일들은 현재 환경의 컨텍스트에서 모든 Python 코드 및 명령을 실행할 수 있도록 하는 데 사용된다. 이 방법으로 글로벌 환경과의 분리가 이루어진다. 이에 대해서는 다음 섹션에서 자세히 설명한다.

더 흥미로운 것은 <code>bin</code> 디렉토리의 **activate scripts**이다. 이 스크립트는 기본적으로 가상 환경에서 사용할 Python 실행 파일과 해당 site-package를 사용할 수 있도록 shell을 설정하는 데 사용된다.

이 환경 패키지/리소스를 사용하여 격리하려면, 가상 환경을 “activate” 해야 한다. 이를 위하여 다음 명령을 실행한다.

``` bash
$ source env/bin/activate
(env) $
```

명령을 실행한 다음 프롬프트 앞에 사용자 가상 환경 이름(위의 예 경우 <code>env</code>)이 보인다. 이는 <code>env</code>가 현재 활성 상태임을 나타내는 표시이며, 이는 python 실행 파일이 이 환경의 패키지 및 설정만 사용함을 뜻하는 것이다.

패키지가 격리되어 작동하는 모습을 보여주기 위해 <code>[bcrypt](https://github.com/pyca/bcrypt/)</code> 모듈을 예로 들 수 있습니다. <code>bcrypt</code>가 시스템에 설치되어 있지만 가상 환경에는 설치되어 있지 않다고 가정해 보자.

Before we test this, we need to go back to the “system” context by executing deactivate:

테스트하기 전에 비활성화를 실행하여 "시스템" 컨텍스트로 돌아야 한다.

``` bash
(env) $ deactivate
$
```

이제 셸 세션이 정상으로 돌아왔고 python 명령은 글로벌 Python 설치를 참조하고 있다. 특정 가상 환경을 사용한 후에는 항상 이 작업을 수행해야 한다.

이제 <code>bcrypt</code>를 설치하고 암호를 해시하는 데 사용한다.

``` bash
$ pip -q install bcrypt
$ python -c "import bcrypt; print(bcrypt.hashpw('password'.encode('utf-8'), bcrypt.gensalt()))"
$2b$12$vWa/VSvxxyQ9d.WGgVTdrell515Ctux36LCga8nM5QTW0.4w8TXXi
```

가상 환경을 활성화시키고 동일한 명령을 시도하면 아래과 같은 상황이 발생한다.

``` bash
$ source env/bin/activate
(env) $ python -c "import bcrypt; print(bcrypt.hashpw('password'.encode('utf-8'), bcrypt.gensalt()))"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ImportError: No module named 'bcrypt'
```

<code>source env/bin/control</code> 명령을 수행한 다음 <code>python -c "import bcrypt"</code>의 실행 결과는 보다시피 바뀌었다.

먼저 환경에서는 <code>bcrypt</code>를 제공했고, 그 다음에서는 그렇지 않았다. 이것이 가상 환경이 추구하는 격리이며, 이를 쉽게 달성할 수 있다.

### 가상 환경은 어떻게 작동하는가?

환경을 "활성화"한다는 것이 정확히 무엇을 의미할까? 특히 실행 환경, 종속성 해결 등을 이해하는 것은 개발자에게 어떤 일이 일어나고 있는지 파악하는 것으로 매우 중요하다.

이 기능의 작동을 설명하기 위해 먼저 여러 python 실행 파일의 위치를 살펴보도록 하자. 가상 환경이 "비활성화"된 상태에서 다음을 실행한다.

``` bash
$ which python
/usr/bin/python
```

이제 활성화하고 명령을 다시 실행한다.

``` bash
$ source env/bin/activate
(env) $ which python
/Users/michaelherman/python-virtual-environments/env/bin/python
```

환경을 활성화 시킨 후, 환경 변수 <code>$PATH</code>가 약간 수정되므로 다른 python 실행 파일 경로를 얻을 수 있다.

활성화 전후 <code>$PATH</code>의 첫 번째 경로 간에 차이를 알 수 있다.

``` bash
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:

$ source env/bin/activate
(env) $ echo $PATH
/Users/michaelherman/python-virtual-environments/env/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:
```

후자의 예에서 가상 환경의 <code>bin</code> 디렉토리는 이제 경로의 시작에 있다. 즉, 명령어 실행 파일을 실행할 때 처음 검색되는 디렉터리인 것이다. 따라서 shell은 시스템 버전 대신 가상 환경의 Python 인스턴스를 호출한다.

**Note**: [Anaconda](https://en.wikipedia.org/wiki/Anaconda_%28Python_distribution%29)와 같이 Python을 묶는 패키지들 또한 활성화 시 경로를 조작하는 경향이 있다. 다른 환경에서 문제가 발생할 경우 이 점에 유의하시오. 한 번에 여러 환경을 활성화시키기 시작하면 문제가 될 수 있다.

이는 아래와 같은 질문을 야기할 수 있다.

* 어쨌든 무엇이 두 실행 파일의 차이점인가?
* 가상 환경의 Python 실행 파일이 어떻게 시스템의 site-packages 이외의 것을 사용할 수 있을까?

이는 Python이 어떻게 시작되고 시스템에서 어디에 위치하는지로 설명할 수 있다. 실제로이 두 Python 실행 파일은 차이가 없다. **그러나 그들의 디렉토리 위치가 중요하다**.

Python은 시작할 때 바이너리의 경로를 확인합니다. 가상 환경에서는 시스템 Python 바이너리의 복사본 또는 심볼 링크일 뿐이다. 다음 이 위치를 기반으로 경로의 <code>bin</code> 부분을 생략하여 <code>sys.prefix</code> 및 <code>sys.exec_prefix</code>의 위치를 설정한다.

상대 경로 <code>lib/pythonX.X/site-packages/</code>를 검색하여 <code>site-packages</code> 디렉터리를 찾는 데 <code>sys.prefix</code>에 있는 경로를 사용한다. 여기서 X.X는 사용 중인 Python 버전이다.

In our example, the binary is located at /Users/michaelherman/python-virtual-environments/env/bin, which means sys.prefix would be /Users/michaelherman/python-virtual-environments/env, and therefore the site-packages directory used would be /Users/michaelherman/python-virtual-environments/env/lib/pythonX.X/site-packages. Finally, this path is stored in the sys.path array, which contains all of the locations where a package can reside.

예에서 바이너리는 <code>/Users/michaelherman/python-virtual-environments/env/bin</code>에 있다. 즉, <code>sys.prefix</code>는 <code>/Users/michaelherman/python-virtual-environments/env</code>가 되고, <code>/Users/michaelherman/python-virtual-environments/env/lib/pythonX.X/site-packages</code>가 사용되는 site-packages 디렉토리이다. 마지막으로 패키지가 상주할 수 있는 모든 위치를 갖고 있는 <code>sys.path</code> 배열에 이 경로를 저장한다.

### virtualenvwrapper으로 가상 환경 관리하기

가상 환경은 패키지 관리와 관련된 몇 가지 큰 문제를 해결하였지만 완벽하지 않다. 몇 가지 환경을 만들고 나면 이러한 환경 자체에서 문제를 일으키기 시작 한다. 대부분 환경 관리를 중심으로 해결할 수 있다. 이를 위해 virtualenvwrapper 도구를 생성하였다. 이는 기본 가상 환경 도구를 둘러싸고 있는 래퍼 스크립트 일부일 뿐이다.

A few of the more useful features of virtualenvwrapper are that it:

virtualenvwrapper의 몇몇 유용한 기능은 아래와 같다.

* 모든 가상 환경을 한 곳에 구성한다.
* 환경을 쉽게 생성, 삭제 및 복사할 수 있는 방법을 제공한다.
* 환경 간에 전환할 수 있는 단일 명령을 제공합니다.

이러한 기능 중 일부는 작거나 중요하지 않아 보일 수 있지만, 곧 워크플로우에 추가할 중요한 도구라는 것을 알수 있게 될 것이다.

시작하려면 아래와 같이 <code>pip</code>을 사용하여 래퍼를 다운로드한다.

``` bash
$ pip install virtualenvwrapper
```

**Note**: 윈도우에서는 virtualenvwrapper-win를 사용한다.

설치되면 shell 기능을 활성화해야 한다. 설치된 [<code>virtualenvwrapper.sh</code>](http://virtualenvwrapper.sh) 스크립트에서 소스를 실행하여 이 작업을 수행할 수 있다. <code>pip</code>을 사용하여 처음 설치할 때 <code>[virtualenvwrapper.sh](http://virtualenvwrapper.sh)</code>의 정확한 위치를 출력에서 알 수 있다. 아니면 아래와 같이 간단한 shell 명령 실행으로도 가능하다.

``` bash
$ which virtualenvwrapper.sh
/usr/local/bin/virtualenvwrapper.sh
```

이 경로를 사용하여 shell의 시작 파일에 아래 세 줄을 추가한다. Bash 셸을 사용하는 경우 이 줄을 <code>~/.bashrc</code> 파일 또는 <code>~/.profile</code> 파일에 저장한다. zsh, csh 또는 fish와 같은 다른 shell의 경우 해당 shell에 해당하는 시작 파일을 사용해야 한다. 중요한 것은 로그인하거나 shell을 새로 수행할 때마다 아래의 명령이 실행된다는 것이다.

``` bash
export WORKON_HOME=$HOME/.virtualenvs   # Optional
export PROJECT_HOME=$HOME/projects      # Optional
source /usr/local/bin/virtualenvwrapper.sh
```

**NOte**: WORKON\_HOME 및 PROJECT\_HOME 환경 변수 정의는 필요 없다. 환경 변수. virtual envwrapper는 기본값으로 해당 값을 갖고 있지만, 값을 재정의할 수 있다.

마지막으로 시작 파일을 다시 수행한다.

``` bash
$ source ~/.bashrc
```

이제 모든 virtualenvwrapper 데이터/파일이 $WORKON\_HOME이 가리키는 디렉토리에 있어야 한다.

``` bash
$ echo $WORKON_HOME
/Users/michaelherman/.virtualenvs
```

이제 환경 관리에 필요한 아래 shell 명령을 사용할 수 있다.

* [workon](http://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html#workon)
* [deactivate](http://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html#deactivate)
* [mkvirtualenv](http://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html#mkvirtualenv)
* [cdvirtualenv](https://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html#cdvirtualenv)
* [rmvirtualenv](https://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html#rmvirtualenv)

명령어, 설치 및 virtualenvwrapper 구성에 대한 더 자세한 내용은 [설명서](http://virtualenvwrapper.readthedocs.org/en/latest/install.html)를 참고 하세요.

이제 아래와 같이 하여 새 프로젝트를 시작할 수 있다.

``` bash
$ mkvirtualenv my-new-project
(my-new-project) $
```

그러면 $WORKON\_HOME에 있는 디렉토리에 새 환경이 생성되고 활성화되며, 모든 virtualenvwrapper 환경이 저장된다.

해당 환경을 더 이상 사용하지 않으려면 이전과 같이 비활성화시킨다.

``` bash
(my-new-project) $ deactivate
$
```

선택할 수 있는 환경이 많은 경우, <code>workon</code> 명령을 사용하여 모든 가상 환경을 리스트할 수 있다.

``` bash
$ workon
my-new-project
my-django-project
web-scraper
```

마지막으로, 활성화 하는 명령은 아래와 같다.

``` bash
$ workon web-scraper
(web-scraper) $
```

단일 툴을 사용하고 Python 버전을 전환하려면 <code>virtualenv</code>를 사용한다. virtualenv는 사용할 Python 버전을 선택할 수 있도록 [매개 변수(parameter)](https://virtualenv.pypa.io/en/stable/reference/#cmdoption-p) <code>-p</code>를 지원한다. 이 옵션을 사용하면, 원하는 Python 버전을 쉽게 선택하여 간단하게 사용할 수 있다. 예를 들어 Python 3을 기본 버전으로 사용하고 싶다면 ...

``` bash
$ virtualenv -p $(which python3) blog_virtualenv
```

위 예는 새로운 Python 3 환경을 생성한다.

어떻게 이 명령이 수행될까? 이 명령은 **$PATH** 변수에서 지정된 명령을 찾고 해당 명령의 전체 경로를 반환하는 데 사용된다. 따라서 PYTHON\_EXE가 사용하는 -p 매개 변수로 python3의 전체 경로가 반환된다. 이는 python2에도 같이 사용될 수 있다. python2(python이 시스템에 python2로 기본 설정되어 있는 경우)를 python3으로 대체하면 된다.

이제 환경을 어디에 설치했는지 기억하지 않아도 된다. 원하는 대로 쉽게 삭제하거나 복사할 수 있으며 프로젝트 디렉토리가 휠씬 깔끔해질 것이다!

### 여러 버전의 Python 사용하기

과거 <code>virtualenv</code> 도구와 달리, <code>pyvenv</code>는 임의 버전 Python을 사용하여 환경을 만드는 것을 지원하지 않는다. 즉, 사용자가 생성하는 모든 환경에서 기본 Python 3 설치를 사용해야 하는 것을 의미한다. (--upgrade 옵션을 통해) 환경을 최신 시스템 버전의 Python으로 업그레이드할 수 있지만, 이 경우 특정 버전을 지정할 수 없다.

[Python을 설치](https://realpython.com/installing-python/)하는 여러 방법이 있지만, 다른 버전의 바이너리를 자주 제거했다가 다시 설치할 수 있을 정도로 쉽거나 유연한 방법은 거의 없다.

여기에 <code>pyenv</code> 역할이 있다.

<code>pyvenv</code>와 <code>pyenv</code>는 유사한 이름을 갖고 있지만, <code>pyvenv</code>는 시스템 레벨과 프로젝트 레벨에서 Python 버전을 전환할 수 있도록 지원하는 데 중점을 두고 있다는 점에서 다릅니다. <code>pyvenv</code>의 목적은 모듈을 분리하는 것인 반면, <code>pyenv</code>의 목적은 Python 버전을 분리하는 것이다.

먼저 [Homebrew](http://brew.sh/)(OS X) 또는 [pyenv-installer](https://github.com/yyuu/pyenv-installer)를 사용하여 pyenv를 설치하여 프로젝트를 시작할 수 있다.

**Homebrew**

``` bash
$ brew install pyenv
```

**pyenv-installer**

``` bash
$ curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

**Note**: 안타깝게도 pyenv는 Windows를 지원하지 않는다. 시도할 수 있는 대안은 [pywin](https://github.com/davidmarble/pywin)과 [anyenv](https://github.com/mislav/anyenv)가 있다.

pyenv를 시스템에 설치하면 다음과 같은 기본 명령을 사용할 수 있다.

``` bash
$ pyenv install 3.5.0   # Install new version
$ pyenv versions        # List installed versions
$ pyenv exec python -V  # Execute 'python -V' using pyenv version
```

위의 몇 명령어로 Python 3.5.0 버전을 설치하고, pyenv에게 사용할 수 있는 모든 버전을 보여달라고 요청한 다음 지정된 버전의 pyenv를 사용하여 python -V 명령을 실행한다.

제어력을 높이기 위해 사용 가능한 버전을 "global" 또는 "local" 용으로 사용할 수 있습니다. <code>local</code> 명령으로 pyenv를 사용하여 local.*python-version* 파일에 버전 넘버를 저장토록 함으로써 특정 프로젝트 또는 디렉토리에서 사용할 Python 버전을 설정할 수 있다. "local" 버전은 다음과 같이 설정할 수 있다.

``` bash
$ pyenv local 2.7.11
```

이렇게 하면 현재 디렉터리에 <code>.python-version</code> 파일이 생성된다.

``` bash
$ ls -la
total 16
drwxr-xr-x  4 michaelherman  staff  136 Feb 22 10:57 .
drwxr-xr-x  9 michaelherman  staff  306 Jan 27 20:55 ..
-rw-r--r--  1 michaelherman  staff    7 Feb 22 10:57 .python-version
-rw-r--r--  1 michaelherman  staff   52 Jan 28 17:20 main.py
```

이 파일에는 "2.7.11"만 포함되어 있을 것이다. 이제 pyenv를 사용하여 Python 스크립트를 실행하면 이 파일을 로드하고 시스템에 해당 버전이 있다면 지정된 버전이 사용된다.

예를 들어, 프로젝트 디렉토리에 다음과 같은 간단한 Python 스크립트 main.py이 있다고 가정해 보자.

``` python
import sys
print('Using version:', sys.version[:5])
```

사용되는 Python 실행 파일의 버전 넘버를 출력한다. <code>pyenv</code>와 <code>exec</code> 명령을 사용하여 설치된 Python다른 버전으로 python 스크립트를 실행할 수도 있다.

``` bash
$ python main.py
Using version: 2.7.5
$ pyenv global 3.5.0
$ pyenv exec python main.py
Using version: 3.5.0
$ pyenv local 2.7.11
$ pyenv exec python main.py
Using version: 2.7.11
```

<code>pyenv Exec python [main.py](http://main.py)</code>은 기본적으로 "global" Python 버전을 사용하지만, 현재 디렉터리에 대해 "local" 버전을 설정한 다음에는 해당 버전을 사용한다.

이는 여러 버전으로 다양한 프로젝트를 수행하는 개발자에게 매우 강력할 수 있다. 글로벌을 통해 모든 프로젝트의 기본 버전을 쉽게 변경할 수 있을 뿐만 아니라 이를 재정의하여 특수한 경우를 지정할 수도 있기 때문이다.

### 맺는 말

이 글에서는 Python 종속성을 저장하고 푸는 방법뿐만 아니라 다양한 패키징 및 버저닝 문제 해결에 도움이 되는 다양한 커뮤니티 도구 사용 방법에 대해 배웠다.

이를 요약하면, 개발자가 주의를 기울여야 하는 프로젝트의 종속성로부터 자유로울 수 있기 위하여 가상 환경이 필요하며, 이를 위하여 격리된 Python 프로그램 실행 환경을 프로젝트에 제공한다. Python 2와 Python 3를 위한 가상 환경을 구축하는 방법으로 <code>virtualenv</code>와 <code>python3 -m venv</code> 명령을 설명하였다. 또한 구축 후 프로젝트 디렉토리 구조와 설치된 요소들에 대하여 간단히 설명하였다. 또한 가상 환경이 작동하는 방법에 대하여 간단히 살명하였다. 마지막으로 가상 환경 활성화, 비활성화 등 관리 명령어에 대하여 다루었다.

보시다시피 거대한 Python 커뮤니티 덕분에 이러한 일반적인 문제를 해결할 수 있는 많은 도구들이 있다. 개발자로서의 역량을 증진시키기 위해 적극적으로 이러한 도구의 사용 방법을 익히도록 하는 것이 바람직하다. 그 결과 의도치 않은 용도를 찾거나 사용하는 다른 언어에 유사한 개념을 적용하는 방법을 배울 수도 있기 때문이다.
