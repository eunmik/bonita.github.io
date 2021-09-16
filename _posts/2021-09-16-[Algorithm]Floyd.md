---
layout: post
title: "[Algorithm] Floyd 플로이드 와샬 알고리즘"
date: 2021-09-15
excerpt: "최소 비용 신장 트리 크루스칼 알고리즘"
tags: ["Algorithm", "Kruskal", "Java"]
comments: true
---
다익스트라 알고리즘은 하나의 정점에서 출발했을 때 다른 모든 정점의 최단 경로를 구하는 알고리즘이다. 하지만 만약에 '모든 정점'에서 '모든 정점'으로의 최단 경로를 구하고 싶다면 플로이드 와샬 알고리즘을 사용해야 한다.

다익스트라 알고리즘은 가장 적은 비용을 하나씩 선택해야 했다면 플로이드 와샬 알고리즘은 기본적으로 '거쳐가는 정점'을 기준으로 알고리즘을 수행한다는 특징이 있다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img1.PNG" />

위와 같은 그래프가 존재한다고 했을때 각각의 정점이 다른 정점으로 가는 비용을 이차원 배열로 초기화 했다.

이 테이블에서 의미하는 바는 '현재까지 계산된 최소비용' 이다. 

노드1을 거쳐가는 경우로 과정을 설명해 보자면

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img2.PNG" />

1을 거쳐가는 경우에는 총 6가지 공간만 갱신해주면 된다. 

[2,3], [2,4], [3,2], [3,4],[4,2],[4,3] 

자기자신인 경우와 1을 포함하고 있는 경우에는 갱신이 되지 않기 때문이다. 

갱신할 때는 다음을 비교하는 방식이 반복 되는 것이다.

**X에서 Y로 가는 최소 비용 VS X에서 노드N으로 가는 비용 + 노드N에서 Y로 가는 비용**

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img3.PNG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img4.PNG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img5.PNG" />

[2,3]의 최소비용을 구할 때 [2,1] + [1,3]의 값과 [2,3]을 비교했을 때 더 작은 값으로 교체가 된다. 

7 + INF < 9  ⇒ FALSE, 그니깐 값은 [2,3]의 값이 그대로 유지된다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img6.PNG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img7.PNG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img8.PNG" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0916/img9.PNG" />

[2,4]의 최소비용을 구할 때는 

[2,1]의 비용과 [1,4]의 비용을 더한 값과 [2,4]의 값을 비교 해 보면

(7 + 8 > INF ⇒ TRUE) 

[2,1]의 비용과 [1,4]의 비용을 더한 값이 더 작기 때문에 갱신해 준다. 

이렇게 반복 하다보면 아래처럼 결과 값이 나온다. 

![슬라이드9.PNG](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/766c7665-d55d-44e0-a3b1-c677782fa280/슬라이드9.png)

Java Code :

```java
public class floydWarshall {
    public static void main(String[] args){
        number = 4;
        INF = Integer.MAX_VALUE;
        int[][] a = {{0, 5, INF, 8}, {7, 0, 9, INF}, {2, INF, 0, 4}, {INF, INF, 3, 0}};
        floydWarshall(a);
    }
    static int number;
    static int INF;
    static void floydWarshall(int[][] a){
        //결과 그래프를 초기화 한다.
        int[][] d = new int[number][number];
        for(int i =0; i<number; i++){
            for(int j =0; j<number; j++){
                d[i][j] = a[i][j];
            }
        }

        // K = 거쳐가는 노드
        for(int k = 0; k <number; k++){
            for(int i = 0; i<number; i++){
                for(int j = 0; j<number; j++){
                    //X에서 Y로 가는 최소 비용 VS X에서 노드N으로 가는 비용 + 노드N에서 Y로 가는 비용
                    if(d[i][k] != INF && d[k][j] != INF && d[i][k] + d[k][j] < d[i][j]){
                        d[i][j] = d[i][k] + d[k][j];
                    }
                }
            }
        }

        //결과를 출력
        for(int i =0; i<number; i++){
            for(int j =0; j<number; j++){
                System.out.print(d[i][j] +" ");
            }
            System.out.println();
        }

    }
}
```

출처 : [24. 플로이드 와샬 알고리즘](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=ndb796&logNo=221234427842)
