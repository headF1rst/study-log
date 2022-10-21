---
toc: true
title: "백트래킹 알고리즘[파이썬코드]"
category:
  - Algorithm
---
## 백트래킹의 정의

주어진 후보군들을 하나씩 따라 들어가며 탐색하다가 해당 후보군이 제약 조건을 만족할 수 없다고 판단되는 즉시, backtrack $($이전 후보군으로 퇴각)하고, 최적의 해를 찾는 방법.

## 백트래킹 구현

고려할 수 있는 모든 후보군을 상태공간트리로 표현.

상태공간트리의 각 후보군을 DFS로 탐색하면서 제약 조건에 맞지 않으면 이전 노드로 퇴각해서 계속 탐색. $($가지치기)

![백트래킹 트리](https://i.imgur.com/rqaX8om.png)

DFS로 a의 인접 노드인 b를 탐색해 나가다가 c노드가 제약조건에 맞지 않으면 c이후의 노드는 더이상 탐색할 필요가 없다. (가지치기)

a노드로 backtrack해서 d부터 탐색을 이어간다.

## 백트래킹 파이썬코드 - N과 M

```python
input = sys.stdin.readline

n, m = map(int, input().split())
ans = []
visited = [False] * (n + 1)

def dfs():
    if len(ans) == m:
        print(' '.join(map(str, ans)))
        return

    for i in range(1, n + 1):
        if not visited[i]:
            visited[i] = True
            ans.append(i)
            dfs()
            j = ans.pop()
            visited[j] = False
    return

dfs()
```