## 벨만-포드 알고리즘

**`최단 경로(Shortest Path)`** 를 구하는 대표적인 알고리즘에는 **`다익스트라(Dijkstra's)`** 알고리즘과 **`벨만-포드(Bellman-Ford's)`** 알고리즘이 있다.

둘의 차이점을 아주 간단하게 비교하자면 

- 다익스트라`O(ElogV)`가 벨만-포드`O(VE)`보다 시간복잡도가 빠르다.

- 다익스트라 알고리즘과 달리 벨만-포드 알고리즘은 가중치가 **음수**인 경우에도 적용 가능하다. $($단 음수 가중치가 사이클을 이루지 않는 경우 해당.)

즉, 가중치가 음수인 케이스를 포함하는 문제의 경우, 최단 경로를 찾기 위해서는 **벨만-포드** 알고리즘을 사용해야한다.

## 벨만-포드 알고리즘 개념
시작노드`s`에서 `v`에 이르는 최단경로는 `s`에서 임의의 지점 `u`까지의 최단경로에 `u`에서 `v`사이의 가중치를 더한 값과 같다.

> **`D(s, v) = D(s, u) + w(u, v)`**

벨만-포드 알고리즘은 `s, u` 사이의 최단경로를 구할 때 그래프 내 모든 간선에 대해 `edge relaxation`을 수행해준다.

> **edge relaxation** : 정점을 하나 선택하고 선택된 정점까지 오는 간선의 가중치를 선택된 정점과 연결된 모든 간선에 더해준다.

이때 벨만-포드 알고리즘은 모든 간선에 대한 edge relaxation을 **`|V| - 1`** 번 수행한다.

`s, u` 사이의 최단경로가 `s`와 `u`뿐일 수도 있지만 `u`를 제외한 그래프의 모든 노드 `(|V| - 1개)`가 `s, u` 사이의 최단경로를 구성할 수도 있기 때문이다.

## 벨만-포드 알고리즘 수행과정
a ~ d의 과정은 그래프 모든 간선에 대한 edge relaxation을 1회 수행한 것이며 `(e)`의 경우 2회 수행에서 갱신된 결과이다.

이러한 과정을 `|V| - 1`번 수행하며 더이상 갱신될 값이 없는지 확인한다.

수행할 때 마다 거리정보와 최단경로가 업데이트 되는것을 확인할 수 있다.

![bellmanford](https://i.imgur.com/hcWT22F.png)

<small> 출처- https://ratsgo.github.io/data%20structure&algorithm/2017/11/27/bellmanford/ </small>

**$($a)**

먼저 시작노드 `s`를 제외한 모든 노드의 거리를 무한대로 초기화 한다.

**$($b)**

`(s, y)`의 경우 **s**에서 **y**에 이르는 거리가 7이고, 이는 기존 거리$($무한대)보다 작으므로 7 `(0 + 7)`을 y에 기록해 둔다.

`(s, t)`또한 마찬가지로 6을 t에 기록해 둔다.

**$($c)**

`(y, x)`의 경우 `7 + (-3) = 4`이고 이는 기존 거리$($무한대) 보다 작으므로 4를 x에 기록해 둔다.

`(t, z)`또한 마찬가지로 2를 z에 기록해 둔다.

**$($d)**

`(x, t)`의 경우 `4 + (-2) = 2`이고 기존거리 6보다 작으므로 2를 t에 기록해 둔다.

a ~ d 까지 edge relaxation 1회 였으며 모든 간선을 확인하며 갱신해야 될 값은 갱신해주었다.
$($갱신되지 않아도 되는 케이스는 설명에서 스킵하였다.)

**$($e)**
edge relaxation이 2회 수행되며 다시 모든 간선을 확인해 가면서 갱신될 값이 있으면 값을 갱신해준다.

`(t, z)`의 경우 `2 + (-4) = -2`이고 이는 기존거리 2보다 작으므로 -2를 z에 기록해 둔다.

`|V| - 1`번 edge relaxation을 반복하면서 거리값을 전부 갱신해준다.

## Negative cycle
다익스트라 알고리즘과 달리 벨만-포드 알고리즘은 **가중치가 음수**여도 적용가능하다.

그러나 아래와 같이 음수 가중치가 사이클$($cycle)을 이룰 경우 작동하지 않는다.

![bellmanford](https://i.imgur.com/46tJqd7.png)

<small> 출처 - https://ratsgo.github.io/data%20structure&algorithm/2017/11/27/bellmanford/ </small>

위 그림에서 `(c, d)`, `(e, f)`가 사이클을 이루고 있다.

c, d의 경우 사이클을 돌 때마다 거리가 커지기 때문에 최단경로를 구할 때 문제가 되지 않는다.

반면 e, f의 경우 사이클이 돌수록 거리가 짧아지기 때문에 벨만-포드 알고리즘으로 최단경로를 구하는것 자체가 의미가 없어진다.

따라서 edge relaxation을 `|V| - 1`번 반복 수행한 뒤, 마지막으로 그래프 모든 간선에 대해 edge relaxation을 1번 더 수행해 준다.

이때 한번이라도 값이 갱신된다면 negative cycle이 존재한다는 의미이기 때문에 결과를 구할 수 없다는 의미의 false를 반환하고 함수를 종료하게 된다.

## 벨만-포드 알고리즘 코드 $($Python)

```python
import sys
input = sys.stdin.readline

INF = int(1e9) # 무한대 값

# 노드, 간선의 개수 입력
v, e = map(int, input().split())
# 모든 간선에 대한 정보를 담는 리스트
edges = []

# 최단거리 테이블을 무한대로 초기화
distance = [INF] * (v + 1)

# 모든 간선의 정보 입력
for _ in range(e):
    sv, ev, cost = map(int, input().split())
    edges.append((sv, ev, cost))

# 벨만-포드 알고리즘
def bellmanFord(start):
    # 시작 노드에 대해서 초기화
    distance[start] = 0
    # v번 edge relaxation을 반복.
    # v - 1번 탐색하고 마지막 한번은 Negative cycle 존재 확인
    for i in range(v):
        # 매 반복마다 모든 간선을 확인하며 갱신
        for j in range(e):
            curNode, nextNode, edgeCost = edges[j]
            # 현재 간선을 거쳐서 다른 노드로 이동하는 거리가 더 짧은 경우
            if distance[curNode] != INF and distance[curNode] + edgeCost < distance[nextNode]:
                distance[nextNode] = distance[curNode] + edgeCost
                # v번째 반복에서 갱신되는 값이 있으면 Negative cycle 존재
                if i == v - 1:
                    return False

    # 벨만-포드 정상종료
    return True

if bellmanFord(1):
    # 1번 노드를 제외한 다른 모든 노드로 가기 위한 최단거리를 출력
    for i in range(2, v + 1):
        # 도달할 수 없는 경우
        if distance[i] == INF:
            print("도달할 수 없다.")
        else:
            print(distance[i])
else:
    print("Negative Cycle Exist")
```

## 시간복잡도
시작 노드를 제외한 모든 정점  $($V - 1), 그리고 negative cycle을 확인하기 위해 한번 더 반복하기 때문에 총 V번 반복에 대해서 해당 정점과 연결된 모든 간선$($E)를 탐색한다.

따라서 `O(VE)`

---

> 참조 <br>
> - [kdb.velog](https://velog.io/@kimdukbae/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%B2%A8%EB%A7%8C-%ED%8F%AC%EB%93%9C-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-Bellman-Ford-Algorithm) <br>
> - [ratsgo's blog](https://ratsgo.github.io/data%20structure&algorithm/2017/11/27/bellmanford/) <br>
