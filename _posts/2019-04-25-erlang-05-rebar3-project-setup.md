---
layout: post
title: Erlang - 05. Rebar3 Project Setup
date: 2019-04-25 14:38 +0900
---

![banner-image]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/banner-image.jpg)

Photo by [Andrew Neel](https://unsplash.com/photos/cckf4TsHAuw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/project?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## 시작하기에 앞서

지난 포스트 [Erlang - 04. Functions]({{ site.url }}{% post_url 2019-04-20-erlang-04-functions %})에서 얼랭의 함수에 대해 알아봤습니다.  

이번 시간부터는 erl 파일을 이용하여 함수를 작성하고 테스트할 예정입니다.  
단순 콘솔 환경에서 모듈 파일을 로드하고 테스트해도 좋지만, IDE를 사용하는 것도 편할 것입니다. 다음의 환경을 사용해보는 것도 권장합니다.

- [Rebar3](https://www.rebar3.org/)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
  - [Erlang Plugin](https://www.jetbrains.com/help/idea/getting-started-with-erlang.html)

Rebar3는 erlang.mk과 같은 Erlang build tool입니다. 기왕이면 첫 시간에 활용했던 [erlang.mk](https://erlang.mk/)를 활용하고 싶지만 IntelliJ IDEA에서 기본적으로 지원하는 환경이 Rebar라 편의를 위해 Rebar를 기준으로 셋업 했습니다.

### Rebar3 셋업

![rebar_banner]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/rebar_banner.png)

1. Rebar3를 다운로드해주세요.
    - 방법 1. [Rebar3 공식 홈페이지](https://www.rebar3.org/)에서 내려받습니다.  
      그리고, 퍼미션을 조정해주세요. `chmod +x rebar3`
    - 방법 2. Shell에서 내려받고, 퍼미션을 조정합니다.  
      `wget https://s3.amazonaws.com/rebar3/rebar3 && chmod +x rebar3`
2. 그리고 적절한 디렉토리(예: `~/bin/`)를 만들고, rebar3를 해당 디렉토리에 둔 다음, `~/.bashrc`, `~/.zshrc` 등 자신이 사용한 Shell 파일의 PATH 환경 변수에 등록해주세요. (예: `export PATH=~/bin/:$PATH`)
3. 그리고 터미널을 다시 띄운 다음 `rebar3 --version`을 입력하여 제대로 설치가 되었는지 확인합니다.

![rebar3_version]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/rebar3_version.png)

## 프로젝트 생성

적절한 디렉토리를 정한 다음, 다음과 같이 프로젝트를 생성합니다. 참고로 이름에 언더 스코어`_` 와 같은 문자가 들어갈 경우, IntelliJ IDEA에서 프로젝트 임포트를 할 때 문제가 발생할 수 있습니다.

```bash
rebar3 new app lecture04
```

빌드 및 실행은 단지 프로젝트 경로에서 `rebar3 shell`로  테스트할 수 있습니다.

```bash
~/lecture04 > rebar3 shell
===> Verifying dependencies...
===> Compiling lecture04
Erlang/OTP 21 [erts-10.2.3] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [hipe] [dtrace]

Eshell V10.2.3  (abort with ^G)
1> ===> The rebar3 shell is a development tool; to deploy applications in production, consider using releases (http://www.rebar3.org/docs/releases)
===> Booted lecture04

1>
```

### IntelliJ IDEA 프로젝트 임포트

Erlang 플러그인이 설치된 IntelliJ IDEA를 실행하고, `Import Project`를 선택하여 프로젝트를 임포트를 합니다.

![import_project]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/import_project.png)

`Import project from external model`에서 Rebar을 선택합니다.

![import_project2]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/import_project2.png)

그리고 이전에 설치한 `rebar3`의 경로를 지정해주세요. 올바른 경로가 지정이 되면 Version이 표시됩니다.

![import_project3]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/import_project3.png)

그다음 임포트 할 프로젝트를 선택합니다. 여기선 `.(프로젝트명)`만 선택하고 `Next` 버튼을 클릭합니다.  
하단의 `_build/~`는 컴파일로 인하여 생성된 디렉토리이므로 무시해도 됩니다.

### 빌드 셋업

![edit_configuration1]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/edit_configuration1.png)

