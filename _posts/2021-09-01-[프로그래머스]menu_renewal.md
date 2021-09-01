---
layout: post
title: "[Algorithm] 백준_14502 연구소"
date: 2021-08-30
excerpt: "백준 문제 14502"
tags: [Java, Algorithm, 백준]
comments: true
---

[프로그래머스_메뉴리뉴얼](https://programmers.co.kr/learn/courses/30/lessons/72411)

문제 푼 날짜 : 2021-08-30
문제 푼 시간 : 저번에 풀었던 (or 답을 봤던) 문제인데 여전히 못풀음.. 힌트 보고 3시간걸림
프로그래머스 난이도 : LEVEL 2

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0901/img1.png" />

문제 풀이 : 

1. 주어진 코스 개수에 맞게 손님이 주문한 단품 요리를 조합한다. 
2. dfs 함수를 만들어서 재귀호출을 한다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0901/img2.png" />

1. cnt가 코스 갯수와 일치하면 map에 집어 넣는다. 
2. map에 넣기 전에 str을 정렬 시켜준다. ("XY", "YX" 이런 상황 때문에) 
3. 담품 요리를 조합한 갯수가 가장 많은 걸로 most 변수를 갱신한다. 
4. for문을 다 돌리고 most 값과 일치하거나 most가 1 보다 클때 Priority Queue에 map의 key(메뉴)를 담아준다. 
5. Priority Queue에 담아주는 이유는 알파벳 오름차순 순서대로 출력해야 하기 때문이다. 

자바 코드 : 

```java

public class 메뉴리뉴얼 {
    public static void main(String[] args){
        메뉴리뉴얼 m = new 메뉴리뉴얼();
        String[] orders = {"XYZ", "XWY", "WXA"};
        int[] course = {2, 3, 4};
        String[] result = m.solution(orders, course);
        for(String r : result){
            System.out.println(r);
        }
    }

    static int most;
    public String[] solution(String[] orders, int[] course) {

        PriorityQueue<String> pq = new PriorityQueue<>();
        for(int c : course) {
            most = 0;
            Map<String, Integer> map = new HashMap<>();
            for (int i = 0; i < orders.length; i++) {
                getMenu(0, "", c, 0, orders[i], map);
            }

            Iterator<String> it = map.keySet().iterator();

            while (it.hasNext()) {
                String key = it.next();
                if (map.get(key) >= most && most > 1) {
                    pq.offer(key);
                }
            }
        }
        String[] result = new String[pq.size()];
        int i =0;
        while(!pq.isEmpty()){
            result[i++] = pq.poll();
        }

        return result;
    }

    public static void getMenu(int cnt, String str, int target, int idx, String word, Map<String, Integer> map){
        if(cnt == target) {
            char[] c = str.toCharArray();
            Arrays.sort(c);
            str = String.valueOf(c);
            map.put(str, (map.get(str) == null ? 1 : map.get(str) + 1));
            if(most < map.get(str)) {
                most = map.get(str);
            }
            return;
        }
        char[] wordArray = word.toCharArray();
        for(int i = idx; i < word.length(); i++){
            getMenu(cnt+1, str+wordArray[i], target, i+1, word, map);
        }
        return;
    }

}
```