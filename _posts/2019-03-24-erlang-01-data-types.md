---
layout: post
title: Erlang - 01. Data Types
date: 2019-03-24 15:16 +0900
---

![banner-image]({{ site.url }}/assets/images/2019-03-24-erlang-01-data-types/banner-image.jpg)

Photo by [Vincent Botta](https://unsplash.com/photos/bv_rJXpNU9I?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/data-type?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

지난 포스트 [Erlang - 00. Hello World]({{ site.url }}{% post_url 2019-03-13-erlang-00-hello-world %})에서  Erlang 개발 환경 세팅에서부터, 연습용 프로젝트를 생성하고, `Hello World`를 출력해보는 것까지 했습니다. 쉽게 따라 하셨을거라 생각합니다. 패키지 매니저가 워낙 좋아서 크게 어려운 점이 없었을 거예요.

오늘부터는 얼랭의 문법에 대해 알아봅시다. 첫 시간에는 그중에 가장 기본 중의 기본인 데이터 타입에 대해 알아봅시다.

## Erlang 에서의 데이터 타입들

### Terms

어떤 데이터 타입의 데이터들은 전부 `term`이라 부릅니다. 기억해주세요!

### Number

얼랭에서 숫자형은 **정수**와, **부동 소수점**이 있습니다. 그리고 일반적인 표현법 외에 얼랭만의 표기법을 이용할 수 있습니다.

- `$char`
    ASCII 값 혹은 유니코드 코드 포인트(code point)를 사용하여 값을 사용할 수 있습니다.
- `base#value`
    `base`는 진수, `value`는 해당 진수의 값을 입력하여 특정 진수를 표기할 수 있습니다.

글로만 설명해서는 잘 와닿지 않을 테니 직접 체험해봅시다. 선호하는 터미널을 열고 `erl` 명령어를 입력해서 얼랭을 실행합니다. 그리고 아래의 예제를 따라 해봅시다.

Examples:

```erlang
1> 42.
42
2> $A.
65
3> $\n.
10
4> 2#101.
5
5> 16#1f.
31
6> 2.3.
2.3
7> 2.3e3.
2.3e3
8> 2.3e-3.
0.0023
```

`$A`를 입력하니 `A`의 아스키코드값인 `65`가 입력됩니다. 그리고 `2#101`을 입력하니 2진수 101의 10진수 값인 `5`가 입력됩니다. 바로 그겁니다.

### Atom

`atom`은 리터럴이며, 이름이 있는 상수입니다. 얼랭 개발을 하며 정말 많이 마주할 것입니다.  
기본적으로 영문 소문자로 시작하며, 숫자, `_`, `@` 이외의 문자가 포함될 경우 따옴표`'`로 묶어줘야 합니다.

Examples:

```erlang
hello
phone_number
'Monday'
'phone number'
```

### Bit Strings and Binaries

bit string은 형식이 지정되지 않은 메모리 영역을 보관할 때 사용됩니다.

Bit strings은 [bit syntax](http://erlang.org/doc/reference_manual/expressions.html#bit_syntax)를 사용해 표현합니다.

8비트로 고르게 나눌 수 있는 Bit strings을 `바이너리(binaries)`라고 부릅니다.

Examples:

``` Erlang
1> <<10, 20>>
<<10,20>>
2> <<"ABC">>.
<<"ABC">>
3> <<1:1, 0:1>>.
<<2:2>>
```

더 자세한 예제는 [프로그래밍 예제](http://erlang.org/doc/programming_examples/bit_syntax.html)를 참고해주세요.

### Reference

Reference는 Erlang 런타임 시스템의 고유한 term이며, `make_ref/0` 호출에 의해 생성됩니다. [참고](http://erlang.org/doc/man/erlang.html#make_ref-0)

Examples:

```erlang
4> Tag = make_ref().
#Ref<0.2589822212.2572943363.197802>
```

### Fun

`fun`은 functional object입니다. `fun`은 익명 함수를 만들고, 그 함수 자체를 argument로 다른 함수에 전달할 수 있게 합니다.

Examples:

```erlang
1> Fun1 = fun (X) -> X+1 end.
#Fun<erl_eval.6.128620087>
2> Fun1(2).
3
```

[Fun Expressions](http://erlang.org/doc/reference_manual/expressions.html#funs)나 [Programming Examples](http://erlang.org/doc/programming_examples/funs.html)에서 더 많은 정보를 찾아볼 수 있습니다.

### Port Identifier

`Port Identifier`는 Erlang port 식별자입니다.

`open_port/2`를 이용하여 포트를 생성하면, 이 데이터 타입으로 값이 반환됩니다.

포트에 대한 자세한 정보는 [Ports and Port Drivers](http://erlang.org/doc/reference_manual/ports.html)에서 확인할 수 있습니다.

### Pid

프로세스 식별자입니다. 아래와 같은 BIFs(Built in Functions)에서 프로세스를 생성하고, return value로 Pid를 반환합니다.

- `spawn/1,2,3,4`
- `spawn_link/1,2,3,4`
- `spawn_opt/4`

Example:

`m.erl`이라는 파일을 생성하고 아래의 코드를 입력해주세요.

```erlang
-module(m).
-export([loop/0]).

loop() ->
    receive
        who_are_you ->
            io:format("I am ~p~n", [self()]),
            loop()
    end.
```

터미널을 열고, `m.erl`이 있는 디렉터리에서 얼랭을 실행합니다.

```erlang
1> c(m). %% 위에서 작성한 m.erl(m 모듈)을 컴파일합니다.
{ok,m}
2> P = spawn(m, loop, []).
<0.84.0>
3> P ! who_are_you.
I am <0.84.0>
who_are_you
```

간단히 설명하면, `spawn` 함수로 이용해 `m` 모듈의 프로세스를 실행합니다. `m` 모듈은 `who_are_you`라는 메세지를 받으면, 자기 자신의 `Pid`를 출력합니다.  
`P ! who_are_you.`는 `P`에 들어있는 `m` 모듈 프로세스의 Pid로, `who_are_you`라는 메세지를 보내라는 의미입니다. [Processes](http://erlang.org/doc/reference_manual/processes.html)에서 자세히 알아볼 수 있습니다.

### Tuple

`Tuple`은 고정된 숫자의 term을 담을 수 있는 복합 데이터 타입입니다.

```text
{Term, ..., TermN}
```

`Tuple` 속에 있는 각 `Term`은 `element`라고 부릅니다. `element`의 갯수는 `Tuple`의 사이즈라고 말합니다.

`Tuple`을 다룰 수 있는 다양한 BIFs가 있습니다.

Examples:

```erlang
1> P = {adam, 24, {july, 29}}.
{adam,24,{july,29}}
2> element(1, P).
adam
3> element(3, P).
{july,29}
4> P2 = setelement(2,P,25).
{adam,25,{july,29}}
5> tuple_size(P).
3
6> tuple_size({}).
0
```

### Map

`Map`은 Key-Value 조합의 복합 데이터 타입입니다.

```text
#{Key1=>Value1,...,KeyN=>ValueN}
```

`Map`도 `Tuple`과 마찬가지로, 각각의 Key-Value Pair를 `element`라 부르고, `element`의 개수가 `Map`의 사이즈라 합니다.

`Map`또한 `Map`을 다루는 다양한 BIFs가 있습니다.

```erlang
1> M1 = #{name=>adam, age=>24, date=>{july,29}}.
#{age => 24,date => {july,29},name => adam}
2> maps:get(name, M1).
adam
3> maps:get(date, M1).
{july,29}
4> M2=maps:update(age,25,M1).
#{age => 25,date => {july,29},name => adam}
5> map_size(M2).
3
6> map_size(#{}).
0
```

### List

`List`는 가변 개수의 term을 가지는 복합 데이터 타입입니다.

```text
[Term1,...,TermN]
```

각각의 term은 element라 부릅니다. 마찬가지로, element의 개수를 리스트의 길이(length)라 말합니다.

일반적으로, 리스트는 빈 리스트 `[]`, **head**(첫 번째 element), **tail**(나머지 element)로 구분됩니다. head와 tail은 `[H|T]`로 표현할 수 있습니다. `[Term1,...,TermN]`는 `[Term1|[...|[TermN|[]]]]` 처럼 응용하여 작성할 수 있습니다.

Examples:

```erlang
1> L1 = [a,2,{c,4}].
[a,2,{c,4}]
2> [H|T] = L1.
[a,2,{c,4}]
3> H.
a
4> T.
[2,{c,4}]
5> L2 = [d|T].
[d,2,{c,4}]
6> length(L1).
3
7> length([]).
0
```

리스트 처리 함수들은 [lists](http://erlang.org/doc/man/lists.html)에서 찾아볼 수 있습니다.

### String

String은 쌍따옴표`"`로 감싸서 사용합니다. 하지만 이는 Erlang의 데이터 타입이 아닙니다. string `"hello"`는 `[$h,$e,$l,$l,$o]`의 줄임말이며, `[104,101,108,108,111]`을 의미합니다.

두 개의 인접한 스트링 리터럴은 하나로 연결됩니다. 이 과정은 컴파일 과정에서 이루어지므로, 런타임 오버헤드가 발생하지 않습니다.

Example:

```erlang
1> "string" "42".
"string42"
```

### Record

`Record`는 고정된 개수의 엘리먼트를 보관하기 위한 데이터 구조입니다. 이름을 가진 필드와 C의 struct와 유사하지만, record는 실제 데이터 타입이 아닙니다. 그 대신에, 컴파일을 진행하는 동안 `Tuple` 표현식으로 변환됩니다. 그러므로, 특별한 액션이 취해지지 않는 이상 `shell`은 Record 표현식을 이해하지 못합니다.  
자세한 사항은 [shell (3)](http://erlang.org/doc/man/shell.html)을 참고해주세요.

Examples:

`person.erl` 파일을 생성하고, 아래의 코드를 입력해주세요.

```erlang
-module(person).
-export([new/2]).

-record(person, {name, age}).

new(Name, Age) ->
    #person{name=Name, age=Age}.
```

터미널을 열고, `person.erl`이 있는 디렉터리에서 얼랭을 실행합니다.

```erlang
1> c(person).
{ok,person}
2> person:new(ernie, 44).
{person,ernie,44}
```

[Records](http://erlang.org/doc/reference_manual/records.html)와 [Programming Examples](http://erlang.org/doc/programming_examples/records.html)에서 더 자세한 정보를 확인할 수 있습니다.

### Boolean

Erlang에서는 Boolean 데이터 타입이 존재하지 않습니다. 대신, atom `true`와 `false`을 Boolean으로 사용합니다.

```erlang
1> 2 =< 3.
true
2> true or false.
true
```

### Escape Sequences

string과 따옴표`'`로 감싼 atom 에서 다음 escape sequences를 인식할 수 있습니다.

- `\b` : Backspace
- `\d` : Delete
- `\e` : Escape
- `\f` : Form feed
- `\n` : Newline
- `\r` : Carriage return
- `\s` : Space
- `\t` : Tab
- `\v` : Vertical tab
- `\XYZ`, `\YZ`, `\Z` : Character with octal representation XYZ, YZ or Z
- `\xXY` : Character with hexadecimal representation XY
- `\x{X...}` Character with hexadecimal representation; X... is one or more hexadecimal characters
- `\^a...\^z`, `\^A...\^Z` : Control A to control Z
- `\'` : Single quote
- `\"` : Double quote
- `\\` : Backslash

### Type Conversions

BIF로 타입 변환을 위한 많은 함수들이 있습니다. [얼랭 메뉴얼](http://erlang.org/doc/man/erlang.html)에서 확인할 수 있습니다.

Examples:

```erlang
1> atom_to_list(hello).
"hello"
2> list_to_atom("hello").
hello
3> binary_to_list(<<"hello">>).
"hello"
4> binary_to_list(<<104,101,108,108,111>>).
"hello"
5> list_to_binary("hello").
<<"hello">>
6> float_to_list(7.0).
"7.00000000000000000000e+00"
7> list_to_float("7.000e+00").
7.0
8> integer_to_list(77).
"77"
9> list_to_integer("77").
77
10> tuple_to_list({a,b,c}).
[a,b,c]
11> list_to_tuple([a,b,c]).
{a,b,c}
12> term_to_binary({a,b,c}).
<<131,104,3,100,0,1,97,100,0,1,98,100,0,1,99>>
13> binary_to_term(<<131,104,3,100,0,1,97,100,0,1,98,100,0,1,99>>).
{a,b,c}
14> binary_to_integer(<<"77">>).
77
15> integer_to_binary(77).
<<"77">>
16> float_to_binary(7.0).
<<"7.00000000000000000000e+00">>
17> binary_to_float(<<"7.000e+00">>).
7.0
```

## Reference

- [Erlang Reference Manual User's Guide](http://erlang.org/doc/reference_manual/data_types.html)