이제 프로젝트를 `Run` 할 수 있게 셋업을 해봅시다! 우측 상단의 `Add Configuration...`을 눌러주세요.

![edit_configuration2]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/edit_configuration2.png)

나타난 창에서 좌상단의 `+`를 누르고 `Erlang Rebar`를 선택해주세요. 이름은 적당히 작성합니다. Command는 단지 shell 만 적어주면 됩니다. 쉽죠? `OK` 버튼을 눌러서 저장합니다. 그러면 모든 환경 세팅은 끝났습니다.

테스트를 위해 lecture04_app.erl의 `start` 함수에 로그 출력 호출을 추가했습니다.

```erlang
start(_StartType, _StartArgs) ->
    logger:notice("Hello"),
    lecture04_sup:start_link().
```

그리고, 초록색 run 버튼 ➡️을 눌러주세요!

![run_test]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/run_test.png)

다음과 같이 `notice: Hello`이 출력되면 성공입니다!

### 유닛 테스트

마지막으로 유닛 테스트 셋업을 해봅시다. rebar3는 `eunit`과 `ct`(common test)라는 built-in test runner를 가지고 있습니다. 그중 EUnit은 매우 강력하고, 유연하며 사용하기 쉬운 얼랭 고유 유닛 테스팅 프레임워크입니다.  
rebar3와 같이 사용할 경우 얼랭 프로젝트 디렉토리에서, `rebar3 eunit`라는 명령으로 바로 유닛 테스트를 이용할 수 있습니다. IntelliJ IDEA에서 eunit를 셋업하고 사용하는 법을 알아봅시다.

프로젝트의 src 디렉토리 하단에 `test`라는 이름의 디렉토리를 만들어주세요.

![unit_test1]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/unit_test1.png)

그리고 `test` 디렉토리 하단에 `my_tests`와 같은 이름의 Erlang File을 생성합니다. Kind는 `EUnit tests`를 선택해주세요.

![unit_test2]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/unit_test2.png)

여기까지 잘 따라오셨다면 프로젝트 구조는 다음과 같은 모습일 겁니다.

![unit_test3]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/unit_test3.png)

다음으로, Run 셋업을 해봅시다. 우상단의 이전에 만들어둔 run 을 누르고, `Edit Configurations...`를 선택해주세요.

![unit_test4]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/unit_test4.png)

다시 `+`버튼을 눌러 템플릿 선택지를 확장하고, `Erlang Rebar Eunit`을 선택하여 유닛 테스트 Run을 만들어줍니다. 이전과 같이 이름은 적당히 지어주고, `Command`는 `eunit`만 입력합니다.  
그리고 `OK` 버튼을 눌러 설정을 저장해주세요. 그러면 끝입니다.

`my_tests.erl`을 적당히 추가해줍니다. eunit의 테스트 함수 이름은 `_test`로 끝나야 합니다.

```erlang
-module(my_tests).
-author("chanwoonoh").

-include_lib("eunit/include/eunit.hrl").

simple_test() ->
  ?assert(true).

simple2_test() ->
  ok.

simple3_test() ->
  ?assertEqual([97, 98, 99], "abc").

simple4_test() ->
  ?assertNot(false).
```

그리고 unit_test run을 하면 테스트 결과를 볼 수 있습니다.

![unit_test5]({{ site.url }}/assets/images/2019-04-25-erlang-05-rebar3-project-setup/unit_test5.png)

## 마무리

수고하셨습니다. Rebar3와 IntelliJ IDEA를 활용한 얼랭 프로젝트 셋업에 대해 알아봤습니다.  
별다른 어려움이 없었을 것이라 생각합니다. 다음 시간에는 얼랭의 꽃과 같은 프로세스에 대해 알아봅시다.

## References

- [Rebar3](https://www.rebar3.org/)
- [IntelliJ IDEA 2019.1 Help : Getting Started with Erlang](https://www.jetbrains.com/help/idea/getting-started-with-erlang.html)
- [EUnit - a Lightweight Unit Testing Framework for Erlang](http://erlang.org/doc/apps/eunit/chapter.html)
- [Stackoverflow : How to debug erlang code during rebar3 eunit?](https://stackoverflow.com/questions/34658714/how-to-debug-erlang-code-during-rebar3-eunit)
