---
layout: post
title: "[Algorithm] DFS, BFS란? "
date: 2021-04-19
excerpt: "DFS, BFS란? "
tags: [Algorithm, DFS, BFS]
comments: true
---
# [Algorithm] DFS/BFS

## BST(Binary Search Tree) 이진트리탐색

이진탐색과 연결리스트를 결합한 자료구조의 일종이다. 이진 탐색의 효율적인 탐색능력을 유지하면서도, 빈번한 삭제와 입력이 가능하게 끔 고안되었다. 

이진 탐색 트리는 아래의 속성을 가지고 있다. 

- 각 노드의 왼쪽 서브트리에는 해당 노드의 값보다 작은 값을 지닌 노드들로 이루어져 있다.
- 각 노드의 오른쪽 서브트리에는 해당 노드의 값보다 큰 값을 지닌 노드들로 이루어져 있다.
- 중복된 노드가 없어야 한다.
- 왼쪽 서브트리, 오른쪽 서브트리 또한 이진탐색트리이다.

이진탐색트리를 순회 할 때는 3가지 방식이있다.  

(근데 이거 완전탐색할 때 했는데 기억안남.. 쏘 스튜핏) 

- 전위 순회 (Pre-Order Traversal) : root → left → right
- 중위 순회 (Inorder Traversal) : left → root → right
- 후위 순회 (Post-Order Traversal) : left → right → root

```java
public class TreeNode {
	int data;
	TreeNode left;
	TreeNode right;

	public TreeNode(int data) {
		this.data = data;
		this.left = this.right = null;
	}
}
```

### 전위 순회 (Pre-Order Traversal)

```java
public void preorderTraversal(TreeNode root) {
	if (root != null) {
		System.out.print(root.data + " ");
		preorderTraversal(root.left);
		preorderTraversal(root.right);
	}
}
```

<img src="https://miro.medium.com/max/625/1*UGoV21qO6N8JED-ozsbXWw.gif">

PreOrder Traversal

### 중위 순회(In-Order Traversal)

```java
public void inorderTraversal(TreeNode root){
				if (root != null) {
								inorderTraversal(root.left);
								System.out.print(root.data+ " ");
								inorderTraversal(root.right);
			}
}
```

<img src="https://miro.medium.com/max/625/1*bxQlukgMC9cGv_MFUllX2Q.gif">

Inorder Traversal

### 후위 순회(Postorder Traversal)

```java
public void postorderTraversal(TreeNode root) {
	if (root != null) {
		postorderTraversal(root.left);
		postorderTraversal(root.right);
		System.out.print(root.data + " ");
	}
}
```

<img src="https://miro.medium.com/max/625/1*UGrzA4qtLCaaCiNAKZyj_w.gif">

PostOrder Traversal 

그래프 탐색의 목적은 모든 정점을 한 번씩 방문 하는 것이다. 크게 두가지로 나누는데 

- DFS (Depth First Search, 깊이 우선 탐색)
    - 구현 방법 : Stack
- BFS (Beneath First Search, 너비 우선 탐색)
    - 구현 방법 : Queue

<img src="https://t1.daumcdn.net/cfile/tistory/2254723E588084F830">

### DFS vs BFS

## DFS(Depth First Search, 깊이 우선 탐색)

DFS는 트리와 그래프를 순회하거나 탐색하는 알고리즘 중에 하나이다.  DFS의 기본 동작은 Stack을 이용해서 갈 수 있는 만큼 최대한 깊이 가고, 더 이상 갈곳이 없다면 이전 정점으로 돌아간다는 것이다. 그래프와 트리의 다른 점은 그래프는 싸이클이 생겨 두 번 방문하는 일이 생길 수 있다는 것이다. 

노드를 두 번방문하는 것을 피하기 위해서 array를 사용해서 방문한 노드를 저장한다. 

### DFS 장점과 단점

# 장점 

- 현 경로상의 노드를 기억하기 때문에 적은 메모리를 사용한다.
- 찾으려는 노드가 깊은 단계에 있는 경우 BFS 보다 빠르게 찾을 수 있다.

# 단점

- 해가 없는 경로를 탐색 할 경우 단계가 끝날 때까지 탐색한다. 효율성을 높이기 위해서 미리 지정한 임의 깊이까지만 탐색하고 해를 발견하지 못하면 빠져나와 다른 경로를 탐색하는 방법을 사용한다.
- DFS를 통해서 얻어진 해가 최단 경로라는 보장은 없다. DFS는 해에 도착하면 탐색을 종료하기 때문이다.

