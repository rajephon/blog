---
title: "프로그래머스 문제풀이 level3 - 야근 지수"
date: 2018-10-14 19:09:54 +0900
categories: programmers
layout: post
comments: true

---

## 문제
> 회사원 Demi는 가끔은 야근을 하는데요, 야근을 하면 야근 피로도가 쌓입니다. 야근 피로도는 야근을 시작한 시점에서 남은 일의 작업량을 제곱하여 더한 값입니다. Demi는 N시간 동안 야근 피로도를 최소화하도록 일할 겁니다. Demi가 1시간 동안 작업량 1만큼을 처리할 수 있다고 할 때, 퇴근까지 남은 N 시간과 각 일에 대한 작업량 works에 대해 야근 피로도를 최소화한 값을 리턴하는 함수 solution을 완성해주세요.
>
> 제한 사항
> - works는 길이 1 이상, 20,000 이하인 배열입니다.
> - works의 원소는 50000 이하인 자연수입니다.
> - n은 1,000,000 이하인 자연수입니다.
>
> 입출력 예
> 
> | works | n | result |
> |----|----|----|
> | [4, 3, 3] | 4 | 12 |
> | [2, 1, 2] | 1 | 6 |
> | [1,1] | 3 | 0 |
>
> 입출력 예 설명
> 
> 입출력 예 #1
> n=4 일 때, 남은 일의 작업량이 [4, 3, 3] 이라면 야근 지수를 최소화하기 위해  4시간동안 일을 한 결과는 [2, 2, 2]입니다. 이 때 야근 지수는 22 + 22 + 22 = 12 입니다.
>
> 입출력 예 #2
> n=1일 때, 남은 일의 작업량이 [2,1,2]라면 야근 지수를 최소화하기 위해 1시간동안 일을 한 결과는 [1,1,2]입니다. 야근지수는 12 + 12 + 22 = 6입니다.
>
> 입출력 예 #3

Link: [https://programmers.co.kr/learn/courses/30/lessons/12927](https://programmers.co.kr/learn/courses/30/lessons/12927)

## 풀이
요령을 알면 쉽고, 모르면 어려운 문제다. 핵심은 `야근 피로도`다.  
남은 일의 작업량의 제곱을 더한 값이 야근 피로도가 되므로, 피로도를 최대한 낮추려면 작업량의 최댓값을 최대한 낮춰야 한다. 그러려면,

`works`에서 가장 큰 값을 찾고  
해당 아이템과 n에서 1을 빼고  
`n`이 0보다 클 경우? `works`에서 가장 큰 값을 찾고  
....

이 과정을 반복해야 하는데, 매 루프마다 가장 큰 값을 새로 찾는 과정에 대한 비용은 비싸다. 이 비용을 고려하지 않고 코딩하면, 이 부분 때문에 효율도 테스트에서 실패한다.

하지만, 이럴 때 우선순위 큐를 사용하면 매번 새로 찾을 필요가 없어 효율도를 크게 높일 수 있다.

## 해답

### C++
```cpp
#include <vector>
#include <queue>
using namespace std;

long long solution(int n, vector<int> works) {
    // std::priority_queue를 정의하고, works에 들어온 작업들을 옮겨담는다.
    std::priority_queue<int> q(works.begin(), works.end());
    // 작업시간 n 만큼 돌며, 작업들의 최댓값에서 1씩 뺀다.
    for (int i = 0; i < n; i++) {
        if (q.top() > 0) {
            // 큐에서 최댓값을 가져오고, -1 해서 다시 넣는다.
            int tmp = q.top();
            q.pop();
            q.push(tmp-1);
        }
    }
    // 마지막으로, 남은 작업량으로 피로도를 계산한다.
    long long answer = 0;
    while(!q.empty()) {
        answer += (long long)q.top() * (long long)q.top();
        q.pop();
    }
    return answer;
}
```


## 2018-10-16 수정 : Python3 풀이 추가

### Python3
```python
import heapq

def solution(n, works):
    for i in range(len(works)):
        works[i] *= -1
    heapq.heapify(works)

    for i in range(0, n):
        m = heapq.heappop(works)
        if m >= 0:
            heapq.heappush(works, m)
            break
        m += 1
        heapq.heappush(works, m)

    answer = 0
    while len(works) > 0:
        m = heapq.heappop(works)
        answer += pow(m * -1, 2)
    return answer
```
Python3 에서는 heapq를 이용해봤다. `heapq.heappop(heap)`가 가장 작은 값을 반환하는데, 우리는 가장 큰 값을 꺼내고 다시 넣는 행동을 해야 하므로, 편의를 위해 입력 값에 전부 -1을 곱해 음수로 보관했다. 그러므로 남은 작업량을 제곱하기 전에 다시 -1을 곱해 양수로 바꿔준다.

## Reference
[https://en.cppreference.com/w/cpp/container/priority_queue](https://en.cppreference.com/w/cpp/container/priority_queue)  
[https://docs.python.org/3/library/heapq.html](https://docs.python.org/3/library/heapq.html)
