---
title: "프로그래머스 문제풀이 고득점 Kit 해시 - 베스트앨범"
date: 2018-10-14 20:29:54 +0900
categories: programmers 고득점Kit
tags: programmers
layout: post
comments: true
pinned: true
---

## 문제
> 스트리밍 사이트에서 장르 별로 가장 많이 재생된 노래를 두 개씩 모아 베스트 앨범을 출시하려 합니다. 노래는 고유 번호로 구분하며, 노래를 수록하는 기준은 다음과 같습니다.
> 
> 속한 노래가 많이 재생된 장르를 먼저 수록합니다.  
> 장르 내에서 많이 재생된 노래를 먼저 수록합니다.  
> 장르 내에서 재생 횟수가 같은 노래 중에서는 고유 번호가 낮은 노래를 먼저 수록합니다.  
> 노래의 장르를 나타내는 문자열 배열 genres와 노래별 재생 횟수를 나타내는 정수 배열 plays가 주어질 때, 베스트 앨범에 들어갈 노래의 고유 번호를 순서대로 return 하도록 solution 함수를 완성하세요.
>
> 제한 사항
> - genres[i]는 고유번호가 i인 노래의 장르입니다.
> - plays[i]는 고유번호가 i인 노래가 재생된 횟수입니다.
> - genres와 plays의 길이는 같으며, 이는 1 이상 10,000 이하입니다.
> - 장르 종류는 100개 미만입니다.
> - 장르에 속한 곡이 하나라면, 하나의 곡만 선택합니다.
> - 모든 장르는 재생된 횟수가 다릅니다.
>
> 입출력 예
> 
> | genres | plays | return |
> |----|----|----|
> | ["classic", "pop", "classic", "classic", "pop"] | [500, 600, 150, 800, 2500] | [4, 1, 3, 0] |
>
> 입출력 예 설명
> 
> classic 장르는 1,450회 재생되었으며, classic 노래는 다음과 같습니다.
> 
> 고유 번호 3: 800회 재생  
> 고유 번호 0: 500회 재생  
> 고유 번호 2: 150회 재생  
> pop 장르는 3,100회 재생되었으며, pop 노래는 다음과 같습니다.
>
> 고유 번호 4: 2,500회 재생  
> 고유 번호 1: 600회 재생  
> 따라서 pop 장르의 [4, 1]번 노래를 먼저, classic 장르의 [3, 0]번 노래를 그다음에 수록합니다.

Link: [https://programmers.co.kr/learn/courses/30/lessons/42579](https://programmers.co.kr/learn/courses/30/lessons/42579)

## 풀이
이번 문제는 다른 문제에 비하여 조건이 조금 많다. 입력 또한 장르 배열과, 재생 횟수 배열이 따로따로 들어온다.  
하지만, 차례차례 필요한 값을 구해주면 생각보다 어렵지 않은 문제다. 조건에 따라 몇 가지만 구하고 처리하면 된다.

1. 장르별로 그룹 짓기
    장르별 전체 재생 횟수, 재생 횟수 기준 순위를 구해야 하므로, 장르별로 그룹 지어야 한다.  
2. 장르별 전체 재생 횟수 구하기  
    출력 우선순위의 1순위가 장르(장르별 전체 재생 횟수에 따른), 2순위가 각 장르 내 재생 횟수이므로, 그룹을 지어주며 각 장르의 전체 재생 횟수를 구한다.
3. 정렬
    위와 마찬가지로 모든 그룹이 완성되면 그룹(장르) 순서 정렬, 그룹 내 순서 정렬을 진행한다.

## 해답
이번에는 Python3를 이용해서 작성해봤다.
```python
def solution(genres, plays):
    '''
    각 노래의 grouping을 진행할 변수를 선언해준다.
    다음과 같은 구조를 가지게 된다.
    {
        '장르이름': {
            't': 전체 재생 시간
            'l': [노래정보리스트, {'v': 재생시간, 'i': 고유번호}]
        }
    }
    '''
    d = {}
    # 노래 갯수 만큼 loop를 돌며 grouping을 진행한다.
    for i in range(len(genres)):
        # 장르에 해당하는 key가 없으면 새로 만들어준다.
        if not genres[i] in d:
            d[genres[i]] = {'t':0, 'l':[]}
        d[genres[i]]['l'].append({'v':plays[i], 'i':i})
        d[genres[i]]['t'] += plays[i]

    # 정렬을 위해 list 형태로 바꿔준다.
    # 앞으로의 연산이나, answer에는 장르 이름이 필요없기에 문제없다.
    d = list(d.values())
    d = sorted(d, key=lambda k: k['t'], reverse=True) 
    answer = []
    # 각 장르별 재생 횟수를 정렬하고, answer를 구한다.
    for i in range(len(d)):
        v = d[i]['l']
        v = sorted(v, key=lambda k: k['v'], reverse=True)
        for j in range(2 if len(v) >= 2 else len(v)):
            answer.append(v[j]['i'])
    return answer
```