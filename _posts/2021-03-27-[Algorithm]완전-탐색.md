---
layout: post
title: "[Algorithm]완전 탐색 이란?"
date: 2021-03-27
excerpt: "무식해보여도 사실은 최고의 방법일때가 있지요."
tags: [Algorithm, Brute-force, BFS, DFS]
comments: true
---
# 완전 탐색 이란?

컴퓨터의 빠른 계산 능력을 이용하여 가능한 경우의 수를 일일이 나열하면서 답을 찾는 방법을 의미한다. '무식하게 푼다'라는 의미인 Brute-Fore, Exhaustive Search라고 한다. 가능한 모든 경우의 수를 다 해보는 것이기에 알고리즘 풀 때 가장 강력하고 확실한 방법이지만 그만큼 시간이 가장 오래 걸리는 탐색 기법이다. 


# 완전 탐색 기법의 종류

- Brute Force : for문과 if문을 이용하여 처음부터 끝까지 탐색하는 방법
- 비트 마스크 : 이진수 표현을 자료구조로 쓰는 기법(AND, OR, XOR, SHIFT, NOT) , 문제에서 나올 수 있는 모든 경우의 수가 각각의 원소가 포함되거나, 포함되지 않는 두 가지 선택으로 구성되는 경우에 유용하게 사용이 가능
- 재귀 함수 : 비트마스크와 마찬가지로 주로 각 원소가 포함되거나, 포함되지 않는 두 가지 선택을 가질 때 이용된다.
- 순열 : 완전 탐색의 대표적인 유형. 서로 다른 n개의 원소에서 r개의 중복을 허용하지 않고 순서대로 늘어 놓은 수
- BFS(너비우선탐색), DFS(깊이우선탐색)

기본적으로 완전탐색은 N의 크기가 작을 때 이용이 되기 때문에 보통 시간 복잡도가 지수승이나, 팩토리얼 꼴로 나올 때 많이 이용된다. 

## BFS(Breadth-First Search)

너비 우선 탐색, 최대한 넓게 이동한 다음, 더 이상 갈 수 없을 때 아래로 이동 

<img src ="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc305k7%2FbtqB5E2hI4r%2Fea7vFo08tkDYo4c8wkfVok%2Fimg.gif">

루트 노드(혹은 임의의 노드)에서 시작해서 인접한 노드를 먼저 탐색하는 방법으로, 시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어져 있는 정점을 나중에 방문하는 순회 방법이다. 

주로 두 노드 사이의 최단 경로를 찾고 싶을 때 이 방법을 선택한다. 

### BFS-Trees

큐가 한개 이상의 노드를 가지고 있는 이상 아래 방법을 반복한다. 

- 큐에서 첫번째 노드를 빼낸다.
- 만약 그 노드가 우리가 찾던 그 노드라면, 검색은 종료된다.
- 아니면 그 노드의 자식을 큐 마지막에 삽입하고 위의 방법을 반복 한다.

### BFS-Gaphs

그래프인 경우에는 방문한 노드의 collection을 유지해야하고 두번 방문하지 않도록 해야 한다.  그렇지 않으면 무한 루프에 빠지게 될 수 있다. 

- 큐에서 첫번째 노드를 빼낸다.
- 노드에 방문했는지 확인한다, 방문 했으면 스킵한다.
- 만약 그 노드가 우리가 찾던 그 노드라면, 검색은 종료된다.
- 아니면 그 노드를 방문한 노드들에 추가한다.
- 아니면 그 노드의 자식을 큐 마지막에 삽입하고 위의 방법을 반복 한다.

### 자바 코드

### Trees

```java
public class Tree<T> {
		private T value;
    private List<Tree<T>> children;

    private Tree(T value) {
        this.value = value;
        this.children = new ArrayList<>();
    }

    public static <T> Tree<T> of(T value) {
        return new Tree<>(value);
    }

    public Tree<T> addChild(T value) {
        Tree<T> newChild = new Tree<>(value);
        children.add(newChild);
        return newChild;
    }
}
```

