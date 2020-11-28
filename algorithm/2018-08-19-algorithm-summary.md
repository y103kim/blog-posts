---
layout: post
category: algorithm
title: Algorithm 정리
modified: 2018-07-15
tags: [algorithm, 알고리즘, Summary]
---

# Data Structures
--------------------------------

## Array

- [검색(Binary Search)](http://doocong.com/algorithm/binary-search/)과 정렬(Merge Sort)
- 삽입과 삭제 불가능

## Hash

- 삽입/삭제/검색이 모두 O(1) 상수시간
- 정렬/Indexing 불가능.

## Stack and Queue

- Stack(DFS), Queue(BFS)

## Linked List

- Josepush 문제에서만 사용

## Search Tree

- STL: RB-tree, 삽입/삭제/검색 O(logN)
- Non STL: Binary-tree, 삽입/삭제/검색 O(logN), worstcast O(N)

## Heap

- STL: priority queue, 피보나치 Heap, push O(1) / pop O(logN)
- Non STL: Binary Heap, [구현](http://doocong.com/algorithm/max-heap/), push O(logN) / pop O(logN)

## Graph

- 인접 배열: Node의 수가 작고, Edge의 수가 많을 때
- 인접 리스트: Node의 수가 일정 이상 많아지면 무조건
