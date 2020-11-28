---
layout: post
category: algorithm
title: Union-Find
modified: 2017-11-21
tags: [algorithm, union-find, disjoint set]
---

## Union-Find

union-find는 서로소 관계의 집합(disjont set)을 표현하기 위한 알고리즘이다. 다음 과정을 통해서 union-find를 수행한다.

1. 각 Node의 부모를 저장하는 배열로 Tree를 표현하는 자료구조를 사용한다.

    예를 들어 1-2, 1-3, 2-4으로 이어진 Tree는 아래와 같이 나타낸다. 부모가 없는 최상위 node는 배열에 값에 자기 번호를 입력한다.

    [![](/images/012/1.png)](/images/012/1.png)

2. Edge로 연결되어 있는 node는 같은 집합에 포함되어 있음을 나타낸다. 즉 최상위에 있는 부모가 같다면 같은 집합에 속해 있다.

    아래 예에서는 1,2,3,4 는 최상위 부모가 1로 같은 집합에 속하지만 5,6은 다른 집합에 속한다.

    [![](/images/012/2.png)](/images/012/2.png)

    최상위 부모를 찾는 코드로는 아래와 같이 재귀적으로 작성한다

   ```c++
   int find(int x) {
       return (parent[x] == x) ? parent[x] : find(parent[x]);
   }
   ```

3. 합집합 연산은 다음과 같이 가능하다. 위의 예에서 두 집합을 합치 때는 각 집합의 최상위 부모를 하나로 통일하면 된다.

    [![](/images/012/3.png)](/images/012/3.png)

    코드로는 아래와 같다.

   ```c++
   void uni(int a, int b) {
       parent[find(b)] = find(a);
   }
   ```

### Tree 압축

위 구현은 node의 수가 많아지면 트리의 depth가 늘어나서, 최상위 부모를 찾는 과정이 점점 느려진다. Best case는 o(1)이지만 Worst는 O(n)이다.

위 문제는 find 함수에서 자녀 node들을 전부 최상위 부모 node 바로 아래로 옮겨주면 해결된다.

```c++
int find(int x) {
    return parent[x] = (parent[x] == x) ? parent[x] : find(parent[x]);
}
```

위 코드는 거의 O(1)에 가까운 것이 알려져 있다.

### Union-Find의 한계점

어디까지나 Disjont set에 국한된 구현으로 아래와 같은 한계점이 있다.

* 교집합이 존재하는 집합을 표현하지 못한다.
* 두 집합을 합하는 것만 가능하다. 집합을 쪼개는 동작이 불가능하다
* 그래프에서 두 노드가 이어져 있는지 확인할 때 이용 가능하지만,
    어떤 edge를 없앨 때, 여전히 이어져 있는지는 확인이 불가능하다.


