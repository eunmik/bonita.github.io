---
layout: post
title: "[Algorithm] Prim 프림 알고리즘"
date: 2021-09-15
excerpt: "프림 Prim 알고리즘"
tags: ["Algorithm", "Prim", "Java"]
comments: true
---

크루스칼 알고리즘과 같은 원리로 동작하는 또 다른 스패닝 트리 알고리즘으로 프림의 최소 스패닝 트리 알고리즘이 있다. 

크루스칼 알고리즘에서는 여기저기서 산발적으로 만들어진 트리의 조각들을 합쳐 스패닝 트리를 만드는 반면, 프림 알고리즘에서는 하나의 시작점으로 구성된 트리에 간선을 하나씩 추가하며 스패닝 트리가 될 때까지 키워 간다. 따라서 항상 선택된 간선들은 중간 과정에서도 항상 연결된 트리를 이루게 된다. 

프림 알고리즘 도 마찬가지로 선택할 수 있는 간선들 중 가중치가 가장 작은 간선을 선택하는 과정을 반복한다. 스패닝 트리를 찾아낼 때 까지 후보 간선들 중 가중치가 가장 작은 간선을 추가하는 과정을 반복한다. 

## 프림 알고리즘의 구현

각 정점이 트리에 포함되었는지 여부를 나타내는 boolean 값 배열 visited[]를 두고, 각 차례 마다 모든 간선을 순회하면서 당므으로 트리에 추가할 간선을 찾으면 된다. 이런 구현은 시간이 너무 오래 걸린다는 문제가 있다. 정점을 하나 추가할 때 마다 $O(|E|)$의 시간이 걸리기 때문에, 전체 시간 복잡도는 $O(|V||E|)$가 된다. 크루스칼 알고리즘의 시간 복잡도가 $O(|E|lg|E|)$임을 떠올려 보면 이대로는 실용성이 거의 없음을 알 수 있다. 

이 알고리즘을 최적화 하는 비결은 한 정점에 닿는 간선이 두 개 이상일 경우 이들을 하나하나 검사하는 과정이 시간 낭비임을 깨닫는 것이다. 트리에 속하지 않은 각 정점에 대해, 트리와 이 정점을 연결하는 가장 짧은 간선에 대한 정보를 저장하고, 각 정점을 순회하면서 다음에 추가할 정점을 찾으면 $O(|E|)$ 대신에 $O(|V|)$  시간에 다음에 추가할 간선을 찾을 수 있다. 

프림 알고리즘은 이를 위해 트리와 이 정점을 연결하는 간선의 최소 가중치를 저장하는 배열 minWeight[]를 유지한다. 이 배열은 트리에 정점을 새로 추가할 때마다 이 정점에 인접한 간선들을 순회하면서 갱신하게 된다. 이렇게 하면 추가할 새 정점을 찾는 작업은 O(|V|)만에 할 수 있고, 모든 간선은 두번씩만 검사되므로 *(간선 (u,v)는 u를 트리에 추가했을 때 한 번, v를 트리에 추가했을 때 한번 검사)* 전체 시간 복잡도는 $O(|V|^2+|E|)$가 된다. 거의 모든 정점들 사이에 간선이 있는 밀집 그래프의 경우 프림알고리즘은 크루스칼보다 빠르게 동작한다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0917/img1.PNG" />

맨처음에 시작할 정점에 대한 destination과weight값을 Priority Queue에 넣어준다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0917/img2.PNG" />

PriorityQueue에서 꺼내서 current.destination과 인접한 정점들을 PriorityQueue에 넣어준다. 

visited[current.destination]을 True로 해준다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0917/img3.PNG" />

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0917/img4.PNG" />

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0917/img5.PNG" />

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0917/img6.PNG" />

같은 방식으로 다른 정점들을 방문한다. 

그러면 최종 결과적으로 최소 신장 트리의 값은 10 이 된다. 

Java Code :

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.PriorityQueue;

public class Prim_Heap {
    public static void main(String[] args){
        int N = 5; //정점의 갯수
        List<Edge>[] graph = new ArrayList[N]; //정점 간 가중치 저장
        boolean[] visited = new boolean[N]; //정점 방문 여부
        int[] minWeight = new int[N]; //정점에 도착할 수 있는 가장 작은 가중치

        for(int i =0; i<N; i++){
            graph[i] = new ArrayList<>();
        }
        //값 초기화, 모두 정수 최대값으로 초기화, 최대값을 경우 길이 없다는 것을 의미
        Arrays.fill(minWeight, Integer.MAX_VALUE);

        //가중치 추가
        graph[0].add(new Edge(2, 3));
        graph[2].add(new Edge(2, 3));
        graph[0].add(new Edge(3, 5));
        graph[3].add(new Edge(0, 5));
        graph[1].add(new Edge(2, 4));
        graph[2].add(new Edge(1, 4));
        graph[1].add(new Edge(3, 4));
        graph[3].add(new Edge(1, 4));
        graph[2].add(new Edge(4, 2));
        graph[4].add(new Edge(2, 2));
        graph[3].add(new Edge(4, 1));
        graph[4].add(new Edge(3, 1));

        //힙으로 구현된 우선순위 큐
        PriorityQueue<Edge> pq = new PriorityQueue<Edge>();
        pq.add(new Edge(0,0)); //0번 정점을 장분하기 위해 필요한 가중치는 0

        while(!pq.isEmpty()){
            Edge edge = pq.poll();
            int destination = edge.destination;
            if(!visited[destination]){ //방문하지 않은 정점일 경우
                visited[destination] = true;
                minWeight[destination] = edge.weight; //최소 가중치 저장
                for(int i =0; i<graph[destination].size(); i++){
                    Edge next = graph[destination].get(i);
                    if(minWeight[next.destination] == Integer.MAX_VALUE){
                        pq.add(next);
                    }
                }
            }
        }
        int sumWeight = 0; //가중치의 합 초기화
        for(int i : minWeight){
            sumWeight += i;
        }
        System.out.println("MST를 만드는 최소 가중치 값: "+sumWeight);

    }

    static class Edge implements Comparable<Edge>{
        int destination;
        int weight;
        public Edge(int destination, int weight){
            this.destination = destination;
            this.weight = weight;
        }

        @Override
        public int compareTo(Edge e1){ //가중치가 작은 순서대로 정렬
            return Integer.compare(this.weight, e1.weight);
        }
    }
}
```

출처 : 알고리즘 문제 해결 전략 책