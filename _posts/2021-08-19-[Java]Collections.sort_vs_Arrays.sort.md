---
layout: post
title: "[Java] Collections.sort vs Arrays.sort "
date: 2021-08-20
excerpt: "Java에서 정렬하는 방법"
tags: [Java, sort]
comments: true
---
## Arrays.sort (오름차순)

Arrays.sort는 java.util.Arrays에 포함되어 있다. 

Arrays.sort는 배열을 파라미터로 받는다. (int[], String[], char[], long[] 등등) 

```java
String[] str = {"1", "a", "banana", "2", "B"};
        System.out.println("BEFORE");
        for(String s : str){
            System.out.print(s+" ");
        }
        System.out.println();
        Arrays.sort(str);
        System.out.println("AFTER");
        for(String s : str){
            System.out.print(s+" ");
        }
```

output

```java
BEFORE
1 a banana 2 B 
AFTER
1 2 B a banana
```

## Arrays.sort(내림차순)

Array.sort(String[] a, Comparator<? super String> c) c 파라미터 자리에 

Collections.reverseOrder()를 넣어주면 내림차순으로 정렬이 된다. 

```java
	String[] str = {"1", "a", "banana", "2", "B"};
	System.out.println("BEFORE");
	for(String s : str){
	    System.out.print(s+" ");
	}
	System.out.println();
	Arrays.sort(str, Collections.reverseOrder());
	System.out.println("AFTER");
	for(String s : str){
	    System.out.print(s+" ");
	}
```

output

```java
BEFORE
1 a banana 2 B 
AFTER
banana a B 2 1
```

## Collections.sort

Collections.sort는 List를 파라미터로 받는다. 

```java
	List<String> list = new ArrayList<>();
	list.add("1");
	list.add("a");
	list.add("banana");
	list.add("2");
	list.add("B");
	Collections.sort(list);
	System.out.println("오름차순");
	for (String s : list) {
	    System.out.print(s + " ");
	}
	System.out.println();
	System.out.println("내림차순");
	Collections.reverse(list);
	for (String s : list) {
	    System.out.print(s + " ");
	}
```

output

```java
오름차순
1 2 B a banana 
내림차순
banana a B 2 1
```