싸이클을 만드는 걸 피하기 위해, 자식노드들은 주어진 값을 바탕으로 클래스에 의해 작성 되어야 한다. search(0 메소드를 만들어 보자. 

```java
public static <T> Optional<Tree<T>> search(T Value, Tree<T> root){
			//...
}
```

위에서 언급했듯이, BFS는 노드를 순회하기 위해 큐를 사용한다. 우선, root 노드를 큐에 추가해야 한다. 

```java
Queue<Tree<T>> queue = new ArrayDeque<>();
queue.add(root);
```

그리고 큐가 비어있지 않는동안 반복해야하고 매번 큐에서 노드를 빼낸다.

```java
while(!queue.isEmpty()){
		Tree<T> currentNode = queue.remove();
}
```

만약 그 노드가 우리가 찾고 있는 것이라면 return 하고 아니면 자식 노드를 큐에 추가 한다. 

```java
if(currentNode.getValue().equals(value)) {
		return Optional.of(currentNode);
} else {
		queue.addAll(currentNode.getChildren());
}
```

마지막으로, 우리가 찾고자 하는 것을 찾지 못하고 모든 노드를 방문 했다면 empty 결과를 반환한다. 

```java
return Optional.empty();
```

아래는 예제이다. 

<img src="https://www.baeldung.com/wp-content/uploads/2019/10/BFS-Tree-Example.png">

```java
Tree<Integer> root = Tree.of(10);
Tree<Integer> rootFirstChild = root.addChile(2);
Tree<Integer> depthMostChild = rootFirstChild.addChild(3);
Tree<Integer> rootSecondChild = root.addChild(4);
```

Value값 4를 찾는다고 할때, 10 → 2→ 4 순서로 순회할 것이다. 

```java
BreadthFirstSearchAlgorithm.search(4, root)
```

```java
c.b.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 10
c.b.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 2 
c.b.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 4
```

## DFS(Depth-First Search)

깊이 우선 탐색, 최대한 깊이 내려 간 뒤, 더이상 깊이 갈 곳이 없을 경우 옆으로 이동

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxC9Vq%2FbtqB8n5A25K%2FGyOf4iwqu8euOyhwtFuyj1%2Fimg.gif">

루트 노드(혹은 다른 임의의 노드)에서 시작해서 다음 분기(branch)로 넘어 가기 전에 해당 분기를 완벽하게 탐색하는 방식이다. 

1. 모든 노드를 방문하고자 하는 경우에 이 방법 선택
2. DFS가 BFS보다 좀더 간단

## Tree DFS

- 전위 순회 (Pre-Order Traversal) : root → left → right
- 중위 순회 (Inorder Traversal) : left → root → right
- 후위 순회 (Post-Order Traversal) : left → right → root

### 전위 순회 Pre-Order Traversal

root → left → right

재귀를 사용해서 전위 순회를 수행할 수 있다. 

- 현재 노드를 방문한다
- 왼쪽 하위노드를 순회한다
- 오른쪽 하위 노드를 순회한다

```java
public void traversePreOrder(Node node) {
    if (node != null) {
        visit(node.value);
        traversePreOrder(node.left);
        traversePreOrder(node.right);
    }
}
```

재귀 없이 전위 순회를 수행할 수 도 있다. 

반복적인 전위 순회를 하기 위해, 우리는 스택이 필요하다. 

- root를 stack에 push 한다
- stack이 empty가 아닌동안
    - 현재 노드를 pop
    - 현재 노드를 방문
    - 오른쪽 자식노드를 push 그리고 왼쪽 자식 노드를 stack으로

```java
public void traversePreOrderWithoutRecursion() {
    Stack<Node> stack = new Stack<Node>();
    Node current = root;
    stack.push(root);
    while(!stack.isEmpty()) {
        current = stack.pop();
        visit(current.value);
        
        if(current.right != null) {
            stack.push(current.right);
        }    
        if(current.left != null) {
            stack.push(current.left);
        }
    }        
}
```

<img src="https://upload.wikimedia.org/wikipedia/commons/a/ac/Preorder-traversal.gif">

### 중위 순회 InOrder Traversal

left → root → right

이진 탐색 트리에서 중위 순회는 그들의 값을 기준으로 오름차순으로 순회 하는것을 의미한다 

재귀와 함께 

```java
public void traverseInOrder(Node node) {
    if (node != null) {
        traverseInOrder(node.left);
        visit(node.value);
        traverseInOrder(node.right);
    }
}
```

재귀 없이 

- root를 stack에 push
- stack이 empty가 아닌 동안
    - 현재 노드의 가장 왼쪽 자식 노드 까지 닿을 때 까지 왼쪽 자식을 스택으로 push 한다
    - 현재 노드를 방문하다
    - 오른쪽 자식을 stack으로 push 한다

```java
public void traverseInOrderWithoutRecursion() {
    Stack<Node> stack = new Stack<Node>();
    Node current = root;
    stack.push(root);
    while(! stack.isEmpty()) {
        while(current.left != null) {
            current = current.left;                
            stack.push(current);                
        }
        current = stack.pop();
        visit(current.value);
        if(current.right != null) {
            current = current.right;                
            stack.push(current);
        }
    }
}
```

후위 순회 PostOrder Traversal 

재회와 함께 

```java
public void traversePostOrder(Node node) {
    if (node != null) {
        traversePostOrder(node.left);
        traversePostOrder(node.right);
        visit(node.value);
    }
}
```

재회 없이 

- root 노드를 stack에 push
- stack이 empty가 아닌 동안
    - 왼쪽과 오른쪽을 이미 방문 했는지 확인한다
    - 아니라면 오른쪽과 왼쪽을 stack으로 push

```java
public void traversePostOrderWithoutRecursion() {
    Stack<Node> stack = new Stack<Node>();
    Node prev = root;
    Node current = root;
    stack.push(root);

    while (!stack.isEmpty()) {
        current = stack.peek();
        boolean hasChild = (current.left != null || current.right != null);
        boolean isPrevLastChild = (prev == current.right || 
          (prev == current.left && current.right == null));

        if (!hasChild || isPrevLastChild) {
            current = stack.pop();
            visit(current.value);
            prev = current;
        } else {
            if (current.right != null) {
                stack.push(current.right);
            }
            if (current.left != null) {
                stack.push(current.left);
            }
        }
    }   
}
```

## DFS vs BFS

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQYkI8%2FbtqB8oDsMGe%2FEEYm0cKGYhxTR0kJhGiJPK%2Fimg.gif">

| DFS                          | BFS                     |
|------------------------------|-------------------------|
| 현재 정점에서 갈 수 있는 점들까지 들어가면서 탐색 | 현재 정점에서 연결된 가까운 점들부터 탐색 |
|  스택 또는 재귀 함수로 구현             | 큐를 이용해서 구현              |


## 시간 복잡도

N은 노드, E는 간선일 때

> 인접 리스트 : O(N+E)인접 행렬 : O(N²)

출처 : <https://www.baeldung.com/java-breadth-first-search>
<https://upload.wikimedia.org/wikipedia/commons/a/ac/Preorder-traversal.gif>