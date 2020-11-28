---
layout: post
category: algorithm
title: 알고리즘 문제 분류
modified: 2018-01-31
tags: [algorithm]
---

## Stack

[**괄호짝 맞추기**](https://leetcode.com/problems/valid-parentheses/)

- 어려운게 없다. 열린 괄호면 넣고, 닫힌 괄호면 빼면 된다.

[**수족관 시리즈 1 : 42. Trapping Rain Water (leetcode)**](https://leetcode.com/problems/trapping-rain-water/)

1. leftmax와 rightmax를 저장하는 dp 형식
2. stack을 사용하는 형식
3. 양쪽에서 더 큰 쪽을 좁혀오면, 반대쪽에서는 오는 방향만 볼 수 있음. 예를 들어서 오른쪽 끝에서 10인 벽이 있었다면, 왼쪽에서는 10 이하인 놈들에 대해서 안심하고 leftmax만 보면서 진행하면 됨.

[**수족관 시리즈 2 : 8982. 수족관 (BOJ)**](https://www.acmicpc.net/problem/8982)

1. stack을 사용하는 방식으로 풀었다. 
   - 감소할 경우 진행하고, 증가할 경우 하나의 entry로 머지한다. 
   - 구멍이 없을 경우에만 merge한 부분을 답에 더해주면 된다.
2. 분할 정복으로도 풀린단다. (안풀어봄)

[**456. 132 Pattern**](https://leetcode.com/problems/132-pattern/)

- 두가지 키포인트는 leftmin을 저장하는 점과, leftmin보다 더 작은 수들은 사용할 일이 없다는 점이다. 아래와 같이 풀자.
  - 감소 할 때 push하면서 쌓다가, leftmin 보다 작은수는 pop 한다.
  - 다음이 성립하면 된다. ```leftmin[i-1] < stack.top() < v[i]```
 
[**1725. 히스토그램**](https://www.acmicpc.net/problem/1725), [**6549. 히스토그램에서 가장 큰 직사각형**](https://www.acmicpc.net/problem/6549)

- 스택으로 풀린다. top보다 더 작은게 쌓인다면 뒤에서 top을 쓸 일이 없다.
- 풀이를 알고서도 엄청 헤멧는데, 생각을 깊게 하지 않고 풀어서다.
- 주의하자!

## Tree DP

[**사회망 서비스**](https://www.acmicpc.net/problem/2533)

- 아래의 Tree Camera 문제랑 매우 유사함
- 과거 풀어놓은거 보변 근데 BFS로 순서 정한뒤에 Child부터 위로 누적해 올라가는 식으로 풀었음.

[**968. Binary Tree Cameras**](https://leetcode.com/problems/binary-tree-cameras/)

- 처음 풀어본 Tree DP, TreeNode*를 key로한 HashMap을 DP 배열로 사용
- 걍 세세하게 상황 나눠가면서 DP 진행하면 풀림
- DFS는 종료조건을 먼저 체크해야 코드가 간단해짐
- Greedy로 풀수 있다는데 Greedy가 제일 어렵다 -_-.. 나중에 정리해야지.

[**124. Binary Tree Maximum Path Sum**](https://leetcode.com/problems/binary-tree-maximum-path-sum)

- Tree판 Kanade이다. 왼쪽 오른쪽 subtree의 sum을 0과 비교한다.
- 현재 value 기준으로 left와 right를 붙여서, max를 매번 구한다.
- 부모에게 올려줄 Return은 왼쪽, 오른쪽, 또는 둘다 안쓰는 경우 중 최대
- Kanade와 동일하게, 전부 음수가 들어오는 경우는 특수 처리를 해줘야함.