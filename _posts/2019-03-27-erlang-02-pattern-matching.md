---
layout: post
title: Erlang - 02. Pattern Matching
date: 2019-03-27 11:05 +0900
categories: Programming
tags: [Erlang,Programming]
comments: true
pinned: true
---

![banner-image]({{ site.url }}/assets/images/2019-03-27-erlang-02-pattern-matching/banner-image.jpg)

Photo by [Trang Nguyen](https://unsplash.com/photos/drke6MEs8Gg?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/pattern?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

지난 포스트 [Erlang - 01. Data Types]({{ site.url }}{% post_url 2019-03-24-erlang-01-data-types %})에서 개성 넘치는 Erlang의 데이터 타입들에 대해 알아봤습니다.  
오늘은 데이터 타입들을 활용하기 위한 첫걸음, 패턴 매칭에 대해 알아봅시다.

# Pattern Matching

패턴 매칭은 함수의 호출, `case`-`receive`-`try`-표현식 그리고 매칭 연산자(`=`, `match operator`)에서 사용되며, 얼랭의 많은 연산중 기본입니다.

좌측 패턴(`pattern`)은 우측 `term`과 대조합니다. 만약 매칭이 일치하면, 패턴의 바운드되지 않은(`unbound`) 변수들이 바운드됩니다. 그리고 매칭이 일치하지 않으면, 런타임 에러가 발생합니다.  
이게 얼랭 패턴 매칭의 모든 동작 원리입니다. 글로만 읽어서는 잘 이해가 되질 않을 겁니다. 바로 실습해봅시다.

## 시작하기 전에

얼랭의 변수 특성을 이해하고 있으면 패턴 매칭을 좀 더 쉽게 이해할 수 있습니다.

얼랭에서의 변수는 대문자나 언더스코어(`_`)로 시작하며 알파벳과 숫자, 언더스코어, `@`를 포함할 수 있습니다. 언바운드된 변수는 오직 패턴을 통해 바운드될 수 있습니다. 그리고 얼랭은 단일 할당(single assignment)이므로 각 변수는 **오직 단 한 번만 바운드될 수 있습니다**. 추후 얼랭의 표현법과 함께 자세히 설명하겠습니다.

## 실습

터미널을 열고 `erl`을 입력해서 얼랭을 실행합니다. 그리고 아래의 코드를 차례대로 입력하며, 동작을 이해해봅시다.

```erlang
1> A.
* 1: variable 'A' is unbound
2> A = 2.
2
3> A.
2
4> A + 1.
3
5> A = 3.
** exception error: no match of right hand side value 3
6> A = 2.
2
```

1. 현재 변수 `A`는 할당되지 않은 상태이므로 `unbound`라고 출력됩니다.
2. 우측 `2`와 좌측 `A`를 패턴 대조합니다. 양쪽 모두 하나의 값이고, 우측 2와 동등한 위치에 있는 `A`는 `unbound`상태이므로, `A`에 `2`를 할당합니다. 당장 동등한 위치라는 게 이해가 안 될 수 있습니다. 뒤에 설명할테니 일단 넘어갑니다.
3. 다시 `A`를 입력하면 이제 `A`에 할당된 `2`가 출력됩니다.
4. 현재 `A`에는 `2`가 할당되어 있으므로, `A + 1`의 수식은 `2 + 1`인 `3`을 출력합니다.
5. `A`와 `3`을 대조합니다. 현재 `A`는 값 `2`가 이미 할당되어있습니다. 만약 `A`가 `unbound`상태라면 `3`이 `A`에 할당되겠지만, 얼랭에서 변수는 단 한 번만 바운드될 수 있으므로 이 수식은 값 할당이 아닌 패턴 대조가 진행됩니다. 좌측 패턴 `A`는 `2`, 우측 `term`은 `3`, 패턴이 일치하지 않으므로 런타임 에러가 발생합니다.
6. 마찬가지로 `A`는 `2`가 할당되어 있으므로 패턴 대조가 이루어집니다. 좌측 `A`는 `2`, 우측 `term`또한 `2`로 일치하므로 별문제 없이 넘어갑니다.
  ```erlang
7> {A, B} = {1, 2}.
** exception error: no match of right hand side value {1,2}
8> {A, B} = {2, 3}.
{2,3}
9> B.
3
10> {A, B} = {2, 3}.
{2,3}
11> {A, B} = {2, 3, 4}.
** exception error: no match of right hand side value {2,3,4}
```

7. 이번엔 조금 응용해서 tuple 패턴 대조를 해봅시다. 보다시피 좌, 우 패턴 모양새는 일치합니다. `B`는 `unbound` 상태라 같은 위치의 값을 할당받을 수 있지만, `A`에는 이미 `2`가 할당되어있고, 우측 term의 동일한 위치엔 `1`이 있어 패턴 매칭에 실패합니다.
8. 이번엔 `A`위치에 `2`가 있어 매칭에 성공합니다. 변수 `B`는 `3`을 할당받게 됩니다.
9. `B`에 3이 할당된 것을 확인할 수 있습니다.
10. 6번과 마찬가지로 `A`, `B`에 할당된 값이 우측과 일치하면 통과합니다.
11. 좌측 패턴과 우측 tuple의 원소 개수가 다르므로 패턴 매칭에 실패합니다.

슬슬 어떤 느낌인지 감이 올 것이라 생각합니다. 한 단계 더 나아가봅시다.

## 응용

{% raw %}
```erlang
1> Person = {person, {name, {{first, "Brie"}, {last, "Larson"}}}, {height, 170}}.
{person,{name,{{first,"Brie"},{last,"Larson"}}},
        {height,170}}
2> {person, FullName, Height} = Person.
{person,{name,{{first,"Brie"},{last,"Larson"}}},
        {height,170}}
3> FullName.
{name,{{first,"Brie"},{last,"Larson"}}}
4> {_, {{_, First}, {_, Last}}} = FullName.
{name,{{first,"Brie"},{last,"Larson"}}}
5> First.
"Brie"
6> Last.
"Larson"
```
{% endraw %}

1. `Person`이라는 변수에 캡틴 마블의 주인공을 맡은 브리 라슨과 관련된 정보의 tuple을 할당했습니다. 변수엔 단순 숫자와 같은 단일 값만 할당할 수 있는 게 아니라, 패턴만 일치한다면 얼랭의 어떠한 데이터 타입도 할당시킬 수 있습니다.
2. 이러한 특성을 이용해 `Person`에 담긴 tuple과 패턴 매칭을 시켜 `FullName`에는 이름 정보가 담긴, `Height`에는 키 정보가 담긴 tuple을 각각 할당했습니다.
3. 2번에서 할당한 `FullName`의 값을 확인해봅니다. `FullName`은 `Person` 두 번째 원소에 있던 이름과 관련된 tuple을 담고 있습니다.
4. `FullName`에서 성과 이름을 변수 `First`와 `Last`에 가져옵니다. 패턴 매칭을 통해 특정 위치의 값만 변수에 할당하고, 다른 값들은 무시하고 싶을 때는 다음과 같이 무시하고 싶은 값이 위치한 자리를 언더스코어(`_`)로 표시합니다. 이러한 언더스코어(`_`)는 익명 변수(anonymous variable)라고 불립니다.
5. `First`에 할당된 값을 확인합니다.
6. `Last`에 할당된 값을 확인합니다.

```erlang
1> [Head | Tail] = [1,2,3,4,5].
[1,2,3,4,5]
2> Head.
1
3> Tail.
[2,3,4,5]
4> {A, A, B, C} = {1, 1, 2, 3}.
{1,1,2,3}
```

1. 이런 패턴 매칭 특성을 리스트에 이용할 수도 있습니다. 지난시간에 리스트의 head와 tail은 `[H|T]`로 표현할 수 있다고 했습니다. 잊어먹었어도 괜찮습니다. 지금 다시 상기하면 됩니다. 해당 표현식과 연계해서 리스트의 첫 번째 값을 변수 `Head`에, 나머지 값들을 `Tail`에 할당할 수 있습니다.
2. `Head`에는 첫 번째 원소 `1`이 할당되었습니다.
3. `Tail`에는 첫 번째 원소 `1`을 제외한 나머지 값들이 할당되었습니다.
4. 이건 재미있는 특성을 소개하기 위해 추가했습니다. 패턴 매칭이 이루어질 때, 값이 동일하다면 좌측 변수를 여러번 적어도 문제없습니다.

## 마무리

수고하셨습니다. 얼랭의 패턴 매칭에 대해 무사히 익혔습니다!  
간단하면서도 차별화되는 중요한 특징입니다. 얼랭의 모든 코드 작성은 패턴 매칭으로 이루어졌다고 생각해도 될 정도로 자주 사용될 것입니다.

다음 포스트에서는 얼랭의 모듈에 대해 알아봅시다.

## References

- [Erlang Reference Manual](http://erlang.org/doc/reference_manual/patterns.html)
