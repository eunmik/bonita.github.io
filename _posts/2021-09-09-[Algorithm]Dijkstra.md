---
layout: post
title: "[Algorithm] Dijkstra 다익스트라 알고리즘"
date: 2021-09-09
excerpt: "최단경로 다익스트라 알고리즘"
tags: [Java, Algorithm]
comments: true
---

**다익스트라(Dijkstra)** 알고리즘은 단일 시작점 최단 경로 알고리즘으로 시작 정점 s에서부터 다른 정점들까지의 최단거리를 계산한다. 다익스트라는 방향 가중 그래프에 최단 경로 문제를 해결한다. 

이때 음의 간선은 포함할 수 없다. 음수 사이클이 있는 경우 최단 경로를 정확하게 찾아줄 수 없기 때문이다. 

다익스트라는 DP를 활용한 대표적인 최단경로 탐색 알고리즘인데 그 이유는 **최단거리는 여러개의 최단 거리로 이루어져 있기 때문**이다. 기본적으로 다익스트라는 하나의 최단 거리를 구할 때 이전까지 구했던 최단 거리 정보를 그대로 사용한다는 특징이 있다. 

아래 백준 최단경로 문제를 예를들어 과정을 이해해 보도록 하겠다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0909/img1.PNG" />

출발 노드를 먼저 1로 설정한다.

출발 노드를 기준으로 각 노드의 최소 비용을 저장할 배열은 dist[] 배열이다. 

노드를 방문했는지 확인하기 위한 배열은 checked[] 이다. 

우선순위 큐에는 가중치를 기준으로 노드를 집어 넣는다. 

각 노드의 최소 비용은 무한대(Integer.MAX_VALUE)로 설정한다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0909/img2.PNG" />

첫번째 출발 노드를 방문했고 자기 자신이기 때문에 dist[1]에는 최소 비용을 0으로 설정한다. 

우선순위 큐에 넣어 놓은 첫번째 출발 노드의 destination과 weight를 확인한다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0909/img3.PNG" />

1번 노드의 인접한 노드 (2번째 노드) 에서 

현재 노드가 가지고 있는 최소 비용과(0)  

2번노드 까지 가는 비용(2)을 더 했을 때 

dist[2] 보다 작으면 최소 비용 노드를 갱신한다. 

```java
if(d[next.destination] >= d[destination] + next.weight){
	d[next.destination] = d[destination] + next.weight;
  pq.add(new Edge(next.destination, d[next.destination]));
}
```

갱신하고 비용이 짧은 순으로 찾을 수 있게 우선순위 큐에 add 한다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0909/img4.PNG" />

같은 방식으로 

1번 노드의 인접한 노드 (3번째 노드) 에서 

현재 노드가 가지고 있는 최소 비용과(0)  

3번노드 까지 가는 비용(3)을 더 했을 때 

dist[3] 보다 작으면 최소 비용 노드를 갱신한다. 

... 

이런식으로 반복 하다보면 모든 노드에 대한 최소비용을 구할 수 있다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0909/img5.gif" />

대표적인 최단경로 문제 

[백준_1753](https://www.acmicpc.net/problem/1753) 

전체 JAVA 코드

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.*;

public class 최단경로 {
    public static void main(String[] args) throws Exception{
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine(), " ");
        int V = Integer.parseInt(st.nextToken());
        int E = Integer.parseInt(st.nextToken());
        int start = Integer.parseInt(br.readLine());
        dist = new int[V + 1];
        checked = new boolean[V + 1];
        adj = new ArrayList[V+1]; //그래프의 인접 리스트
        for(int i =0; i<V+1; i++){
            adj[i] = new ArrayList<>();
        }

        for(int i =0; i<E; i++){

            st = new StringTokenizer(br.readLine(), " ");
            int u = Integer.parseInt(st.nextToken());
            int v = Integer.parseInt(st.nextToken());
            int w = Integer.parseInt(st.nextToken());

            adj[u].add(new Edge(v, w));

        }

        dijkstra(start);

        StringBuilder sb = new StringBuilder();
        for(int i = 1; i< dist.length; i++){
            if(dist[i] == Integer.MAX_VALUE){
                sb.append("INF\n");
            }else {
                sb.append(dist[i]+"\n");
            }

        }
        System.out.println(sb.toString());

    }
    static ArrayList<Edge>[] adj;
    static int[] dist;
    static boolean[] checked;
    static void dijkstra(int start){
        Arrays.fill(dist, Integer.MAX_VALUE);
        PriorityQueue<Edge> pq = new PriorityQueue<Edge>();
        pq.add(new Edge(start, 0));
        dist[start] = 0;

        while(!pq.isEmpty()){
            Edge current = pq.poll();
            int destination = current.destination;
            
            //방문한 노드이면 넘어간다. 
            if(checked[destination])
                continue;
            else
                checked[destination] = true;
            
            //인접한 정점들을 모두 검사한다. 
            for(Edge next : adj[destination]){
                //더 짧은 경로를 발견하면, dist[]를 갱신하고 우선순위 큐에 넣는다. 
                if(dist[next.destination] >= dist[destination] + next.weight){
                    dist[next.destination] = dist[destination] + next.weight;
                    pq.add(new Edge(next.destination, dist[next.destination]));
                }
            }
        }
    }

    static class Edge implements Comparable<Edge>{
        int destination;
        int weight;

        public Edge(int destination, int weight){
            this.destination = destination;
            this.weight = weight;
        }
        @Override
        public int compareTo(Edge e1){
            return Integer.compare(this.weight, e1.weight); //Integer.compare(e1.eight, this.weight)를 하니깐 틀렸음. 오름차순, 내림차순 차이
        }

    }
}
```

출처 : 

[동빈나_다익스트라 알고리즘(Dijkstra Algorithm)](https://m.blog.naver.com/ndb796/221234424646)
[다익스트라 알고리즘(Dijkstra Algorithm)](https://velog.io/@lre12/%EB%8B%A4%EC%9D%B5%EC%8A%A4%ED%8A%B8%EB%9D%BC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98Dijkstra-Algorithm)ㅁ