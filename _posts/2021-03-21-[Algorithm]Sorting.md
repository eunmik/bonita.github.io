---
layout: post
title: "[Algorithm]정렬(Sorting)란?"
date: 2021-03-21
excerpt: "알고리즘의 다양한 정렬에 대해 알아보자."
tags: [Algorithm, Selection, Insertion, Heap, Bubble, Merge]
comments: true
---

# 내부 정렬(internal Sorting)

데이터의 쿠기가 주 기억장소 용량보다 적을 경우 기억장소를 활용하여 정렬하는 방법
→ 버블 정렬, 삽입 정렬, 선택 정렬, 퀵 정렬, 쉘 정렬, 힙 정렬

## 내부 정렬의 종류 :
## 선택 정렬(Selection Sort)

- 입력 배열 (정렬되지 않은 값들) 이외에 다른 추가 메모리를 요구하지 않는다.
- 해당 순서에 원소를 넣을 위치는 이미 정해져 있고, 어떤 원소를 넣을지 선택하는 알고리즘
- 제자리 정렬 알고리즘의 하나
- 다음과 같은 순서로 이루어진다.
    1. 주어진 리스트 중에 최소 값을 찾는다. 
    2. 그 값을 맨 앞에 위치한 값과 교체한다(패스(pass))
    3. 맨 처음 위치를 뺀 나머지 리스트를 같은 방법으로 교체한다. 

    <img src="https://eunmik.github.io/bonita/assets/img/210321-selection.png">

시간 복잡도 : Θ(n2)

## 버블 정렬(Bubble Sort)

- 두 인접한 원소를 검사하여 정렬하는 방법
- 코드가 단순하기 때문에 자주 사용 된다.

```java
void bubbleSort(int[] arr) {
    int temp = 0;
	for(int i = 0; i < arr.length - 1; i++) {
		for(int j= 1 ; j < arr.length-i; j++) {
			if(arr[j]<arr[j-1]) {
				temp = arr[j-1];
				arr[j-1] = arr[j];
				arr[j] = temp;
			}
		}
	}
	System.out.println(Arrays.toString(arr));
}
```

<img src="https://eunmik.github.io/bonita/assets/img/210321-bubble.png">

시간 복잡도 : O(n^{2})

## 삽입 정렬(insertion sort)

- 자료 배열의 모든 요소를 앞에서부터 차례대로 이미 정렬된 배열 부분과 비교하여, 자신의 위치를 찾아 삽입하는 정렬
- 선택정렬, 거품 정렬과 비교하여 빠르다.

```java
void insertionSort(int[] arr)
{

   for(int index = 1 ; index < arr.length ; index++){

      int temp = arr[index];
      int aux = index - 1;

      while( (aux >= 0) && ( arr[aux] > temp ) ) {

         arr[aux+1] = arr[aux];
         aux--;
      }
      arr[aux + 1] = temp;
   }
}
```

<img src="https://eunmik.github.io/bonita/assets/img/210321-insertion.png">

시간 복잡도 : О(n2)

## 퀵 정렬(quick sort)

- 대표적인 분할 정복 알고리즘
- 다음과 같은 순서로 이루어진다.
    1. 리스트 가운데서 하나의 원소를 고른다. 이렇게 고른 원소를 피벗이라고 한다. 
    2. 피벗 앞에는 피벗보다 값이 작은 모든 원소들이 오고, 피벗 뒤에는 피벗보다 값이 큰 모든 원소들이 오도록 피벗을 기준으로 리스트를 둘로 나눈다. 이렇게 리스트를 둘로 나누는 것을 분할이라고 한다. 분할을 마친 뒤에 피벗은 더 이상 움직이지 않는다. 
    3. 분활된 두 개의 작은 리스트에 대해 재귀적으로 이 과정을 반복한다. 재귀는 리스트의 크기가 0이나 1이 될 때까지 반복된다. 

    ```java
    public void quickSort(int[] arr, int left, int right) {
        int i, j, pivot, tmp;
        if (left < right) {
            i = left;   j = right;
            pivot = arr[(left+right)/2];
            //분할 과정
            while (i < j) {
                while (arr[j] > pivot) j--;
                // 이 부분에서 arr[j-1]에 접근해서 익셉션 발생가능함.
                while (i < j && arr[i] < pivot) i++;

                tmp = arr[i];
                arr[i] = arr[j];
                arr[j] = tmp;
            }
            //정렬 과정
            quickSort(arr, left, i - 1);
            quickSort(arr, i + 1, right);
        }
    }
    ```

<img src="https://eunmik.github.io/bonita/assets/img/210321-quick.png">

시간 복잡도 : O(N * logN) 

## 힙 정렬(heap sort)

- 최대 힙 트리나 최소 힙 트리를 구성해 정렬을 하는 방법
- 다음과 같은 순서로 이루어 진다.
    1. n개의 노드에 대한 완전 이진 트리를 구성한다. 이 때 루트 노드부터 부모노드, 왼쪽 자식노드, 오른쪽 자식노드 순으로 구성한다. 
    2. 최대 힙을 구성한다. 최대 힙이란 부모노드가 자식노드보다 큰 트리를 말하는데, 단말 노드를 자식노드로 가진 부모노드부터 구성하며 아래부터 루트까지 올라오며 순차적으로 만들어 갈 수 있다. 
    3. 가장 큰 수(루트에 위치)를 가장 작은 수와 교환한다.
    4. 2와 3을 반복한다. 

<img src="https://eunmik.github.io/bonita/assets/img/210321-heap.png">

시간 복잡도 : O(log n)

# 외부 정렬(external sorting)

데이터의 크기가 주기억장소의 용량보다 클 경우 외부 기억장치(디스크, 테이프 등)를 사용하여 정렬하는 방법 

→ 머지 정렬

## 합병 정렬(Merge Sort)

- 비교 기반 정렬 알고리즘이다.
- 합병 정렬의 개념은 다음과 같다.
    1. 정렬되지 않은 리스트를 각각 하나의 원소만 포함하는 n개의 부분리스트로 분할한다. (한 원소만 든 리스트는 정렬된 것과 같으므로)
    2. 부분리스트가 하나만 남을 때까지 반복해서 병합하며 정렬된 부분리스트를 생성한다. 마지막 남은 부분리스트가 정렬된 리스트이다.
- 흔히 쓰이는 하향식 2-way 합병 정렬은 다음과 같이 작동한다.
    1. 리스트의 길이가 1 이하이면 이미 정렬된 것으로 본다. 그렇지 않은 경우에는
    2. 분할(divide) : 정렬되지 않은 리스트를 절반으로 잘라 비슷한 크기의 두 부분 리스트로 나눈다.
    3. 정복(conquer) : 각 부분 리스트를 재귀적으로 합병 정렬을 이용해 정렬한다.
    4. 결합(combine) : 두 부분 리스트를 다시 하나의 정렬된 리스트로 합병한다. 이때 정렬 결과가 임시배열에 저장된다.
    5. 복사(copy) : 임시 배열에 저장된 결과를 원래 배열에 복사한다.

<img src="https://eunmik.github.io/bonita/assets/img/210321-merge.png">

시간 복잡도 : 

## 시간 복잡도

<img src="https://eunmik.github.io/bonita/assets/img/210321-complexity.png">