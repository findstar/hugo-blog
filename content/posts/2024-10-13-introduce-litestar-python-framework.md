---
date: '2024-10-13 18:05:13 +09:00'
group: blog
image: /images/posts/litestar/litestar-framework.png
tags: ["python", "framework", "litestar", "ASGI"]
title: "Litestar: 새로운 Python 웹 프레임워크"
url: /2024/10/13/introduce-litestar-python-framework
type: post
summary: "python 웹 프로그래밍을 하면서 새로운 프레임워크를 찾아보았다. Flask, Django, FastAPI 등 다양한 선택지가 있지만, 새로운 프레임워크인 Litestar를 사용해보고 나서 소감을 정리해보았다."
목적 : "Litestar를 소개하고 앞으로의 가능성을 점쳐본다."
대상독자 : "FastAPI의 대체제를 찾는 파이썬 개발자"
---

# Litestar: 새로운 파이썬 프레임워크 소개

웹 개발을 위한 파이썬 웹 프레임워크는 많다. Django, Flask 부터 최근에 부쩍(?) 많이 보이는 FastAPI 까지 다양한 선택지가 있다. 나 또한 1년전부터 파이썬으로 애플리케이션을 개발하면서 FastAPI를 업무에 사용하고 있다.

하지만 FastAPI를 사용하면서 몇 가지 불편함이 있어서 새로운 프레임워클 찾던 중 **Litestar** 라는 새로운 프레임워크를 발견했다. 가벼우면서도 확장가능한 구조가 마음에 들어 간단하게 사용해본 소감을 정리해보았다. 

이 글에서는 Litestar의 특징과 간단한 사용법, 그리고 사용하면서 느낀 장단점을 공유하고자 한다.

## 0. 새로운 프레임워크를 찾게된 이유

작년 하반기 부터 python 을 사용하여 웹 애플리케이션을 개발하고 있다. 신규 프로젝트를 구성하는데 있어서 어떤 프레임워크를 사용할지 고민이 있었는데, 당시에 주변에서 **FastAPI** 를 많이 추천받았다. 개발하려던 애플리케이션이 **AI Model 의 Serving**을 지원하는 용도인데, 이쪽 분야에서는 대부분이 FastAPI를 사용하고 있어서 업계의 트렌드를 따르고자(?) FastAPI를 선택했다.   

다만 시간이 지날 수록 FastAPI의 단점을 느끼게 되었는데, 대표적으로 다음과 같이 정리해볼 수 있다.

### 기능 부족
* 먼저 FastAPI 프레임워크 레벨에서 제공되는 기능이 부족하다는 아쉬움이 있었다. 
* 예를 들면 특정 Route 에서만 동작하는 미들웨어를 설정하고 싶었는데 FastAPI는 글로벌 미들웨어만 사용해서 특정 path 에서만 동작하는 미들웨어를 적용하기 어려웠다.
* 프레임워크 레벨에서 Event Listener 기능을 제공하지 않아 도메인영역별 관심사 분리하려고 할 때 별도의 라이브러리를 연결해야했다.
* 또한 Depends 에 의존하는 route 정의 방식은 HTTP 테스트를 할 때 Mocking 을 어렵게해 테스트 작성하기 어렵다고 느껴졌다. 
* Cache 에 대한 지원이 부족했다. 

### 방향성의 차이
* 개발하면서 느낀 점은 FastAPI는 말 그대로 Fast 하게 API를 작성하는데 초점을 두고, 나머지 필요한 기능은 직접 구현하거나 다른 라이브러리를 사용하는 것이 일반적이라는 것이다.
* 하지만 나의 경우에는 시간도 촉박하고 팀 구성원중에 Python이 익숙하신 분이 한 분 밖에 없어 추가 기능 구현을 위한 리서치에 시간을 들이기 어려웠다.
* 검색해보면 FastAPI 레퍼런스를 많이 찾을 수 있었지만, 이 대안이 우리에게 적절한 해법이 될지 검토하는데 시간이 많이 들었다.

