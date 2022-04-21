---
layout: post
title: "[JAVA] Comparator and CompareTo"
date: 2022-04-21
excerpt: "What is comparator and compareto? How to use them?"
tags: ["java", "comparator", "compareTo"]
comments: true
---
# What is Java Comparator?

Java Comparator is a interface for sorting Java objects. Comparators can be passed to a sort method (such as Collections.sort or Arrays.sort) to allow precise control over the sort order. 

## Compare Method

Method `compare` compares its two arguments for order and returns a negative integer, zero, or a positive integer as the first argument is less than, equil to, or greater than the second. 

### Parameters :

o1 - the first object to be compared.

o2 - the second object to be compared.

### Returns :

a negative integer : the first argument is less than the second.

zero : the first argument is equal to the second

a positive integer : the first argument is greater than the second. 

| Returns | Meaning |
| --- | --- |
| negative integer | the first argument should be on the left |
| zero | the first argument and the second argument are the same |
| positive integer | the first argument should be on the right |

## Example [c,a,eb,cd]

```java
Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                if(o1.length() == o2.length()){
                    for(int i =0; i<o1.length(); i++){
                        return o1.compareTo(o2); //sort ascending 
                    }
                }
                return  o1.length() - o2.length();

            }
        });
```

| o1 | o2 | return | meaning |
| --- | --- | --- | --- |
| c | a | psotivie | c should be on the right |
| a | c | nagative | a should be on the left  |
| c | eb | negative | c should be on the left |
| eb | cd | positive | eb should be on the right |

Result 

```java
[a, c, cd, eb]
```

# What is Java String compareTo()?

The Java String compareTo() method is used for comparing two strings lesicographically. Each character of both the strings is converted inta a Unicode value for comparison. If both the strings are equal then this method returns 0 else it returns positive or negative value. 

### Example

```java
public class JavaExample {
    public static void main(String args[]) {
	String str1 = "Cow"; 
	//This is an empty string
	String str2 = "";
	String str3 = "Goat";
		
	System.out.println(str1.compareTo(str2));

	System.out.println(str2.compareTo(str3));
   }
}
```

Output

```java
3
-4
```

reference : 

[The Java String compareTo(),returns positive or negative value](https://beginnersbook.com/2013/12/java-string-compareto-method-example/#:~:text=The%20Java%20String%20compareTo(),returns%20positive%20or%20negative%20value, "").

[Comparator.html#compare-T-T-](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#compare-T-T-, "")