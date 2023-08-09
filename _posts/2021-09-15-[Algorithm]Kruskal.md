---
layout: post
title: "[Algorithm] Krusukal 크루스칼 알고리즘"
date: 2021-09-15
excerpt: "최소 비용 신장 트리 크루스칼 알고리즘"
tags: ["Algorithm", "Kruskal", "Java"]
comments: true
---
**크루스칼 알고리즘**은 가장 적은 비용으로 모든 노드를 연결하기 위해 사용하는 알고리즘 이다. 

다시 말해 최소 비용 신장 트리 (Spanning Tree) 를 만들기 위한 대표적인 알고리즘이라고 할 수 있다. 

여기서 말하는 **신장트리 (Spannig Tree)**는 원래 그래프의 정점 전부와 간선의 부분 집합으로 구성된 부분 그래프이다. 이때 스패닝 트리에 포함된 간선들은 정점점들을 트리형태로 전부 연결해야 한다. 

트리 형태여야 한다는 말은 선택된 간선들이 사이클이 이루지 않는다는 의미이며, 정점들이 꼭 부모-자식 관계로 연결될 필요는 없다는데 유의한다. 



가중치 그래프의 스패팅 트리 중 가중치의 합이 가장 작은 트리를 찾는 문제를 최소 스패닝 트리 문제라고 한다. 즉, 최소 스패닝 트리 문제는 그래프의 연결성을 그대로 유지하는 가장 '저렴한' 그래프를 찾는 문제라고 할 수 있다. 

크루스칼 알고리즘은 그래프의 모든 간선을 가중치의 오름차순으로 정렬한 뒤, 스패닝 트리에 하나씩 추가해 간다. 물론 가중치가 작다고 해서 무조건 간선을 트리에 더하는 것은 아니다. 자칫 하다가는 선택한 간선들이 사이클을 이룰 수 있기 때문이다. 따라서 결과적으로 사이클이 생기는 간선은 고려에서 제외 해야 한다. 

흔히 여러개의 도시가 있을 때 도시를 도로를 이용해 연결하고자 할 때 비용을 최소한으로 하고자 할 때 실제로 적용되는 알고리즘이다. 

크루스칼 알고리즘의 핵심은 간선 숫자는 노드의 갯수에서 하나를 뺀 숫자이다. E = V - 1 

크루스칼 과정을 본다면

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img1.PNG" />

일단 모든 노드를 최대한 적은 비용으로 '연결만' 시키면 되기 때문에 **모든 간선 정보를 오름차순**으로 정렬한 뒤에 비용이 적은 간선부터 차근차근 그래프에 포함시키면 된다. 

모든 간선들의 정보를 모드 저장 한 다음에 거리(비용)을 기준으로 오름차순으로 정렬을 수행했다. 

다음의 알고리즘에 맞게 그래프를 연결하면 가장 적은 비용으로 모든 노드를 연결한 그래프인 '최소 비용 신장 트리(Spannig Tree)가 만들어 진다. 

1. 정렬된 순서에 맞게 그래프에 포함 시킨다.
2. 포함시키기 전에는 사이클 테이블을 확인 한다.
3. 사이클을 형성하는 경우 간선을 포함시키지 않는다. 

최소 비용 신장 트리에서는 사이클이 발생하면 안된다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img2.PNG" />

첫번째 간선을 선택하여 연결한다.  1 ↔ 7  비용은 12

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img3.PNG" />

두번 째 간선을 선택하여 연결한다. 4 ↔ 7 비용은 13. 누적 합은 25 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img4.PNG" />

세번 째 간선을 선택하여 연결한다. 1 ↔ 5 비용은 17. 누적 합은 42 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img5.PNG" />

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img6.PNG" />

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img7.PNG" />

반복 하다가 1 ↔ 4 일 때는 사이클이 만들어 지기 때문에 안된다. parent node가 동일하기 때문에 무시하고 넘어 간다. 

이렇게 계속 반복 하다보면 값이 나온다. 

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img8.gif" />

Java Code : 

```java
import sun.awt.windows.WPrinterJob;

import java.util.*;

public class Kruskal {
    //부모 노드를 가져옴
    static int getParent(int[] parent, int x){
        if(parent[x] == x){
            return x;
        }
        return parent[x] = getParent(parent, parent[x]);
    }

    //부모 노드를 병합
    static void unionParent(int[] parent, int a, int b){
        a = getParent(parent, a);
        b = getParent(parent, b);
        if(a > b){
            parent[a] = b;
        }else {
            parent[b] = a;
        }
    }

    //같은 부모를 가지는 지 확인
    static boolean findParent(int[] parent, int a, int b){
        a = getParent(parent, a);
        b = getParent(parent, b);
        if( a == b){
            return true;
        }else {
            return false;
        }
    }

    public static class Edge implements Comparable<Edge>{
        int[] node = new int[2];
        int distance;
        public Edge(int a, int b, int distance){
            this.node[0] = a;
            this.node[1] = b;
            this.distance = distance;
        }
        @Override
        public int compareTo(Edge e1){
            return Integer.compare(this.distance, e1.distance);
        }
    }

    public static void main(String[] args){
        int n = 7;
        int m = 11;

        PriorityQueue<Edge> pq = new PriorityQueue<>();
        pq.add(new Edge(1, 7, 12));
        pq.add(new Edge(1, 4, 28));
        pq.add(new Edge(1, 2, 67));
        pq.add(new Edge(1, 5, 17));
        pq.add(new Edge(2, 4, 24));
        pq.add(new Edge(2, 5, 62));
        pq.add(new Edge(3, 5, 20));
        pq.add(new Edge(3, 6, 37));
        pq.add(new Edge(4, 7, 13));
        pq.add(new Edge(5, 6, 45));
        pq.add(new Edge(6, 7, 73));

        //간선의 비용으로 오름차순 정렬

        //각 정점이 포함된 그래프가 어디인지 저장
        int[] set = new int[n+1];
        for(int i =1; i<n+1; i++){
            set[i] = i;
        }

        int sum = 0;
        for(int i =1; i<m+1; i++){
            Edge edge = pq.poll();
            if(!findParent(set, edge.node[0], edge.node[1])){
                sum += edge.distance;
                unionParent(set, edge.node[0] , edge.node[1]);
            }
        }
        System.out.println(sum);
    }

}
```

##다익스트라 vs 크루스칼 
<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0915/img9.png" />

출처 : [18. 크루스칼 알고리즘](https://m.blog.naver.com/ndb796/221230994142)
[다익스트라 vs 크루스칼](https://buddev.tistory.com/21)