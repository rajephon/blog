---
layout: post
title: Erlang - 04. Functions
date: 2019-04-20 14:48 +0900
categories: Programming
tags: [Erlang,Programming]
comments: true
pinned: true
---

![banner-image]({{ site.url }}/assets/images/2019-04-20-erlang-04-functions/banner-image.jpg)

Photo by Farzad [Nazifi](https://unsplash.com/photos/p-xSl33Wxyc?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/functions?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

지난 포스트 [Erlang - 03. Lists]({{ site.url }}{% post_url 2019-04-14-erlang-03-lists %})에서 리스트에 대해 알아봤습니다.  
이번 시간은 모듈과 함수에 대해 알아보고 패턴 매칭, 리스트 등 지금까지 배워온 것들을 활용해봅시다.

## 모듈과 함수

모듈은 함수의 묶음이자 `.erl` 확장자를 가진 하나의 파일입니다. 우리가 작성한 모든 코드는 모듈에 담깁니다.

### Examples

```erlang
-module(lect4).
-export([say_hello/1]).

say_hello({earthian, Name}) ->
    io:fwrite("Hello, Earthian ~p~n", [Name]);
say_hello({martian, Name}) ->
    io:fwrite("Hello, Martian ~p~n", [Name]);
say_hello({_,Name}) ->
    io:fwrite("Hello, ~p~n", [Name]).

```

다음은 얼랭 모듈 파일의 예시입니다. `-module()`, `-export()`과 같은 것들은 `module attribute`라 칭합니다. `-module()`은 모듈의 이름 정의를, `-export()`은 모듈 외부에 노출시킬 모듈 내 함수들을 입력합니다. 입력 규칙은 `함수명/파라미터 수`입니다. 일단 이것과 관련된 더 자세한 이야기는 지금 하지 않겠습니다. 일단 함수 정의 및 사용에 더 집중합시다.

얼랭은 같은 개수의 파라미터를 가지고 있는 동명의 함수를 여러 번 정의할 수 있습니다. 각각의 `say_hello` 함수 정의를 절(clause)이라고 부릅니다. 각 함수 절은 세미콜론 `;` 으로 구문하고, 마지막 절은 콤마 `.` 로 끝납니다. 함수 내의 식(expression)은 컴마 `,` 로 구문하고 가장 마지막의 명령의 결과 값이 함수의 리턴 값으로 결정됩니다.

## 절(clause)

그러면, 이제 이러한 의문이 생길지도 모릅니다. *함수명이 같고, 파라미터의 개수까지 같은데 어떻게 동작하죠?*  
해답은 바로 **패턴 매칭**에 있습니다. 파라미터의 패턴만 다르면 됩니다. 이전 강의에서 패턴 매칭에 대해 배운 것 기억하시죠? 여러 절을 가진 함수는 먼저 정의된 함수부터 패턴이 매칭되는 함수를 찾습니다. 그리고 매칭에 성공한 함수를 만나면 함수를 실행합니다.

글로만 읽으면 이해가 잘 안될 수 있습니다. 직접 컴파일하고, 호출해보며 익혀봅시다.  
적절한 디렉토리에 `lect4.erl`라는 파일을 생성하고, 상단의 `Examples` 코드를 입력합니다. 그리고 해당 파일이 위치한 디렉토리에서 `erl` 명령어로 얼랭 시작 후 모듈을 컴파일합니다. 컴파일은 간단합니다. BIF인 `c(ModuleName)`을 호출합니다.

```bash
bash> erl
Erlang/OTP 21 [erts-10.2.3] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [hipe] [dtrace]

Eshell V10.2.3  (abort with ^G)
1> c(lect4).
{ok,lect4}
2>
```

`.erl`로 작성된 얼랭 코드가 `.beam` 확장자를 가진 바이트코드 파일로 컴파일됩니다. `.beam` 파일은 얼랭 가상머신(VM) BEAM에서 동작합니다.

다음과 같이 `{ok,lect4}`가 나타나면 컴파일에 성공한 것입니다. 그리고 모듈 내 함수를 호출해봅시다. 호출 방법은 간단합니다. `module_name:function_name(param)`입니다.

```erlang
2> lect4:say_hello({martian, "Chanwoo"}).
Hello, Martian "Chanwoo"
ok
3> lect4:say_hello({earthian, "John"}).
Hello, Earthian "John"
ok
4> lect4:say_hello({blahblah, "Smith"}).
Hello, "Smith"
ok
5>
```

튜플의 첫 번째 요소에 따라 각 `martian`이면 Martian, `earthian`이면 Earthian을 출력하도록 정의한 함수에 찾아갑니다.  
`say_hello` 함수의 동작을 C++과 같은 언어로 표현하면 다음과 같을 것입니다.

```c++
#include <iostream>
#include <string>

class Lect4 {
public:
    enum Group {
        Earthian = 0,
        Martian
    };

    void sayHello(Group group, const std::string &name) {
        switch (group) {
            case Earthian:
                std::cout << "Hello, Earthian " << name << std::endl;
                break;
            case Martian:
                std::cout << "Hello, Martian " << name << std::endl;
                break;
            default:
                std::cout << "Hello, " << name << std::endl;
                break;
        }
    }
};

/* 선언 및 호출
auto lect4 = std::make_shared<Lect4>();
lect4->sayHello(Lect4::Group::Martian, "Chanwoo");
*/
```

언어마다 문법은 다르겠지만, 일반적으로 함수를 정의하고 내부에서 `if`, `switch`와 같은 구문으로 분기점을 만들어 동작을 달리 지정하는 것은 유사합니다.  
하지만 얼랭은 함수 내에서 나뉘는 것이 아니라 함수 접근부터 패턴 매칭 메커니즘이 적용되는 것이 특징입니다.

여기서 한 가지 주의사항이 있습니다. 정의된 함수부터 패턴이 매칭되는 함수를 찾는다고 했습니다. 그럼 만약에 상단에 더 큰 매칭 범위를 갖는 함수가 정의되어있으면 어떻게 될까요? `say_hello`정의를 다음과 같이 바꿔봅시다.

```erlang
say_hello(Name) ->
    io:fwrite("Hello, Astronaut ~p~n", [Name]);
say_hello({earthian, Name}) ->
    io:fwrite("Hello, Earthian ~p~n", [Name]);
say_hello({martian, Name}) ->
    io:fwrite("Hello, Martian ~p~n", [Name]);
say_hello({_,Name}) ->
    io:fwrite("Hello, ~p~n", [Name]).
```

그리고, 다시 컴파일을 한 다음 함수를 호출해봅시다.

```erlang
1> c(lect4).
lect4.erl:6: Warning: this clause cannot match because a previous clause at line 4 always matches
lect4.erl:8: Warning: this clause cannot match because a previous clause at line 4 always matches
lect4.erl:10: Warning: this clause cannot match because a previous clause at line 4 always matches
{ok,lect4}
2> lect4:say_hello({earthian, "Chanwoo"}).
Hello, Astronaut {earthian,"Chanwoo"}
```

얼랭이 아래에 위치한 절(clause)들이 매치될 수 없다고 경고를 출력합니다.  
그리고 예상한 바와 같이 튜플의 첫 번째 요소에 `earthian`을 넣어도 상단에 위치한 `say_hello(Name)`에서 매치되므로 `say_hello(Name)`가 호출됩니다. 함수를 작성할 때 이런 점을 주의해야 합니다.

## 반환 값(return value)

다음으로 간단히 리턴을 활용해봅시다. 아래의 코드를 추가합니다.

```erlang
area(circle, R) ->
    io:fwrite("Calculate circle area(radius: ~w)~n", [R]),
    math:pi() * R * R.

area(rectangle, H, W) ->
    io:fwrite("Calculate rectangle area(width: ~w, height: ~w)~n", [H,W]),
    H * W.
```

`export`에도 추가하는 것을 잊지 맙시다. 이번에는 파라미터가 2개, 3개이므로 각 `area/2, area/3`을 추가합니다.

```erlang
-export([say_hello/1, area/2, area/3]).
```

그리고 컴파일합니다. 위의 두 함수는 각 원과 직사각형의 넓이를 구하는 함수입니다. 중간의 출력 구문 `io:fwrite()` 끝의 콤마 `,`와 같이 명령을 구분하며 함수의 절(clause)을 이어나갈 수 있습니다.

```erlang
17> c(lect4).
{ok,lect4}
18> lect4:area(circle, 4).
Calculate circle area(radius: 4)
50.26548245743669
19> lect4:area(rectangle, 4, 5).
Calculate rectangle area(width: 4, height: 5)
20
```

함수의 수행 결과가 출력되는 것을 확인할 수 있습니다.

```erlang
21> A = lect4:area(rectangle, 3, 4).
Calculate rectangle area(width: 3, height: 4)
12
22> B = lect4:area(rectangle, 4, 5).
Calculate rectangle area(width: 4, height: 5)
20
23> A * B.
240
24> lect4:area(circle, lect4:area(rectangle, 5, 6)).
Calculate rectangle area(width: 5, height: 6)
Calculate circle area(radius: 30)
2827.4333882308138
```

여타 마찬가지로 함수의 실행 결과(return value)를 변수에 할당하거나, 다른 함수의 파라미터로 바로 할당할 수 있습니다.

## 재귀 호출 (Recursion)

얼랭의 함수는 패턴 매칭으로 호출이 되고, 여타 다른 언어와 마찬가지로 함수의 실행 결과를 바로 다른 함수에 전달할 수 있으며, 함수의 가장 마지막 명령의 결과가 함수의 반환 값이 된다고 했습니다.  
이러한 특징들로 정말 놀랍도록 짧고, 강력하고, 재미있는 재귀 호출을 만들 수 있습니다!

쉬운 것부터 해볼까요? 입력한 숫자까지의 합을 구하는 함수를 작성해봅시다.

```erlang
sum(0) ->
    0;
sum(N) ->
    N + sum(N - 1).
```

`sum()`의 파라미터가 0일 경우 0을, 0이 아닐 경우 `N + sum(N - 1)`을 리턴하도록 작성했습니다.  
입력 값이 3일 경우 `3 + sum(2)`이 되므로 `sum(2)`가 다시 한 번 호출되고, `3 + 2 + sum(2-1)`... 결국엔 `3 + 2 + 1 + sum(0)`이 됩니다.  
`sum(0)`은 0을 리턴하도록 설정되어있으므로, 결국엔 결과 6이 리턴됩니다.

```erlang
sum(N) when N > 0 ->
    N + sum(N - 1);
sum(0) ->
    0.
```

참고로 함수를 정의할 때 `when`라는 구문으로 조건을 줄 수 있습니다. 다음과 같이 N이 0보다 크다는 조건을 줄 경우 `sum(0)`이 `sum(N)` 하단에 정의해도 문제없습니다. 이러한 방법을 `Guards`라고 부릅니다. 결과는 똑같으니 취향에 알맞게 사용하면 됩니다.

```erlang
reverse([]) ->
    [];
reverse([LH | LT]) ->
    reverse(LT) ++ [LH].
```

다음과 같이 재귀 호출을 이용해 리스트를 뒤집어볼 수도 있습니다(물론, `lists:reverse/1`라는 함수가 있긴 합니다). 리스트의 Head와 Tail을 분리하여, Tail은 다시 한 번 호출 파라미터로 넣고, Head는 뒤에 붙이는 거죠.

```erlang
3> c(lect4).
{ok,lect4}
4> lect4:reverse([1,2,3,4,5]).
[5,4,3,2,1]
```

조금 더 재미있는 것을 해봅시다. 재귀를 배울 때 한 번쯤 해보는 [피보나치](https://en.wikipedia.org/wiki/Fibonacci_number)를 만들어볼까요?  
피보나치 수의 규칙은 다음과 같습니다.

1. F0 = 0, F1 = 1
2. FN = F(N-1) + F(N-2)

*0번째는 0, 1번째는 1이며, N번째는 F(N-1) + F(N-2)와 같다.*

이게 전부입니다. 이 규칙은 얼랭 함수로 구현하기에 정말 잘 어울립니다.

```erlang
fibonacci(N) when N < 2 ->
    N;
fibonacci(N) ->
    fibonacci(N - 1) + fibonacci(N - 2).
```

훌륭합니다! 단 네 줄만으로 피보나치를 구하는 함수를 만들었습니다.

```erlang
9> lect4:fibonacci(0).
0
10> lect4:fibonacci(1).
1
11> lect4:fibonacci(5).
5
12> lect4:fibonacci(10).
55
13> lect4:fibonacci(30).
832040
16> lect4:fibonacci(40).
102334155
```

물론 효율적인 구조는 아닙니다. 매 호출마다 이전 값을 새로 구하므로 N이 커질수록 기하급수적으로 시간을 더 필요로 합니다. 피보나치 40번째 수의 결과를 구하는데 벌써 수 초가 걸립니다.  

이전에 구한 값을 재활용하도록 최적화를 해봅시다.

```erlang
fibonacci(N) ->
    fibonacci(N, 0, 1).

fibonacci(0, Result, _Next) ->
    Result;
fibonacci(Iter, Result, Next) ->
    fibonacci(Iter - 1, Next,  Result + Next).
```

`fibonacci(N)`이 호출되면, `fibonacci(N, 0, 1)`을 재호출합니다. 파라미터는 각각 `Iterator`, `Result`, `Next`입니다.  
`Iterator`는 이름 그대로 반복 차수에 대한 계산을 담당하며, `Result`는 현재 차수까지의 계산 결과, `Next`는 다음 차수에서 `Result`의 값입니다.  
동작이 머릿속에 잘 그려지지 않으면 노트에 `fibonacci(6)`이 호출되었을 때 호출 구조를 그려보거나, `fibonacci(Iter, Result, Next)`에서 로그를 출력해보는 것도 좋습니다.

```erlang
17> c(lect4).
{ok,lect4}
18> lect4:fibonacci(40).
102334155
19> lect4:fibonacci(100).
354224848179261915075
20> lect4:fibonacci(1000).
43466557686937456435688527675040625802564660517371780402481729089536555417949051890403879840079255169295922593080322634775209689623239873322471161642996440906533187938298969649928516003704476137795166849228875
23> lect4:fibonacci(10000).
33644764876431783266621612005107543310302148460680063906564769974680081442166662368155595513633734025582065332680836159373734790483865268263040892463056431887354544369559827491606602099884183933864652731300088830269235673613135117579297437854413752130520504347701602264758318906527890855154366159582987279682987510631200575428783453215515103870818298969791613127856265033195487140214287532698187962046936097879900350962302291026368131493195275630227837628441540360584402572114334961180023091208287046088923962328835461505776583271252546093591128203925285393434620904245248929403901706233888991085841065183173360437470737908552631764325733993712871937587746897479926305837065742830161637408969178426378624212835258112820516370298089332099905707920064367426202389783111470054074998459250360633560933883831923386783056136435351892133279732908133732642652633989763922723407882928177953580570993691049175470808931841056146322338217465637321248226383092103297701648054726243842374862411453093812206564914032751086643394517512161526545361333111314042436854805106765843493523836959653428071768775328348234345557366719731392746273629108210679280784718035329131176778924659089938635459327894523777674406192240337638674004021330343297496902028328145933418826817683893072003634795623117103101291953169794607632737589253530772552375943788434504067715555779056450443016640119462580972216729758615026968443146952034614932291105970676243268515992834709891284706740862008587135016260312071903172086094081298321581077282076353186624611278245537208532365305775956430072517744315051539600905168603220349163222640885248852433158051534849622434848299380905070483482449327453732624567755879089187190803662058009594743150052402532709746995318770724376825907419939632265984147498193609285223945039707165443156421328157688908058783183404917434556270520223564846495196112460268313970975069382648706613264507665074611512677522748621598642530711298441182622661057163515069260029861704945425047491378115154139941550671256271197133252763631939606902895650288268608362241082050562430701794976171121233066073310059947366875
```

이 구조는 시간 복잡도 O(n)로, 이전 구조와 비교해서 훨씬 빠르게 연산 결과를 출력합니다.

```erlang
qsort([]) -> [];
qsort([Pivot| L]) ->
    qsort([V || V <- L, V < Pivot ]) ++ [Pivot] ++ qsort([V || V <- L, V >= Pivot]).
```

마지막으로 퀵 소트를 만들어봅시다. 놀랍도록 짧게 구현할 수 있습니다!

```erlang
32> lect4:qsort([12, 16, 2, 20, 22, 14, 30, 34, 3, 7, 18, 21, 40, 39, 1, 11, 33, 29, 28, 10, 8]).
[1,2,3,7,8,10,11,12,14,16,18,20,21,22,28,29,30,33,34,39,40]
```

다만 이 구조는 `Pivot`을 중심으로 작은 값과 큰 값을 나눌 때, 일반적인 퀵소트는 한 번의 루프에서 분할을 하는 것에 비하여 작은 값과 큰 값 각각 한 번씩 루프를 따로 돌고 있습니다. 한 번의 루프에서 분할을 할 수 있도록 수정해봅시다.

```erlang
qsort([]) -> [];
qsort([Pivot| L]) ->
    {Smaller, Larger} = partition(Pivot, L, [], []),
    qsort(Smaller) ++ [Pivot] ++ qsort(Larger).

partition(_, [], Smaller, Larger) -> {Smaller, Larger};
partition(P, [H|T], Smaller, Larger) when H >= P ->
    partition(P, T, Smaller, Larger ++ [H]);
partition(P, [H|T], Smaller, Larger) ->
    partition(P, T, Smaller ++ [H], Larger).
```

Pivot과 리스트를 입력받으면 리스트에서 값을 하나씩 꺼내 Smaller와 Larger로 구분하는 `partition`라는 함수를 만들었습니다.  
리스트가 빌 때까지 하나씩 꺼내며 재귀 호출을 하므로, O(n) 시간 복잡도로 Pivot보다 작은 값과 큰 값으로 구분할 수 있습니다.

```erlang
39> lect4:qsort([12, 16, 2, 20, 22, 14, 30, 34, 3, 7, 18, 21, 40, 39, 1, 11, 33, 29, 28, 10, 8]).
[1,2,3,7,8,10,11,12,14,16,18,20,21,22,28,29,30,33,34,39,40]
```

이제 완벽한 퀵 소트의 구현에 성공했습니다!

## 마무리

수고하셨습니다. 얼랭의 모듈과 함수에 대해 무사히 익혔습니다!  
일단 함수에 대해 알기 위하여 모듈에 대한 설명은 짧게 넘어갔습니다. 추후 얼랭의 프로젝트, 프로세스, gen_server 등의 설명을 들어가기 전에 다시 설명하겠습니다.
다음 포스트에서는 좀 더 깊게 배우기 위해 Rebar3를 이용한 프로젝트 생성에 대해 알아봅시다.

## References

- [Erlang Reference Manual : Functions](http://erlang.org/doc/efficiency_guide/functions.html)
- [Learn you some erlang : Recursion](https://learnyousomeerlang.com/recursion)
- [Wikipedia : Fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number)