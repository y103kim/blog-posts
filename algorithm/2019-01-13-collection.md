---
layout: post
category: algorithm
title: 알고리즘 박제
modified: 2019-01-31
tags: [algorithm]
---

## 추가 정리가 필요한 리스트

그래프
 - DFS Iterative, Recursive
 - BFS Iterative, Recursive
 - Topological Sort
 - Detect Cycle (Backedge, Indegree)
 - SCC
 - Union-Find, MST
 - 오일러 회로, 경로

정렬 
 - Merge-sort

헤싱
 - Hash Table

문자열
 - KMP
 - Manacher

쿼리 자료구조
 - SQRT Decomposition (평방 분할)

## Pow(b,e), Fact(n), Comb(n,r)

페르마의 소정리, 분할정복 사용.

```c++
LL fact(int x){
  for(int i = saved+1; i <= x; i++)
    dp[i] = (dp[i-1] * i) % MOD;
  saved = max(x, saved);
  return dp[x];
}
 
LL pow(LL b, LL e) {
  LL ans = 1;
  while(e) {
    if(e&1) ans = (ans*b)%MOD;
    b = (b*b)%MOD;
    e /= 2;
  }
  return ans;
}
 
LL comb(int n, int r) {
  if(n <= r || r == 0) return 1;
  LL a = fact(n);
  LL b = (fact(r) * fact(n-r)) % MOD;
  LL inv = pow(b, MOD-2);
  return (a * inv) % MOD;
}
```

## Trie

```c++
struct Trie {
  Trie* next[26]; bool isEnd;
} root;

void insert(string& s) {
  Trie* cur = &root;
  for(char c: s) {
    Trie* &next = cur->next[c-'a'];
    // If next is not initialized, then make it.
    cur = next ? next : next = new Trie();
  }
  cur->isEnd = true;
}

bool find(string& s) {
  Trie* cur = &root;
  for(int j = 0; j < s.size(); j++) {
    // 1. Most important thing : 먼저 진행한 뒤에
    cur = cur->next[c-'a']; 
    // 2. Todo Some thing ...
    if(!cur) return false; // Leaf of the Tree
  }
  return cur->isEnd; // Check there is target string in Trie.
}
```

## Quick Select

```c++
int partiton(vector<int>& v, int s, int e) {
  swap(v[e], v[s + rand() % (e-s+1)]);
  int t = s;
  for(int i = s; i < e; i++)
    if(v[i] < v[e])
      swap(v[i], v[t++]);
  swap(v[t], v[e]);
  return t;
}

void qselect(vector<int>& v, int k, int s, int e) {
  if(s == e) return;
  int p = partition(v, s, e);
  if(p == k) return;
  else if( p < k ) qselect(v, k, p+1, e);
  else qselect(v, k, s, p-1);
}
```

## Quick Sort

Partiton은 Quick Select와 공유

```c++
void qsort(vector<int>& v, int s, int e) {
  if(s == e) return;
  int p = partition(v, s, e);
  if( p < e ) qsort(v, p+1, e);
  if( s < p ) qsort(v, s, p-1);
}
```


## radix sort

음수는 UB를 더해서 양수로 만들어 넣어야 한다.
C의 짝/홀에 따라서 결과가 기록되는 Array가 다르다.

```c++
const int R = 7;
const int C = 3;
const int P = 1 << R;
const int MASK = P - 1;
int N;
#define KEY (a[i] >> k) & MASK
#define DKEY MASK-((a[i] >> k) & MASK)

void rsort(int* a, int* b) {
    int cnt[P], *t;
    for(int k = 0; k < C*R; k += R) {
        for(int i = 0; i < P; i++) cnt[i] = 0;
        for(int i = 0; i < N; i++) cnt[DKEY]++;
        for(int i = 1; i < P; i++) cnt[i] += cnt[i-1];
        for(int i = N-1; i >= 0; i--) b[--cnt[DKEY]] = a[i];
        t = a, a = b, b = t; 
    }
}
```


## Binary Search

f에는 적절한 함수를 넣는다.
 - ub를 v.size()를 하는 이유는 전부 false를 인 경우를 위해
 - lb가 m+1인 이유는 /2 연산이 항상 버림이기 때문

```c++
int bs(vector<int>& v, int k) {
  int lb = 0, ub = v.size();
  while(lb < ub) {
    int m = (lb + ub) / 2;
    if(f(m)) ub = m;
    else lb = m+1;
  }
  return lb;
}
```

## Segment Tree

segment Tree는 RMQ(Ranged Minimum Query)를 구하는데 좋다.
다만 Ranged Update에는 불리해 보인다.

```c++
class segmentTree {
public :
  vector<long long> data;
  int dataSize;

  segmentTree(int N) : dataSize(N), data(2*N) {
    for (int i = 0; i < N; ++i)
      data[N + i] = fio::readInt();
    for (int i = dataSize - 1; i > 0; --i)
      data[i] = data[2*i] + data[2*i+1];
  }

  void update(int p, long long v)
  {
    for (data[p += dataSize] = v; p > 1; p /= 2) 
      data[p/2] = data[p] + data[p ^ 1];
  }

  long long query(int l, int r)
  {
    long long ans = 0;
    for (l += dataSize, r += dataSize; l < r; l /= 2, r /= 2) {
      if (l & 1) ans += data[l++];
      if (r & 1) ans += data[--r];
    }
    return ans;
  }
};
```

## Fenwick Tree

펜윅 트리는 두가지 경우에 사용한다.
1. sum[0:N] 이 필요할 때, diff 쿼리가 가능할 경우
2. max[0:N] 이 필요할 때, query가 오름차순으로 들어오는 경우

``` c++
vector<int> tree;
// sum version
void update(int p, int v) {
   while(p > 0) {
     tree[p] = (tree[p], v);
     p -= (p & -p);
   }
}

int query(int p) {
   int ans = 0;
   while(p < tree.size()) {
     ans = max(ans, tree[p]);
     p += (p & -p);
   }
   return ans;
}

// maximum version
void update(int p, int diff) {
   while(p < tree.size()) {
     tree[p] += diff;
     p += (p & -p);
   }
}

int query(int p) {
   int ans = 0;
   while(p > 0) {
     ans = += tree[p];
     p -= (p & -p);
   }
   return ans;
}
```

