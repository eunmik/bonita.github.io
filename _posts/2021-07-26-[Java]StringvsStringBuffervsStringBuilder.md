---
layout: post
title: "[Java] String vs StringBuffer vs StringBuilder"
date: 2021-07-26
excerpt: "String과 StringBuffer와 StringBuilder의 차이"
tags: [Java, String, StringBuilder, StringBuffer]
comments: true
---

```java
public static void main(String[] args){
    String stringValue1 = "TEST 1";
    String stringValue2 = "TEST 2";

    System.out.println("stringValue1: " + stringValue1.hashCode()); //-1823841245
    System.out.println("stringValue2: " + stringValue2.hashCode()); //-1823841244

    stringValue1 = stringValue1 + stringValue2;
    System.out.println("stringValue1: " + stringValue1.hashCode()); //833872391

    StringBuffer sb = new StringBuffer();

    System.out.println("sb: " + sb.hashCode()); //876563773

    sb.append("TEST StringBuffer");
    System.out.println("sb: " + sb.hashCode()); //876563773
}
```

같이 생성된 클래스의 주소값이 다르다. String은 새로운 값을 할당할 때 마다 새로 생성되기 때문이다. 이와 달리 StringBuffer는 값은 memory에 append 하는 방식으로 클래스를 직접 생성하지 않는다. 

논리적으로 따져보면 클래스가 생성될 때 method들과 variable도 같이 생성되는데, StringBuffer는 이런 시간을 사용하지 않는다. 

수십번 String이 더해지는 경우에는 각 String의 주소값이 stack에 쌓이고 클래스들은 Garbage Collector가 호출되기 전까지 heap에 지속적으로 쌓이게 된다. 메모리 관리적인 측면에서는 치명적이라고 볼 수 있다. 

---

String class의 내부 구조를 보면 String에서 저장되는 문자열은 알고보면 char의 배열형태로 저장되며 이 값들은 외부에서 접근할 수 없도록 private으로 보호된다. 또한 final형이기 때문에 초기값으로 주어진 String의 값은 불변으로 바뀔 수가 없게 되는 것이다. (immutable)

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0726/img1.png" />

---

StringBuilder는 변경가능한 문자열(mutable)이지만 synchronization이 적용되지 않는다. 하지만 StringBuffer는 thread-safe라는 말에서처럼 변경가능하지만 multiple thread 환경에서 안전한 클래스라고 한다. 

이것이 StringBuilder와 StringBuffer의 가장 큰 차이점이다. 

예를 들어, text를 수정해야하고 multiple threads를 사용하고 있다면 StringBuffer를 사용하는게 낫다. 만약 text를 수정해야하고 single thread를 사용한다면 StringBuilder를 사용하는게 낫다. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0726/img2.png" />
---

StringBuilder Methods insert() vs append()

- insert()는 StringBuilder의 특정 인덱스에 추가할 수 있도록 한다.
- append()는 StringBuilder object의 마지막에 추가하도록 한다.
- StringBuilder 마지막에 insert()를 한다면 append()와 시간복잡도 면에서 동일하다. (O(n))

```java
public class StringBuilderDemo {
    public static void main(String[] args){
        String palindrome = "Dot saw I was Tod";
        StringBuilder sb = new StringBuilder(palindrome);
        sb.reverse(); // reverse it
        System.out.println(sb); //doT saw I was toD

        sb.append(" by Eunmi ");
        sb.append("let it go ").append(true);
        System.out.println(sb); //doT saw I was toD by Eunmi let it go true
        
        sb.insert(2, false);
        sb.insert(sb.length(), "haha");
        System.out.println(sb); // dofalseT saw I was toD by Eunmi let it go truehaha
    }
}
```

```java
public class StringDemo {
    public static void main(String[] args){
        String palindrome = "Dot saw I was Tod";
        int len = palindrome.length();
        char[] tempCharArray = new char[len];
        char[] charArray = new char[len];

        //put original string in an array of chars
        for (int i = 0; i < len; i++) {
            tempCharArray[i] = palindrome.charAt(i);
        }

        //reverse array of chars
        for (int j = 0; j < len; j++) {
            charArray[j] = tempCharArray[len - 1 - j];
        }

        String reversePalindrome = new String(charArray);
        System.out.println(reversePalindrome); //doT saw I was toD

    }
}
```

출처 [https://novemberde.github.io/2017/04/15/String_0.html](https://novemberde.github.io/2017/04/15/String_0.html), [https://www.atechdaily.com/posts/Difference-between-StringBuilder-insert-and-append-method](https://www.atechdaily.com/posts/Difference-between-StringBuilder-insert-and-append-method)
[https://docs.oracle.com/javase/tutorial/java/data/buffers.html](https://docs.oracle.com/javase/tutorial/java/data/buffers.html)


