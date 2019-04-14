---
layout: post
title: Erlang - 03. Lists
date: 2019-04-14 17:29 +0900
categories: Programming
tags: [Erlang,Programming]
comments: true
pinned: true
---

![banner-image]({{ site.url }}/assets/images/2019-04-14-erlang-03-lists/banner-image.jpg)

Photo by [Scott Webb](https://unsplash.com/photos/VnF3HCHB0gE?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/list-up?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

지난 포스트 [Erlang - 02. Pattern Matching]({{ site.url }}{% post_url 2019-03-27-erlang-02-pattern-matching %})에서 패턴 매칭에 대해 알아봤습니다.  
오늘은 얼랭의 모듈에 대해 설명하려고 계획했습니다만, 그전에 리스트(`list`)에 대해 더 알아보고, 그것을 활용하여 함수에 대한 예제를 다양하게 풀어가보려 합니다. 모듈은 이번 시간에 리스트에 대해 배우고, 그다음 함수에 대해 설명한 다음 포스팅하겠습니다.

## 리스트

여타 다른 언어에서 리스트가 자주 사용되는 것처럼 얼랭에서도 늘 함께할 것입니다. 심지어 어떤 부분에서는 다른 언어보다 더 많이, 공기와 같이 항상 함께합니다!  

리스트에는 어떠한 것들도 다 담을 수 있습니다!

```erlang
1> A = [1,2,3,4,5].
[1,2,3,4,5]
2> B = [1, 2.0, "3", <<"4">>, e, {f,6}, [7,8,9]].
[1,2.0,"3",<<"4">>,e,{f,6},[7,8,9]]
```

상단의 예제처럼 정수, 소수, 문자열, 바이너리, atom, map, 등 무엇을 넣어도 상관없습니다.

```erlang
1> Fun1 = fun (X) -> X+1 end.
#Fun<erl_eval.6.128620087>
2> Fun2 = fun (X) -> X+2 end.
#Fun<erl_eval.6.128620087>
3> A = [Fun1, Fun2].
[#Fun<erl_eval.6.128620087>,#Fun<erl_eval.6.128620087>]
4> lists:nth(1,A).
#Fun<erl_eval.6.128620087>
5> (lists:nth(1,A))(1).
2
6> F1 = lists:nth(1,A).
#Fun<erl_eval.6.128620087>
7> F1(1).
2
8> F2 = lists:nth(2,A).
#Fun<erl_eval.6.128620087>
9> F2(1).
3
```

심지어 이처럼 `Fun`같은 functional object를 담아도 상관없습니다. 이 리스트를 어떻게 사용할지는 사용자 마음입니다!  

## 리스트를 다뤄보자

자, 이제 값을 할당해봤으니 요소를 추가, 제거하거나, 변경, 선택 등 조작법에 대해 알아봅시다.  

### Append

```erlang
1> [1,2,3,4] ++ [5,6].
[1,2,3,4,5,6]
2> [1,2,3,4] ++ [5].
[1,2,3,4,5]
3> lists:append([1,2,3], [4,5,6]).
[1,2,3,4,5,6]
4> lists:append([[1,2],[3,4],[5,6]]).
[1,2,3,4,5,6]
5> lists:append("abc", "def").
"abcdef"
6> "abc" ++ "def".
"abcdef"
```

리스트를 덧붙이는(append) 작업은 `++` 연산자를 활용합니다. `++` 연산자는 리스트 두 개를 이어붙여줍니다. 좌측의 리스트를 복사하여 우측의 리스트를 덧붙인 값을 얻을 수 있습니다. 상단의 1번과 2번처럼 좌측과 우측의 리스트를 붙여서 반환합니다.  
3번의 `lists:append(A, B)` 는 `A ++ B`와 같습니다. 한 번에 여러 리스트를 묶고 싶은 경우 4번처럼 리스트 안에 여러 리스트를 파라미터로 넘기는 방법을 사용할 수 있습니다.  
또한 [Erlang - 01. Data Types]({{ site.url }}{% post_url 2019-03-24-erlang-01-data-types %})에서 언급한 바와 같이 문자열은 리스트와 같기에 문자열 또한 5번과 6번처럼 리스트처럼 다룰 수 있습니다.

### Subtract

```erlang
1> [1,2,3,4] -- [1,3].
[2,4]
2> [1,1,2,3,4] -- [1,3].
[1,2,4]
3> [1,2,3,4] -- [2,5].
[1,3,4]
4> "1123445" -- "135".
"1244"
5> lists:subtract([1,1,2,3,4], [1,3]).
[1,2,4]
```

리스트를 덧붙이는 방법이 있다면, 빼는 방법도 있겠죠? 바로 `--` 연산자입니다. 좌측의 리스트에서 우측 리스트의 요소가 있다면 해당 요소를 제거한 값을 반환합니다. 다만, 2번처럼 좌측에 같은 값이 여러 개 있어도 우측 리스트에 있는 만큼만 빼줍니다. 또한 3번의 `5`처럼 좌측에 없는 요소가 우측에 있다면 무시합니다. 마찬가지로 문자열 제어에 사용할 수 있고, 동등한 동작을 하는 함수 `lists:subtract/2`[^1] 가 있습니다.

#### 주의사항, 혹은 알아두면 좋은 점

```erlang
1> [1,2,3] -- [1,2] -- [3].
[3]
2> [1,2,3] -- [1,2] -- [2].
[2,3]
3> [1,2,3] ++ [4,5] ++ [6].
[1,2,3,4,5,6]
```

`++`, `--` 연산자가 **right associative**(우우선 결합, 혹은 우측 결합)이라는 점입니다. 우측의 수식이 먼저 계산됩니다.  
1번에서 결과로 빈 리스트 `[]`가 아닌 `[3]`이 나온 이유가 그겁니다. 과정을 풀어봅시다.  
우측 마지막 수식 `[1,2] -- [3]`가 가장 먼저 계산됩니다. `[1,2]`에는 `3`이 요소로 포함되어있지 않아 계산 결과 `[1,2]`가 나옵니다. 그러므로 `[1,2,3] -- [1,2]`를 계산하게 되고, 그 결과 `[3]`이 나옵니다.  
2번도 마찬가지입니다. 좌측에서 우측으로 풀이되었다면 `[3]`이 결과로 나왔겠지만, 우측(뒤)에서부터 풀이되므로, `[1,2] -- [2]`로 인해 `2`가 없어지고 `[1,2,3] -- [1]`이라는 수식이 만들어집니다. 그 결과 `[2,3]`만 남았습니다. 3번도 마찬가지입니다.

### Head

```erlang
1> hd([1,2,3]).
1
2> lists:nth(1,[1,2,3]).
1
3> [Head| _ ] = [1,2,3].
[1,2,3]
4> Head.
1
```

얼랭의 리스트에서 Head(첫 번째 원소)를 가져오는 세 가지 방법이 있습니다.  

1. BIF(Built-in Functions)를 활용하는 방법입니다. BIF는 순수 얼랭이 아닌 C나 기타 다른 언어를 이용해 구현되어있습니다. 그리고 컴파일 도중 에뮬레이터에 정적으로 내장되고 표준 얼랭 라이브러리에서 사용됩니다. `hd`도 BIF 중 하나입니다.
2. 얼랭 표준 라이브러리 함수 `lists:nth(N, List)`를 활용하는 방법입니다. 이 함수는 리스트의 N 번째 요소를 반환합니다.  
    단, 주의할 점은 **N은 0이 아닌 1부터 시작합니다.**
3. 마지막으로, 패턴 매칭을 이용하는 방법입니다. 바로 직전 포스트에서 패턴 매칭에 대해 이야기했습니다. 예시 중 하나로 리스트의 `Head`와 `Tail`를 나눠 받는 법을 설명드렸는데요, 바로 그것입니다. 하단에서 깊게 설명하겠습니다. 지금은 이해가 가지 않아도 괜찮습니다.

### Tail

```erlang
1> tl([1,2,3,4,5]).
[2,3,4,5]
2> lists:nthtail(1, [1,2,3,4,5]).
[2,3,4,5]
3> lists:nthtail(2, [1,2,3,4,5]).
[3,4,5]
4> [_| Tail] = [1,2,3,4,5].
[1,2,3,4,5]
5> Tail.
[2,3,4,5]
6> [_ | [_ | TailInTail]] = [1,2,3,4,5].
[1,2,3,4,5]
7> TailInTail.
[3,4,5]
```

Tail은 Head를 제외한 나머지 부분을 Tail이라고 부릅니다. 가져오는 방법은 Head와 거의 유사합니다.

1. BIF로 `tl`이라는 함수가 있습니다. `tl`을 활용하여 Tail을 가져올 수 있습니다.
2. `lists:nth(N, List)`의 Tail 버전 `lists:nthtail(N, List)`입니다. 다만 살짝 다른 게, List에서 Tail을 가져오고, 그것의 N 번째 자리의 원소를 가져옵니다.
3. 패턴 매칭을 이용하는 방법입니다. 마찬가지로 직전 포스트에서 설명했지만, 밑에서 조금 더 살펴보겠습니다. 6번처럼 겹쳐서 사용할 수도 있다는 점을 기억해주세요

#### 리스트의 가장 마지막 값을 얻고 싶을 때

```erlang
1> lists:last([1,2,3,4]).
4
```

`lists:last/1`을 이용하면 됩니다.

### Cons Operator

```erlang
1> [H1, H2 | [H3 | Tails]] = [1,2,3,4,5,6,7,8,9,10].
[1,2,3,4,5,6,7,8,9,10]
2> h1.
h1
3> H2.
2
4> H3.
3
5> Tails.
[4,5,6,7,8,9,10]
6> X1 = [1|[2,3]].
[1,2,3]
7> X1.
[1,2,3]
8> X2 = [1,2 | [3 | [4,5]]].
[1,2,3,4,5]
9> X2.
[1,2,3,4,5]
```

리스트 중간에 `|` 기호를 두고 좌측은 Head 원소, 우측은 Tail 원소로 나누는 이 연산자를 *Cons Operator*라고 부릅니다.  
이건 패턴 매칭을 통해 값을 나눠 받거나, 반대로 변수에 리스트를 할당할 때도 사용됩니다. 예제를 통해 살펴봅시다.

- 1번과 같이 `|`의 좌측 Head의 자리에 변수를 하나가 아닌 여러 개를 둬서 첫 번째 원소부터 변수의 개수만큼 순서대로 할당받을 수 있습니다.  
    또한 우측 Tail 리스트 변수 자리에도 다시 한 번 Cons Operator를 둬서 패턴 매칭을 통해 할당받을 수 있습니다.  
    잘 이해가 안 가면 순차적으로 생각해봅시다. `|` 기호의 왼쪽 Head의 자리에 `H1`과 `H2`가 있습니다. `H1`는 리스트의 가장 첫 번째 원소인 `1`이 들어갑니다.  
    `H2`는 바로 다음 원소인 `2`가 들어갑니다. 그리고 `|`의 우측 Tail 자리엔 Head에서 빠진 원소를 제외한 나머지와 패턴 매칭을 수행합니다.  
    결과적으로 `[H3 | Tails] = [3,4,5,6,7,8,9,10]`를 수행할 차례가 되는 거죠. 결과는 어떻게 나올지 아시겠죠?
- 6번과 8번은 반대로 Head와 Tail을 Cons Operator를 통해 합친 값을 변수에 할당합니다. 원리는 똑같습니다.  
    Head와 Tail의 합으로 결과가 나오는 과정을 머릿속에 그려봅시다.

### List Comprehensions

List Comprehension(*적절한 한국어 표기를 못 찾아 그대로 표기합니다*)은 기존 리스트와 수식, 조건을 이용하여 새로운 리스트를 만드는 방법입니다.

```erlang
1> [ X*2 || X <- [1,2,3,4] ].
[2,4,6,8]
```

이를 이해하려면 새로운 연산자 `||`와 `<-`를 이해해야 합니다.  
`<-`은 우측 리스트의 요소를 각각 하나씩 X에 담습니다. 그리고 그것을 이용해 `||`의 좌측에 있는 계산을 수행하고, 결과를 새로 만들어질 리스트에 추가합니다.  

생각보다 어렵지 않습니다. 파이썬으로 치면 다음과 같습니다.

```python
>>> [x*2 for x in [1,2,3,4]]
[2, 4, 6, 8]
```

표현식이 조금 다를 뿐이지, 용도는 거의 유사합니다.

```erlang
1> [ X*Y || X <- [1,2,3] , Y <- [4,5,6] ].
[4,5,6,8,10,12,12,15,18]
2> [ {X, Y} || X <- [1,2,3] , Y <- [4,5,6] ].
[{1,4},{1,5},{1,6},{2,4},{2,5},{2,6},{3,4},{3,5},{3,6}]
3> [ X*Y || X <- [1,2,3] , Y <- [4,5,6], X*Y rem 2 == 0 ].
[4,6,8,10,12,12,18]
```

다음과 같이 여러 개의 리스트를 혼합하여 사용할 수 있습니다. 1번과 같이 X와 Y에 부여된 리스트를 반복적으로 돌면서 결과물을 얻어냅니다.  
순회하는 순서는 2번에서 유추해볼 수 있습니다. X와 Y의 중첩 Loop와 같습니다.  
또한 3번과 같이 조건을 부여할 수 있습니다. `rem`은 나머지(remainder)의 약어로 다른 언어의 `%`와 같습니다. 다음과 같이 조건을 줘서, 결과물이 짝수인 경우만 출력하도록 했습니다.

#### 활용 예
{% raw %}
```erlang
1> Members = [{{name,"Iron Man"},{age,48}},
1>  {{name,"Captain America"}, {age, 100}},
1>  {{name,"Hawkeye"},{age,47}},
1>  {{name,"Black Panther"},{age,42}},
1>  {{name,"Vision"},{age,3}},
1>  {{name,"Groot"},{age,4}},
1>  {{name,"SpiderMan"},{age,18}},
1>  {{name,"StarLord"},{age,38}}].
[{{name,"Iron Man"},{age,48}},
 {{name,"Captain America"},{age,100}},
 {{name,"Hawkeye"},{age,47}},
 {{name,"Black Panther"},{age,42}},
 {{name,"Vision"},{age,3}},
 {{name,"Groot"},{age,4}},
 {{name,"SpiderMan"},{age,18}},
 {{name,"StarLord"},{age,38}}]
2> Younger = [Name || {{_, Name}, {_, Age}} <- Members, Age < 20].
["Vision","Groot","SpiderMan"]
```


마블 시네마틱 유니버스 시리즈에는 정말 다양한 연령대의 영웅들이 나오고 있습니다. 영웅의 이름과 나이 정보를 리스트에 할당하고, 20세 이하의 영웅들만 추려내봅시다. (영웅이 너무 많아서 여기선 일부 영웅들의 정보만 활용했습니다)  
먼저, `{{name,"Iron Man"},{age,48}}`과 같은 튜플 형태로 리스트를 만들어 `Members`에 할당합니다. 그리고 패턴 매칭과 List Comprehensions를 활용하여 20세 미만의 영웅들을 추려냅니다.  
`{{_, Name}, {_, Age}} <- Members`로 각 영웅들의 이름과 나이 정보를 각각 `Name`과 `Age`에 가져옵니다. atom의 위치는 편의상 익명 변수(*anonymous variable*)를 사용했지만, `{{name, Name}, {age, Age}} <- Members`처럼 작성해도 무방합니다.  
`Age`를 이용해 조건식에서 나이를 비교하고, 조건에 만족할 경우 `Name`를 새로 생성될 리스트에 순차적으로 담았습니다.

### 기타 알아두면 유용한 리스트 함수들

```erlang
1> lists:all(fun(X) -> X rem 2 == 0 end, [1,2,3]). %% 리스트의 모든 요소가 조건에 부합하는지 검사합니다.
false
2> lists:all(fun(X) -> X < 5 end, [1,2,3]).
true
4> lists:any(fun(X) -> X rem 2 == 0 end, [1,2,3]). %% 리스트의 어느 한 요소라도 조건에 부합하는지 검사합니다.
true
5> lists:seq(1, 10). %% 첫 번째 param부터 두 번째 param까지의 연속적인 수의 리스트를 생성합니다.
[1,2,3,4,5,6,7,8,9,10]
6> lists:seq(1, 20, 3). %% 세 번째 param으로 주기를 만들 수 있습니다.
[1,4,7,10,13,16,19]
7> lists:seq(-1, -10, -1). %% 다음과 같이 감소하는 리스트를 만들 수 있습니다.
[-1,-2,-3,-4,-5,-6,-7,-8,-9,-10]
8> lists:sort([1,3,2,7,6,4,5]). %% 리스트를 정렬합니다.
[1,2,3,4,5,6,7]
9> lists:sort(fun(X,Y) -> X > Y end, [1,3,2,7,6,4,5]). %% functional object를 줘서 정렬 조건을 부여할 수도 있습니다.
[7,6,5,4,3,2,1]
10> lists:sum([1,2,3,4,5]).
15
15> A = [{X, some_value} || X <- lists:seq(1,5)].
[{1,some_value},
 {2,some_value},
 {3,some_value},
 {4,some_value},
 {5,some_value}]
16>  lists:keyfind(3, 1, A). %% tuple 리스트에서 특정 key를 가진 요소를 찾습니다. 두 번째 param은 tuple 내에서 비교할 key의 위치입니다. (*1부터 시작합니다.*)
{3,some_value}
```
{% endraw %}

## 마무리

수고하셨습니다. 얼랭의 리스트에 대해 무사히 익혔습니다!  
리스트 관련 함수들은 위에 설명한 것들 이외에도 정말 많이 있습니다. 전부 뼈가 되고 살이 되니 한 번쯤 [Erlang Reference Manual : lists](http://erlang.org/doc/man/lists.html)을 훑어보는 것을 추천합니다.  
다음 포스트에서는 얼랭의 함수에 대해 알아봅시다.

## References

- [Erlang Reference Manual : lists](http://erlang.org/doc/man/lists.html)
- [Erlang Reference Manual : 3 List Comprehensions](http://erlang.org/doc/programming_examples/list_comprehensions.html)
- [Erlang Reference Manual : 8.26 Operator Precedence](http://erlang.org/documentation/doc-6.4/doc/reference_manual/expressions.html#id82476)
- [Learn you some Erlang](https://learnyousomeerlang.com/starting-out-for-real#lists)
- [Wikibooks: List Comprehensions](https://en.wikibooks.org/wiki/Erlang_Programming/List_Comprehensions)
- [Stackoverflow: erlang How to get an element form a list by its positional index?](https://stackoverflow.com/a/22343507/5286905)

[^1]: `2`는 파라미터 갯수입니다.