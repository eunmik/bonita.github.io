---
layout: post
title: "[Algorithm] 조합을 사용해 경우의 수 구하기"
date: 2021-07-16
excerpt: "경우의 수를 구해보자."
tags: [Java, Combination, number_of_cases]
comments: true
---
## 조합이란?

조합이란 n개의 숫자 중에서 r개의 수를 순서 없이 뽑는 경우를 말한다. 

예를 들어 [1, 2, 3] 배열에서의 조합은 아래와 같이 나온다. 

```java
[1, 2]
[1, 3]
[2, 3]
```

순열 이라는 것은 주어진 수열에서 순서에 따라 결과가 달라지는 방식을 순열이라고 한다. 말 그대로, 순서가 존재하는 열이라는 것이다. 

즉 순열에서 {1,2,3}과 {1,3,2}, {2,1,3}등 모두 다른 결과를 가져온다. 순서가 다르기 때문이다. 

조합은 순서가 상관이 없는 모임을 의미한다. 

순서가 상관이 없기 때문에 {1,2,3}, {1,3,2}, {2,1,3} 모두 같은 것으로 취급을 한다. 즉, 1,2,3 이라는 3개의 숫자로 이루어져 있다는 점에서 똑같은 취급을 하는 것이 조합이다. 

## 조합의 원리

조합의 기호는 nCr로 나타내며 

$nCr = n-1Cr-1 + n-1Cr$ 로 나타낼 수 있다. 

즉, 조합은 **하나의 원소를 선택할 경우 + 하나의 원소를 선택하지 않을 경우 이 둘의 합**을 나타낸다. 

예를 들어, (1,2,3)중에서 2개를 뽑는 조합이라고 생각해보면 → 3C2

- (1,X) : 1을 뽑는 경우 (하나의 원소를 선택할 경우)
- (X,X) : 1을 뽑지 않는 경우 (하나의 원소를 선택하지 않는 경우)

이처럼 2가지로 나뉠 수 있다. 

- 1을 뽑은 경우 나머지 (2,3) 중 1개를 선택해야 함. (총 2개의 경우 (1,2) (1,3)) → 2C1
- 1을 뽑지 않은 경우 (2,3) 모두 선택해야 한다. (총 1개의 경우 (2,3)) → 2C2

즉, (1,2,3)에서 2개를 뽑는 조합은 둘을 합해 (1,2)(1,3)(2,3) 총 3가지가 된다. 

또다른 예로 (1,2,3,4,5) 중에 3개를 뽑는다고 가정했을 때 

3을 포함하고 {1,2,4,5} 중에 나머지 2개를 뽑으면 4C2이고, 

3을 포함하지 않고 {1,2,4,5} 중에 3개를 뽑으면 4C3이다. 

두 경우를 합하면 결국 3개 중에서 3개를 뽑는것이 된다. 

이게 조합의 기본원리이고 특징이다. 이를 바탕으로 더 이상 쪼개지지 않을 때 까지 즉, nC0 = 1이 될 때까지 구한다면 nCr을 구할 수 있게 되는 것이다. 

## 1. 조합 경우의 수 구하기

3개 중에서 2개를 뽑는 경우 

3C2 = 2C1 + 2C2 > ... > 3C0 = 1이 될 때까지 재귀 호출을 통해 구현한다. 

- 재귀를 통해 호출을 하다가 r == 0이 될 경우, 즉 r개를 다 뽑은 경우는 더 이상 선택의 여지가 없으므로 1을 리턴.
- 전채 개수 n이 뽑아야 할 개수 r과 같아졌다면 역시 다 뽑는 것 말고 선택의 여지가 없으므로 1을 리턴.
- 그 이외에는 원소를 선택할 경우 + 선택하지 않을 경우 둘의 합을 계속 재귀로 호출.

```java
public static void main(String[] args){
        System.out.println(combination(3,2)); //3
        /**
         * (1,2,3) -> (1,2),(1,3),(2,3) 
         */
    }
    //조합의 경우의 수 구하기
    public static int combination(int n, int r){
        if(n == r || r == 0)
            return 1;
        else
            return combination(n - 1, r - 1) + combination(n - 1, r);
    }
```

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0716/img1.png" />

## 2. 조합 구하기

조합을 구하는 방법은 배열의 처음부터 마지막까지 돌며 

1. 현재 인덱스를 선택하는 경우 
2. 현재 인덱스를 선택하지 않는 경우 

이렇게 두가지로 모든 경우를 완전 탐색하는 것이다. 

