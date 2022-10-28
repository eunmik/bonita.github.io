---
layout: post
title: "[Algorithm] 백준_1011 Fly me to the Alpha Centauri"
date: 2021-08-23
excerpt: "백준 문제 1011"
tags: [Java, Algorithm, 백준]
comments: true
---
[1011번: Fly me to the Alpha Centauri](https://www.acmicpc.net/problem/1011)

문제 푼 날짜 : 2021-08-23
문제 푼 시간 : 힌트 얻어서 풀었지만 그래도 3시간 이나 걸림.. 
백준 난이도 : GOLD V 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0823/img1.png" />

---

알고리즘으로 구현하는 문제라기 보단 공식을 발견해야 하는 문제 였다. 

제곱근을 이용해서 규칙을 적용하는데 제곱근을 까먹어서 멍청이가 된 기분이었다.... 

표를 그려보면 뭔가 규칙같은게 보인다. 

이동거리가 저렇게 나오는 이유는 바로 아래 그림으로 설명했다. 

횟수가 나오는 숫자를 보면 (max * 2) -1, max * 2, max*2 +1 라는 수식이 적용이 된다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0823/img2.png" />

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0823/img3.png" />

코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class FlymetotheAlphaCentauri {
    static int caseNum;
    static int[][] cases;
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        caseNum = Integer.parseInt(st.nextToken());
        cases = new int[caseNum][2];
        for(int i = 0; i< cases.length; i++){
            st = new StringTokenizer(br.readLine(), " ");
            for(int j=0; j<2; j++){
                cases[i][j] = Integer.parseInt(st.nextToken());
            }
        }
        for(int[] testCase : cases){
            System.out.println(solution(testCase));
        }

    }
    public static int solution(int[] testCase){
        int dist = testCase[1] - testCase[0];

        int max = (int) Math.sqrt(dist);

        if(max == Math.sqrt(dist)) { //제곱근이 정확히 떨어질 때
            return max*2-1;
        } else if(dist <= (max*max)+max){
            return max*2;
        } else {
            return max*2+1;
        }

    }
}
```