example.  

```java
Input: n = 4, e = 6 
~~0 -> 1~~, ~~0 -> 2~~, ~~1 -> 2~~, ~~2 -> 0~~, ~~2 -> 3~~, 3 -> 3 
Output: DFS from vertex 1 : 1 2 0 3 
Explanation: 
DFS Diagram:
```

```java
DFS(u)
for each neighbor v of u
  if v is unvisited, tree edge, DFS(v)
  else if v is explored, bidirectional/back edge
  else if v is visited, forward/cross edge
u : 1 -> DFS(2) 
u : 2 -> DFS(0)
u : 0 -> v : 1 is exlpored, v: 2 is explored, 사이클이 생김
backtrack을 한다 
u : 2 -> DFS(3)
```

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20200507074112/ezgif.com-gif-maker61.gif">

```java
Input: n = 4, e = 6 
2 -> 0, 0 -> 2, 1 -> 2, 0 -> 1, 3 -> 3, 1 -> 3 
Output: DFS from vertex 2 : 2 0 1 3 
Explanation: 
DFS Diagram:
```

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20200507075041/ezgif.com-gif-maker7.gif">

### DFS FOR A GRAPH 예제

Practice : [https://practice.geeksforgeeks.org/problems/depth-first-traversal-for-a-graph/1](https://practice.geeksforgeeks.org/problems/depth-first-traversal-for-a-graph/1)

```java
// Java program to print DFS
//mtraversal from a given given
// graph
import java.util.*;

// This class represents a
// directed graph using adjacency
// list representation
class Graph {
    private int V; // No. of vertices

    // Array of lists for
    // Adjacency List Representation
    private LinkedList<Integer> adj[];

    // Constructor
    @SuppressWarnings("unchecked") Graph(int v)
    {
        V = v;
        adj = new LinkedList[v];
        for (int i = 0; i < v; ++i)
            adj[i] = new LinkedList();
    }

    // Function to add an edge into the graph
    void addEdge(int v, int w)
    {
        adj[v].add(w); // Add w to v's list.
    }

    // A function used by DFS
    void DFSUtil(int v, boolean visited[])
    {
        // Mark the current node as visited and print it
        visited[v] = true;
        System.out.print(v + " ");

        // Recur for all the vertices adjacent to this
        // vertex
        Iterator<Integer> i = adj[v].listIterator();
        while (i.hasNext())
        {
            int n = i.next();
            if (!visited[n])
                DFSUtil(n, visited);
        }
    }

    // The function to do DFS traversal.
    // It uses recursive
    // DFSUtil()
    void DFS(int v)
    {
        // Mark all the vertices as
        // not visited(set as
        // false by default in java)
        boolean visited[] = new boolean[V];

        // Call the recursive helper
        // function to print DFS
        // traversal
        DFSUtil(v, visited);
    }

    // Driver Code
    public static void main(String args[])
    {
        Graph g = new Graph(4);

        g.addEdge(0, 1);
        g.addEdge(0, 2);
        g.addEdge(1, 2);
        g.addEdge(2, 0);
        g.addEdge(2, 3);
        g.addEdge(3, 3);

        System.out.println(
                "Following is Depth First Traversal "
                        + "(starting from vertex 2)");

        g.DFS(1);
    }
}
// This code is contributed by Aakash Hasija
```

### DFS의 시간 복잡도

DFS는 총 V번 부르게 된다. 그러나, 인접 행렬로 구현한 경우와 달리 인접 리스트로 구현한 DFS는 DFS 하나당 각 정점에 연결되어 있는 간선의 개수만큼 탐색을 하게 되어 DFS 하나의 수행 시간이 일정하지 않다. 

전체 DFS가 다 끝난 이후를 생각해 보면, DFS가 다 끝났을 시점에는 모든 정점을 한번씩 방문하고, 모든 간선을 한번 씩 검사하게 되므로 O(V+E)의 시간이 걸린다고 말할 수 있다. 

따라서, 인접리스트로 구현할 경우 DFS의 시간 복잡도는 O(V+E)이다. 

## BFS(Breadth First Search, 너비 우선 탐색)

BFS의 기본 동작 방식은 queue를 이용하여 지금 위치에서 갈 수 있는 것들을 모두 큐에 넣는 방식이다. 이때, queue에 넣을 시점에 해당 노드를 방문했다고 체크해야한다. (DFS는 일단 들어가서 체크한다) 

BFS도 DFS와 마찬가지로 그래프에서는 싸이클이 생겨 두 번 방문하는 일이 생길 수 있다. 그래서 노드를 두 번방문하는 것을 피하기 위해서 array를 사용해서 방문한 노드를 저장한다. 

### BFS FOR A GRAPH 예제

```java
// Java program to print DFS
//traversal from a given given
// graph
import java.util.*;

// This class represents a
// directed graph using adjacency
// list representation
class ConnectedGraph {
    private int V; // No. of vertices

    // Array of lists for
    // Adjacency List Representation
    private LinkedList<Integer> adj[];

    // Constructor
    @SuppressWarnings("unchecked")
    ConnectedGraph(int v)
    {
        V = v;
        adj = new LinkedList[v];
        for (int i = 0; i < v; ++i)
            adj[i] = new LinkedList();
    }

    // Function to add an edge into the graph
    void addEdge(int v, int w)
    {
        adj[v].add(w); // Add w to v's list.
    }

    // prints BFS traversal from a given source s
    void BFS(int s)
    {
        // Mark all the vertices as not visited(By default
        // set as false)
        boolean visited[] = new boolean[V];

        // Create a queue for BFS
        LinkedList<Integer> queue = new LinkedList<Integer>();

        // Mark the current node as visited and enqueue it
        visited[s]=true;
        queue.add(s);

        while (queue.size() != 0)
        {
            // Dequeue a vertex from queue and print it
            s = queue.poll();
            System.out.print(s+" ");

            // Get all adjacent vertices of the dequeued vertex s
            // If a adjacent has not been visited, then mark it
            // visited and enqueue it
            Iterator<Integer> i = adj[s].listIterator();
            while (i.hasNext())
            {
                int n = i.next();
                if (!visited[n])
                {
                    visited[n] = true;
                    queue.add(n);
                }
            }
        }
    }

    // Driver Code
    public static void main(String args[])
    {
        ConnectedGraph g = new ConnectedGraph(4);

        g.addEdge(0, 1);
        g.addEdge(0, 2);
        g.addEdge(1, 2);
        g.addEdge(2, 0);
        g.addEdge(2, 3);
        g.addEdge(3, 3);

        System.out.println(
                "Following is Breadth First Traversal "
                        + "(starting from vertex 2)");

        g.BFS(2);
    }
}
// This code is contributed by Aakash Hasija
```

<img src="https://ksshlee.github.io/assets/img/al_lib/2019_12_31/how_bfs_works.gif">

### BFS 시간 복잡도

연결 리스트를 바탕으로 구현된 DFS의 시간 복잡도를 구할 때와 비슷한 논리로 시간 복잡도를 구할 수 있다. 

BFS가 끝난 뒤를 생각해 보면, 해당 라인은 모든 간선에 대해서 한 번씩 비교할 것이기 때문에 총 E번 반복할 것이고, 각 정점을 하 번씩 모두 방문하여 V의 시간이 걸릴 것이다. 

따라서, 인접 리스트로 구현한 BFS의 시간 복잡도는 O(V+E)이다. 

### 어디에 쓰일까?

- 웹 크롤링 - 동적으로 생성되는 무한한 인터넷 페이지를 구글이 크롤링 하기 위해서 BFS를 한다.
- 네트워크 브로드 캐스트
- 가비지 컬렉션 -

### BFS의 장점과 단점

# 장점

- 답이 되는 경로가 여러 개인 경우에도 최단 경로임을 보장한다.
- 최단 경로가 존재하면 깊이가 무한정 깊어진다고 해도 답을 찾을 수 있다.

# 단점

- 경로가 매우 길 경우에는 탐색 가지가 급격히 증가함에 따라 보다 많은 기억 공간을 필요로 하게 된다.
- 해가 존재하지 않는다면 유한 그래프의 경우에는 모든 그래프를 탐색한 후에 실패로 끝난다.
- 무한 그래프의 경우에는 결코 해를 찾지도 못하고, 끝내지도 못한다.

## 잘 정리 되어있는 블로그

[https://covenant.tistory.com/132](https://covenant.tistory.com/132)