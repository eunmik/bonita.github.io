---
layout: post
title: "[Java]Comparable vs Comparator"
date: 2021-03-22
categories: [Language, Java]
tags: [Java]
comments: true
---
알고리즘 스터디를 하면서 다른 사람들이 쓴 코드를 접하게 됬는데,
내가 몰랐던 기능들을 사용하여 아주 간단하게 코드를 작성 한다. 
너무 멋있어......😎
그래서 오늘은 정렬 문제를 풀면서 알아낸 Comparator을 알아보자! 

# Java Comparable Comparator

자바에서 배열이나 리스트를 정렬 할 때 java.util.Arrays 클래스의 sort() 메서드로 간편하게 정렬할 수 있다. 

## 배열의 오름차순 정렬

Arrays.sort()메서드의 매개값으로 기본 타입 배열이나 String 배열을 지정해주면 자동으로 오름차순 정렬이 된다. 

### 기본 타입 배열 오름차순 정렬

```java
import java.util.Arrays;

public class Sort{
    public static void main(String[] args)  {
        int arr[] = {4,23,33,15,17,19};
        Arrays.sort(arr);
        
        for (int i : arr) {
            System.out.print("["+i+"]");
        }
    }
}
```

[4][15][17][19][23][33]

### String 배열 오름차순 정렬

```java
import java.util.Arrays;

public class Sort{
    public static void main(String[] args)  {
        String arr[] = {"apple","orange","banana","pear","peach","melon"};
        Arrays.sort(arr);
        
        for (String i : arr) {
            System.out.print("["+i+"]");
        }
    }
}
```

[apple][banana][melon][orange][peach][pear]

## 배열의 내림차순 정렬

배열을 내림차순으로 정렬할 때는 Collections 클래스의 reverseOrder()함수를 사용하면 된다. 만약 기본 타입 배열을 내림차순으로 정렬하고 싶다면 기본 타입의 배열을 래퍼클래스로 만들어 Comparator를 두번째 인사에 넣어주어야 역순으로 정렬할 수 있다. 

### 기본 타입 배열 내림차순 정렬

```java
import java.util.Arrays;

public class Sort{
    public static void main(String[] args)  {
        Integer arr[] = {4,23,33,15,17,19};
        Arrays.sort(arr,Collections.reverseOrder());
        
        for (int i : arr) {
            System.out.print("["+i+"]");
        }
    }
}
```

[33][23][19][17][15][4]

### String 배열 내림차순 정렬

```java
import java.util.Arrays;

public class Sort{
    public static void main(String[] args)  {
        String arr[] = {"apple","orange","banana","pear","peach","melon"};
        Arrays.sort(arr,Collections.reverseOrder());
        
        for (String i : arr) {
            System.out.print("["+i+"]");
        }
    }
}
```

[pear][peach][orange][melon][banana][apple]

## 배열 일부분만 정렬

Arrays.sort(0 메서드의 매개값으로 배열, 시작 index, 끝을 넣어주면 일부분만 정렬할 수 있다 

```java
import java.util.Arrays;

public class Main{
   public static void main(String[] args)  {
        int arr[] = {4,23,33,15,17,19};

        Arrays.sort(arr, 0, 4); // 0,1,2,3 요소만 정렬
        
        for (int i : arr) {
            System.out.print("["+i+"]");
        }
    }
}
```

[4][15][23][33][17][19]

## 객체 배열 정렬

객체로만 이루어진 배열의 경우에는 객체 클래스가 Comparable 인터페이스의 cpmareTo() 메서드를 구현하고 있어야 정렬이 된다. 

Comparable 인터페이스의 compareTo() 메서드를 통해 인자로 넘어온 같은 타입의 다른 객체와 대소 비교가 가능하다. 

메서드를 호출하는 객체가 인자로 넘어온 객체보다 작을 경우에는 음수를 리턴하고, 크기가 동일하다면 0, 클 경우에는 양수를 리턴해야 한다. 

```java
public class Player implements Comparable<Player> {
    private String name;
    private int score;

    public Player(String name, int score) {
        this.name = name;
        this.score = score;
    }

    // Getters, Setters 생략
	  @Override
    public int compareTo(Player o){
			return o.getScore() - getScore();
	}
}

public class Sort{
    public static void main(String[] args) {
			List<Player> players = new ArrayList<>();
			players.add(new Player("Alice", 899));
			players.add(new Player("Bob", 982));
			players.add(new Player("Chloe", 1090));
			players.add(new Player("Dale", 982));
			players.add(new Player("Eric", 1018));
			Collections.sort(players);
		}
}
```

## Comparator 객체 사용

만약 정렬 대상 클래스의 코드를 직접 수정할 수 없는경우에는 어떻게 객체의 정렬 기준을 정의할 수 있을까? 또는 정렬하고자 하는 객체에 이미 존재하고 있는 정렬 기준과 다른 정렬 기준으로 정렬하고 싶을 때는 어떻게 해야 할까? 

이 때 필요한 것이 바로 Comparator 인터페이스이다. 

Comparator 인터페이스의 구현체를 Arrays.sort()나 Collections.sort()와 같은 정렬 메서드의 추가 인자로 넘기면 정렬 기준을 누락된 클래스의 객체나 기존 정렬 기준을 무시하고 새로운 정렬 기준으로 객체를 정렬 할 수 있다. 

```java
Comparator<Player> comparator = new Comparator<Player>() {
    @Override
    public int compare(Player a, Player b) {
        return b.getScore() - a.getScore();
    }
};

Collections.sort(players, comparator);
```

```java
Collections.sort(players, (a, b) -> b.getScore() - a.getScore());
```