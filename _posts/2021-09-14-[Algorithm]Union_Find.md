---
layout: post
title: "[Algorithm] Union-Find 합집합 찾기 알고리즘"
date: 2021-09-14
excerpt: "합집합 찾기 알고리즘"
tags: ["Algorithm", "Union-Find", "Java"]
comments: true
---
**Union-Find**는 대표적인 그래프 알고리즘으로 합집합 찾기라는 의미를 가졌다.

여러 개의 노드가 존재할 때 두 개의 노드를 선택해서, 현재 이 두 노드가 서로 같은 그래프에 속하는지 판별하는 알고리즘 이다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0914/img1.png" />

위와 같이 모드 연결되지 않고 각자 자기 자신만을 집합의 원소로 가지고 있을 때는 

모든 값이 자기 자신을 가리키도록 만든다. 

표에서는 첫번째 행은 '노드번호'를 위미하고 두번째 행은 '부모 노드 번호'를 의미한다.

즉, 자신이 어떤한 부모에 포함되어 있는지를 의미한다.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0914/img2.png" />

1과 2가 연결이 되었을 때는 2번째 인덱스의 값에 '1'이 들어간다. 부모를 합칠 때는 일반적으로더 작은 값쪽으로 합친다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0914/img3.png" />

2와 3도 연결이 되어있다면 1과 3은 부모가 각각 1과 2로 다르기 때문에 '부모 노드'만 보고는 한번에 파악할 수 없다. 그래서 재귀 함수가 사용된다. 

3의 부모를 찾기 위해서 3이 가르키고 있는 2를 찾는다. 2의 부모가 1을 가리키고 있으므로 결과적으로 3의 부모는 1이 되는구나 라고 파악할 수 있다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0914/img4.gif" />

Java Code: 

```java

//합집합 찾기
public class UnionFind {
    static int getParent(int parent[], int x){
        if(parent[x] == x){
            return x;
        }
        return parent[x] = getParent(parent, parent[x]); //재귀 함수를 사용한다. 
    }

    //각 부모 노드를 합친다.
    static void unionParent(int parent[], int a, int b){
        a = getParent(parent, a);
        b = getParent(parent, b);
        if(a < b) { //더 작은 쪽으로 합친다. 
            parent[b] = a;
        } else {
            parent[a] = b;
        }
    }

    //같은 부모 노드를 가지는지 확인한다.
    static boolean findParent(int parent[], int a, int b){
        a = getParent(parent, a);
        b = getParent(parent, b);
        if(a == b){
            return true;
        }else {
            return false;
        }
    }

    public static void main(String[] args){
        int[] parent = new int[11];
        for(int i =1; i<= 10; i++){
            parent[i] = i; //모든 값이 자기 자신을 가르키도록 만든다. 
        }

        unionParent(parent, 1, 2);
        unionParent(parent, 2, 3);
        unionParent(parent, 4, 7);
        unionParent(parent, 5, 6);

        System.out.println(findParent(parent, 2, 6));
        System.out.println(getParent(parent, 3));

    }

}
```

출처 : [17. Union-Find(합집합 찾기)](https://m.blog.naver.com/ndb796/221230967614)