### 1인 개발 거버넌스 우려
* FastAPI의 개발이 [tiangolo](https://github.com/tiangolo)에게 의존적이라는 점도 우려스러운 부분이었다. (1.0 은 언제 나오는 것인가..)
* 물론 최근에는 팀의 지원을 받고 있고 스폰서쉽도 활발히 진행되는 것을 보면 프레임워크 발전이 더디다고 보기는 어렵다.
* 하지만 대부분의 기능과 관련된 PR은 creator 인 tiangolo 가 진행하고 있는 것으로 보인다. 
* 한 사람이 코드를 주로 관리하는 상황자체가 꼭 문제가 있는 것은 아니지만 이 때문에 PR 머지가 느려지거나 중요한 버그 픽스가 지연되는 것에 대한 불만글을 보게 되어 신경이 쓰였다.

이런 단점들 때문에 다른 대안을 찾았고, 그 중에 **Litestar** 프레임워크를 알게되었다.

## 1. Litestar 프레임워크 소개

**Litestar**는 ASGI를 잘 지원하고, FastAPI와 유사하게 타입 힌트를 적극적으로 활용하고 있다. 스타일을 따져보면 FastAPI와 유사한 부분이 많아 FastAPI 사용자는 큰 어려움 없이 적응할 수 있다.
2021년 12월 7일 시작했고 1.0 버전까지는 "Starlite"라는 이름으로 릴리즈 하다가, [Starlette](https://www.starlette.io/)과 혼동된다는 피드백을 수용해서 지금의 **Litestar**로 이름을 변경했다.
2024년 10월 현재는 [버전 2.12.1](https://github.com/litestar-org/litestar/releases/tag/v2.12.1) 이 최신버전이다. 

Maintainer 는 [총 5명](https://litestar.dev/about/organization.html)으로 [Litestar Org](https://github.com/litestar-org) 에서 관리하고 있다.  

포지션으로만 보면 Django와 FastAPI의 중간정도에 위치한게 아닌가 라는 생각이 드는데, Django와 같이 전체 기능을 모두 아우르는 에코시스템을 추구하지 않으면서도 FastAPI와 같은 단순함을 가지고 지향하고 있다.
사용해보면서 프레임워크의 기능들에 대한 커스터마이징 하기 수월하도록 많은 신경을 쓰고 있구나 라고 느껴졌다.   

## 2. 설치 및 기본 사용법

다음과 같이 설치하면 된다. 

```bash 
$ pip install "litestar[standard]"
```

간단하게 "Hello world" 를 출력하는 웹 API를 작성해보면 다음과 같다. 

```python
# app.py
from litestar import Litestar, get

@get("/")
async def index() -> str:
    return "Hello, world!"

app = Litestar([index])
```

그리고 cli 에서 litestar run 명령어를 입력하면 다음과 같은 화면을 확인할 수 있다.

```bash 
$ litestar run --reload 
Using Litestar app from app:app
Starting server process ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
┌──────────────────────────────┬──────────────────────┐
│ Litestar version             │ 2.12.1               │
│ Debug mode                   │ Disabled             │
│ Python Debugger on exception │ Disabled             │
│ CORS                         │ Disabled             │
│ CSRF                         │ Disabled             │
│ OpenAPI                      │ Enabled path=/schema │
│ Compression                  │ Disabled             │
└──────────────────────────────┴──────────────────────┘
INFO:     Started server process [9697]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)

```

Route 를 등록할 때 함수를 등록할 수도 있지만, Controller 클래스 기반으로 등록할 수도 있다. 

```python
# controller.py
from litestar import Controller, get
from litestar.exceptions import NotFoundException

from dataclasses import dataclass


@dataclass
class Todo:
    id: int
    title: str
    content: str

todos = [
    Todo(1, "마트 장보기", "계란 2개, 우유 1개, 당근 1개 사기"),
    Todo(2, "도서관 책 반납하기", "계란 2개, 우유 1개, 당근 1개 사기"),
]

class TodoController(Controller):
    path = "/todos"

    @get()
    async def list_todos(self) -> list[Todo]:
        return todos


    @get("/{todo_id:int}")
    async def get_todo(self, todo_id: int) -> Todo:
        # todo_id에 해당하는 Todo 객체를 찾기
        for todo in todos:
            if todo.id == todo_id:
                return todo
        # 해당하는 Todo가 없으면 404 에러 반환
        raise NotFoundException(f"Todo id {todo_id} 를 찾을 수 없습니다 ")
        
        
# app.py 
from litestar import Litestar, get

from controller import TodoController


@get("/")
async def index() -> str:
    return "Hello, world!"


app = Litestar([index, TodoController])
```

## 3. Litestar 의 장점 

### ASGI 지원

**Litestar**의 가장 큰 장점 중 하나는 최신(?!) 프레임워크답게 Python **ASGI**를 프레임워크 레벨에서 잘 지원하고 있다는 점이다. Route 등록할 때 async 로 등록해도 프레임워크가 알아서 잘 처리해준다.

```python 
from litestar import Litestar, get

@get("/")
def hello_world() -> dict[str, str]:
    """def 로 생성해도 라우트 등록에 문제가 없다."""
    return {"hello": "world"}


app = Litestar(route_handlers=[hello_world])
```

### 다양한 기능 

그리고 FastAPI 에서 아쉬원던 다양한 기능들을 `out of box`(크게 설정하지 않고도 기본적으로) 잘 제공하고 있다. 대표적으로 편리하다고 생각하는 기능은 아래와 같다. 

* [DI](https://docs.litestar.dev/2/usage/dependency-injection.html)
    - 애플리케이션 레벨 / 라우터 레벨 / 컨트롤러 레벨에서 다양하게 의존성을 정의할 수 있다. 
    - scope 에 맞게 의존성 override 가 되는 기능도 지원해서 장점이라고 생각한다.
    - 애플리케이션 bootstrapping 과정에서 필요한 dependency 를 register 해놓으면 컨트롤러에서는 편리하게 사용할 수 있다.
    - 살짝(?) 적응이 되지 않았던 건 TypeHint 기반이 아닌 parameter name 기반으로 DI가 동작한다는 점이다. 
* [Event](https://docs.litestar.dev/2/usage/events.html)
    - 객체지향/DDD 스타일로 코드를 작성할 때 도메인 영역간의 의존성 방향을 정하는 것은 아주 중요하다고 생각한다. 
    - 이 때 의존하는 도메인 영역의 이벤트를 전송할 수 있는 수단이 꼭 필요하다고 느끼는데, **Litestar** 에서는 프레임워크 레벨에서 이벤트 기능을 제공해서 좋았다.
    - 특히나 async 로 정의해놓으면 이벤트를 발생시키는 쪽에서 이벤트를 수신하는 로직을 기다리지 않아도 되게끔 잘 지원하고 있어서 좋았다.
* [OpenAPI](https://docs.litestar.dev/2/usage/openapi/index.html)
    - FastAPI 에서는 Swagger를 기본 API 문서화 도구로 지원하는데 **Litestar** 역시 이를 지원한다. **Swagger** 기본 주소는 `/schema/swagger` 가 된다.
    - 이 밖에 다양한 `Redoc`, `Scalar`, `Repidoc` 도 지원하고 커스텀 설정도 쉽게 할 수 있다.
* 이 밖에 [Cache](https://docs.litestar.dev/latest/usage/caching.html) / [Guard](https://docs.litestar.dev/2/usage/security/guards.html) / [OpenTelemetry](https://docs.litestar.dev/latest/usage/metrics/open-telemetry.html) 지원도 유용하다고 느껴졌다.

### 유용한 라이브러리와의 연결

* **Litestar**는 **SQLAlchemy**와의 연결을 지원하고 있고, 추가 [Plugin 기능](https://docs.litestar.dev/2/usage/databases/sqlalchemy/models_and_repository.html)도 제공하고 있다.
* **Litestar**는 FastAPI처럼 **Pydantic**을 통해 데이터 검증을 할 수 있다. 거기에 더해 [MessageSpec](https://jcristharif.com/msgspec/) 도 지원하고 있다.
* 템플린 엔진은 [Jinja2](https://jinja.palletsprojects.com/en/3.0.x/), [Mako](https://www.makotemplates.org/), [Minijinja](https://github.com/mitsuhiko/minijinja/tree/main/minijinja-py) 를 지원한다.

### 컨셉 

* **Litestar**는 FastAPI와 비슷하게 빠르게 시작하고 간단한 설정만으로 웹애플리케이션을 작성이 가능하다. 
* **Litestar**와 Flask, FastAPI와 같은 프레임워크의 다른점은 마이크로프레임워크를 지향하지 않는 다는 점이다. 웹애플리케이션 개발에 필요한 **다양한 기능**을 제공하고 필요한 부분들을 손쉽게 커스터마이징 할 수 있는 인터페이스를 제공한다.
* 다만 "차세대 Django"가 되는 것을 목표로 하지도 않기 때문에 자체적인 전체 에코시스템을 형성한다기 보다 여러 기능들을 잘 조합하고 적제적소에 연결할 수 있는 프레임워크라고 보는 것이 좋다.
* **그래서 전체적으로는 "Django" 와 "FastAPI"의 중간지점이 아닐까 한다.**

## 4. 단점

**Litestar**가 장점만 있으면 좋겠지만, 모든 요구사항을 만족하는 프레임워크는 없다. 그래서 사용해보면서 체감하는 단점을 생각해보았다. 

### 레퍼런스 부족

아직은 다른 프레임워크에 비해서 사용자 수가 작기 때문에 레퍼런스를 찾기 어렵다. Github Star 를 비교해보면, Django 가 79.6k, FastAPI 76.5k 인데 비해서 **Litestar** 는 5.4k 밖게 되지 않는다.
그래서 레퍼런스를 찾는 주요한 수단이 공식 Repo 로 제공하는 [Litestar-Fullstack 예제](https://github.com/litestar-org/litestar-fullstack) 혹은 [Discord 커뮤니티](https://discord.com/invite/X3FJqy8d2j)에 직접 참여해서 물어보는 방법이다.

### 문서화

문서가 아직 정돈되어 있지 않다고 느끼는데 목차가 조금 더 정리되어 그룹핑되면 좋겠다고 생각한다. 그리고 문서에 예제코드가 간단한 구조로만 되어 있어서 좀 더 상세한 예시와 설명이 있으면 좋겠다고 느끼고 있다.


## 5. 정리 및 소감

* **Litestar**는 Python 웹 개발을 위한 **새로운 가능성**을 제시하는 프레임워크이다. FastAPI처럼 ASGI를 잘 지원하며, 사용하기 쉬운 구조 덕분에 빠르게 적응할 수 있었다. 
* 나의 바램은 FastAPI 보다는 프레임워크에서 지원하는 기능이 풍부하고 그러면서도 Django 만큼 허들이 높지 않은 그럼 프레임워크를 필요로 했는데 여기에 부합한다고 느껴졌다. 
* 굳이 표현하자면 프레임워크 포지션상 FastAPI 와 Django 의 중간정도에 있다고 느껴졌다. 
* FastAPI를 사용하면서 느꼈던 다양한 단점들을 **Litestar** 쓰면서 쉽게 해소될 수 있다고 느껴져 앞으로 활용처를 늘려볼 생각이다. 
* 하지만, 이 프레임워크는 아직 초기 단계이기 때문에 큰 프로젝트에서 사용하기에는 아직 레퍼런스가 많이 부족하다. 문서도 빈약한 부분이 많아서 개선의 여지가 많다고 느껴진다. 
* 그럼에도 Litestar가 어떻게 발전할지 기대가 되며, 다른 한국의 Python 웹 개발자 분들도 **Litestar**를 사용해보며 그 매력을 느껴보면 좋겠다.

---

## 참고사항

- [Litestar 공식 사이트](https://litestar.dev)
- [Litestar Dicord](https://discord.com/invite/X3FJqy8d2j)
- [Exploring LiteStar: A Python Framework for Lightweight Web Development](https://medium.com/@rajputgajanan50/exploring-litestar-a-python-framework-for-lightweight-web-development-e3a9749f23de)
- [Python Litestar Introduction Youtube](https://www.youtube.com/watch?v=MCWwII_REY8&list=PLIO3UV9ODwNDYJVemuMB-obsUNON0-gpV&ab_channel=R3ap3rPy)
- [Litestar for Python API Development / Pydantic Model Integration](https://www.youtube.com/watch?v=lK234IODJ9A&ab_channel=BugBytes)
- [Litestar: Effortlessly Build Performant APIs - Talk Python to Me Ep.433](https://www.youtube.com/watch?v=8gnB4ToIkQg&t=3781s&ab_channel=TalkPython)
- https://dev.to/v3ss0n/litestar-20-beta-speed-of-light-power-of-stars-1j62
 
