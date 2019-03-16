---
layout: post
title: Erlang - 00. Hello World
date: 2019-03-13 00:11 +0900
categories: Programming
tags: [Erlang,Programming]
comments: true
pinned: true
---

![banner-image]({{ site.url }}/assets/images/2019-03-13-erlang-00-hello-world/banner-image.png)

지난주에 [얼랭 학습기 (20190303~20190309)]({{ site.url }}{% post_url 2019-03-09-erlang-learned-20190303-20190309 %})를 작성했었습니다.  
적고나서보니 기왕이면 기초부터 쭉 정리하고싶은 마음이 밀려왔습니다.

그래서 시작했습니다. 셋업부터 시작하는 얼랭 강의입니다!

## 시작하기 전에

- 셋업 등은 `MacOS`를 기준으로 작성되었습니다. 리눅스, Windows 등에서의 셋업과 다소 차이가 있을 수 있습니다.
- 얼랭 언어에 대한 특징 설명은 생략하겠습니다. 영문 위키피디아의 [Erlang(programming language)](https://en.wikipedia.org/wiki/Erlang_(programming_language)) 항목에 자세하게 나와있어서 참고를 추천드립니다.

## Erlang 설치

- 터미널을 열어주세요. 기본 터미널, iTerm2 등 상관없습니다. 선호하는 것으로 사용합시다
- MacOS에서 Erlang을 설치하려면 `Homebrew`가 필요합니다. 설치되어있지 않을 경우 다음의 명령어를 이용해서 설치해주세요

```bash
  ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null
```

- brew를 이용해 erlang을 설치합시다

```bash
brew update
brew install erlang
```

- 설치가 완료되면 얼랭을 실행해봅시다. 터미널에 `erl`을 입력합니다. 다음과 같이 나오면 성공적입니다.

![erl-00]({{ site.url }}/assets/images/2019-03-13-erlang-00-hello-world/00.png)

- 이제, 어느 프로그래밍 언어를 배울때와 마찬가지로, 헬로월드를 출력해봅시다. 콘솔 출력 명령어는 다음과 같습니다.

```erlang
io:fwrite("Hello world!~n").
```

![erl-01]({{ site.url }}/assets/images/2019-03-13-erlang-00-hello-world/01.png)

- 훌륭합니다! 당신은 이제 얼랭의 세계에 첫 발을 힘껏 내딛었습니다.
- 간단히 설명하면, 우리는 `io`라는 모듈의 `fwrite` 함수에 `"Hello world!~n"`라는 파라미터를 넣어 호출했습니다. 줄의 맨 끝`.`은 함수의 끝을 말합니다. 얼랭의 문법에서 `,`, `;`, `.`은 매우 중요한 역할을 합니다.

## Erlang Build Tool을 활용해보자

이제 터미널에서 얼랭을 시작하는 방법을 알았으니, 얼랭 프로젝트를 구성해봅시다.
얼랭에서 사용하는 build tool은 크게 `rebar3`와 `erlang.mk`가 있습니다. `rebar3`는 Erlang/OTP에서 공식적으로 후원하는 build tool이며 완전히 얼랭으로 작성되었습니다. `erlang.mk`는 Makefile을 사용합니다. Makefile에 익숙하신 분들이라면 사용하기 편할 겁니다. 다른 여러 블로거분들께서 `rebar3`로 시작하는 글을 많이 작성해주셔서 여기서는 `erlang.mk`를 사용해보겠습니다.

- 먼저 프로젝트를 셋업할 적절한 디렉토리를 만들어주세요.

```bash
mkdir erlang_hello_world
cd erlang_hello_world
```

- `erlang.mk`를 내려받습니다.

```bash
wget https://erlang.mk/erlang.mk
# 혹은
curl -O https://erlang.mk/erlang.mk
```

- OTP 어플리케이션을 만들어봅시다. OTP 어플리케이션은 supervision tree를 가지고있는 얼랭 어플리케이션입니다. 즉, 항상 프로세스가 동작하는 프로그램입니다. `bootstrap` 타겟을 주기만하면 `Erlang.mk`를 이용해 쉽게 생성 가능합니다.

> 여기서 OTP는 Open Telecom Platform의 약자입니다. 이름에 텔레콤이 들어가지만 통신에 국한되지 않고 굉장히 유용한 미들웨어, 라이브러리 등이 있으며 Ericsson에서 Erlang/OTP을 오픈 소스로 공개했습니다.

```bash
make -f erlang.mk bootstrap bootstrap-rel
```

- `src/`는 소스 파일 디렉토리입니다. `erlang_hello_world_app.erl`는 애플리케이션, `erlang_hello_world_sup`는 supervisor 파일입니다. supervisor에 대한 자세한 설명은 다음에 하겠습니다.
- 선호하는 IDE에서 프로젝트 디렉토리 `erlang_hello_world`를 열어주세요. 저는 IntelliJ IDEA를 사용하겠습니다. 그리고 `erlang_hello_world_app.erl`을 편집합시다.
- `start(_Type, _Args)`함수에 한 줄을 추가해서 다음과 같이 보이도록 수정해주세요.

```erlang
start(_Type, _Args) ->
  io:fwrite("Hello world!~n"),
  erlang_hello_world_sup:start_link().
```

- `io:fwrite("Hello world!~n"),`의 끝은 콤마(`,`)인 점을 주의해주세요.
- 수정이 끝날 경우 `erlang_hello_world_app.erl`은 다음과 같은 모습입니다.

```erlang
-module(erlang_hello_world_app).
-behaviour(application).

-export([start/2]).
-export([stop/1]).

start(_Type, _Args) ->
  io:fwrite("Hello world!~n"),
  erlang_hello_world_sup:start_link().

stop(_State) ->
  ok.
```

- 완료되었으면 저장 후 실행시켜봅시다.

```bash
make run
```

![erl-02]({{ site.url }}/assets/images/2019-03-13-erlang-00-hello-world/02.png)

짜잔! 다음과 같이 `Hello world!`가 출력되면 정상적으로 동작한 것입니다.

## 구조 설명

- `%% erlang_hello_world_app.erl` 얼랭에서의 주석입니다. `%%`는 여타 다른 언어의 `//`와 유사합니다.
- `-module(erlang_hello_world_app).`모듈 이름 정의입니다. 이 이름을 이용해서 모듈을 부르거나 모듈 내의 함수를 호출할 수 있습니다.
- `-behaviour(application).` behaviour는 공통적(여러 모듈에서 반복적)으로 사용되는 패턴을 형식화 시킨겁니다. 답변이 잘 되었는지 모르겠네요, 나중에 주제로 잡고 추가 설명을 진행하겠습니다.
- `-export([start/2]).` 파라미터 2개를 가진 메소드`start`를 외부에서 접근할 수 있게 export하라는 명령입니다. 같은 이름의 함수더라도 파라미터 갯수에 따라 별도로 추가해야 합니다.
- `-export([stop/1]).` 위와 같습니다. 참고로 - `-export([start/2, stop/1]).`와 같이 한 번에 적을 수 있습니다.
- `start(_Type, _Args) ->` 함수의 시작입니다. 마지막에 `.` 바로 앞에서 결정된 값이 반환됩니다. 함수 내에서 파라미터를 사용하지 않을 경우에는 `_Type`, `_Args`와 같이 이름을 언더바(_)로 시작하게 합니다. 혹은 `start(_, _)`처럼 작성할 수도 있습니다.
- `erlang_hello_world_sup:start_link()`는 supervisor가 동작하도록 `start()/2`의 끝에서 호출합니다.
- `stop(_State) ->`은 `ok`를 반환하는 메소드입니다.

## 정리

축하합니다! 당신은 얼랭에서 Hello World 찍기 퀘스트를 성공적으로 완료했습니다.  
다음 포스트에 주제를 이어서 얼랭의 문법에 대해 다뤄보겠습니다.

## 기타 사항

- make에 실패하면?
  - `make --version`으로 `make`가 설치되어있는지, 버전은 몇 인지 확인해주세요.
  - 만약, `brew install make`로 설치했음에도 기존에 설치되어있던 `make`로 연결될 경우, `.bashrc` 혹은 `.zshrc`와 같은 스크립트 파일의 끝에 `PATH` 참조 경로를 추가합니다.
    - 예: `export PATH="/usr/local/opt/make/libexec/gnubin:$PATH"`

## Reference

- http://erlang.org/doc
- https://github.com/erlang/rebar3
- https://github.com/ninenines/erlang.mk