- arr : 조합을 뽑아낼 배열
- r : 조합의 길이 (뽑을 갯수)
- visited : 조합에 뽑혔는지 확인하기 위한 배열

순열과 달리 조합은 r을 유지할 필요가 없으므로 숫자를 하나 뽑을 때마다 하나씩 줄여준다. 

r == 0 일 때가 r 개의 숫자를 뽑은 경우이다. 

### 백트래킹의 방법

start 변수를 기준으로 start보다 작으면 뽑을 후보에서 제외, 

start 보다 크면 뽑을 후보가 된다. 

예를 들어, [1,2,3] 배열이 있고 start가 0부터 시작한다. 

조합을 뽑는 comb 함수에 들어오면 먼저 i 정점부터 시작하는 for문에 들어간다. 

만약 0 인덱스에 있는 값 1을 뽑는 다면 visited[i] = true가 되고 아니면 visited[i] = false가 된다. 

즉, 1을 선택한 경우 (visited[i] = true)와 선택하지 않은 경우(visited[i] = false) 둘다 봐야 한다. 

하지만 더이상 1은 고려 대상이 아니기 때문에 다음 for 문은 2부터 시작해주어야 한다. 

```java
//경우의 수를 구해보자.
public class NumberOfCases {
    public static void main(String[] args){
        comb1(new int[]{1,2,3}, new boolean[3], 0, 3, 2);
    }
 
    //백트래킹
    static void comb1(int[] arr, boolean[] visited, int start, int n, int r){
        if(r == 0){
            print(arr, visited);
            return;
        } else {
            for(int i = start; i<n;i++){
                visited[i] = true;
                comb1(arr, visited, i + 1, n, r - 1);
                visited[i] = false;
            }
        }
    }

    static void print(int[] arr, boolean[] visited){
        for(int i=0; i<arr.length; i++){
            if(visited[i] == true){
                System.out.print(arr[i] + " ");
            }

        }System.out.println();
    }

}
```

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0716/img2.png" />

### 재귀 이용해 구현

depth 변수는 현재 인덱스이고 0부터 시작한다.

현재 인덱스를 뽑는다면 = true, 뽑지 않는다면 visited[depth] = false로 진행한다. 

마찬가지로 뽑은 경우와 뽑지 않은 경우 모두 봐야하고, 그때 이전에 본 값들은 더 이상 고려 대상이 아니기 때문에 무조건 1씩 증가한다. 

depth == n 이 되면 모든 인덱스를 다 보았으므로 (배열의 끝에 도착 → 조합 만들 수 x) 재귀를 종료한다. 

또한 r == 0 이되면 뽑을 개수를 다 뽑아 조합이 완성되니 재귀를 종료한다. 

```java
//2. 재귀 사용해 구현 
    static void comb2(int[] arr, boolean[] visited, int depth, int n, int r) {
        if(r == 0) {
            print(arr, visited, n);
            return;
        }
        if(depth == n) {
            return;
        } else {
            visited[depth] = true;
            comb2(arr, visited, depth + 1, n, r - 1);
 
            visited[depth] = false;
            comb2(arr, visited, depth + 1, n, r);
        }
    }
```

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0716/img3.png" />

### visited배열을 쓰지 않고 정수 변수 index로 판별해 구해보기

index변수는 0부터 시작해서 배열의 인덱스를 증가시키면서 해당 인덱스를 뽑을지 안뽑을지를 결정 해 조합을 만들기 위한 변수이다. 

(뽑는다면 index +1, r-1 / 안뽑는다면 index 그대로, r 그대로) 

target 변수는 0~n까지 나열된 원소 배열 안에서 어떤 숫자를 타깃으로 해 배열에 집어 넣을지 고를 때 쓰인다. 

combination()함수가 실행 될때마다 항상 +1씩 해 판별한다. 

index 변수에 따라 target에 있는 숫자가 뽑힐지 안뽑힐지가 결정된다. 

target+1이 nCr에서 n-1 → n-2 → n-3 → ... → 1 까지 가는 역할을 수행한다. 

```java
//visited 배열을 쓰지 않고 정수 변수 index로 판별해 구해보기
    static void combi3(int[] arr, int index, int n, int r, int target){
        if( r== 0) {
            for (int i = 0; i < index; i++) {
                System.out.print(arr[i] + " ");
            }
            System.out.println("");

        } else if (target == n){
          return;
        } else {
            arr[index] = target;
            combi3(arr, index + 1, n, r - 1, target + 1);
            combi3(arr, index, n, r, target + 1);
        }
    }
```

<img src ="https://eunmik.github.io/bonita.github.io/assets/img/2021/0716/img4.png" />