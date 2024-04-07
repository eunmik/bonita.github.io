---
layout: post
title: "[TDD] 4장 TDD with mock"
date: 2021-04-09
categories: [Language, Java]
tags: [Java, TDD]
comments: true
---

## Mock 객체란 무엇인가?

조작하기 쉬운 재료(보통 나무나 점토 등)를 이용해 추후 만들어질 제품의 외양을 흉내 낸 모조품을 말한다. 마찬가지로 소프트웨어 개발에 있어서도 모듈의 겉모양이 실제 모듈과 비슷하게 보이도록 만든 가짜 객체를 Mock 객체라고 한다. 

example.

```java
@Test
    public void testSavePassword() throws Exception {
        UserRegister register = new UserResgister();
        Cipher cipher = new MockMD5Cipher() ;
        String userId = "test88";
        String pwd = "potato";

        register.savePassword(userId, chipher.encrypt(password)_;
        String decryptedPwd = cipher.decrypt(register.getPassword(userId));
        assertEquals(pwd, register.decryptedPwd);
    }

    public class MockMD5Cipher implements Cipher {
        @Override
        public String decrypt(String source){
            return "potato";
        }
        @Override
        public String encrypt (String source){
            return "8ee2027983915ec78acc45027d874316";
        }

    }
```

## 언제 Mock 객체를 만들 것인가?

- 테스트 작성을 위한 환경 구축이 어려워서
- 테스트가 특정 경우나 순간에 의존적이라서
- 테스트 시간이 오래 걸려서

## Mock에 대한 기본적인 분류 개념, 테스트 더블

오리지널 객체를 사용해서 테스트를 진행하기가 어려울 경우 이를 대신해서 테스트를 진행할 수 있도록 만들어주는 객체를 지칭한다. 

<img src="https://repo.yona.io/files/3996">

### 더미 객체(Dummy Object)

더미 객체는 말 그대로 멍청한 모조품, 단순한 껍데기에 해당한다. 오로지 인스턴스화될 수 있는 수준으로만 인터페이스를 구현한 객체다.  단지 인스턴스화된 객체가 필요할 뿐 해당 객체의 기능까지는 필요하지 않은 경우에 사용한다. 

### 테스트 스텁(Test Stub)

더미 객체가 마치 실제로 작동하는 것처럼 보이게 만들어놓은 객체다. 객체의 특정 상태를 가정해서 만들어놓은 단순 구현제다. 

### 페이크 객체(Fake Object)

여러 개의 인스턴스를 대표할 수 있는 경우이거나, 좀 더 복잡한 구현이 들어가 있는 객체를 지칭한다. 복잡한 로직이나, 객체 내부에서 필요로 하는 다른 외부 객체들의 동작을, 비교적 단순화하여 구현한 객체이다. 

### 테스트 스파이(Test Spy)

테스트에 사용되는 객체에 대해서, 특정 객체가 사용됐는지, 그리고 그 객체의 예상된 메소드가 정상적으로 호출됐는지를 확인해야 하는 상황이 발생한다. 보통은 호출 여부를 몰래 감시해서 기록했다가, 나중에 요청이 들어오면 해당 기록 정보를 전달해준다. 그런 목적으로 만들어진 테스트 더블을 테스트 스파이라고 부른다. 

### Mock 객체(Mock Object)

일반적인 테스트 더블은 상태(state)를 기반으로 테스트 케이스를 작성한다. 

Mock 객체는 행위(behabiour)를 기반으로 테스트 케이스를 작성한다. 

> 💡 상태 기반 테스트 vs 행위 기반 테스트
> - 상태 기반 테스트(state base test) 
> 
> 특정한 메소드를 거친 후, 객체의 상태에 대해 예상값과 비교하는 방식이 상태 기반 테스트이다. 
>
> - 행위 기반 테스트(behaviour base test) 
> 
> 올바른 로직 수행에 대한 판단의 근거로 특정한 동작의 수행 여부를 이용한다. 보통은 메소드의 리턴값이 없거나 리턴값을 확인하는 것만으로는 예상대로 동작했음을 보증하기 어려운 경우에 사용한다. 

Mock 객체는 행위를 검증하기 위해 사용되는 객체를 지칭한다. 수동으로 만들 수도 있고, Mock 프레임워크를 이용할 수도 있다. 

## Mock 프레임워크 - Mockito

### Mockito의 특징

 - Mock 프레임워크는 어때야 하는가? 

- 단순해야 한다.
- Mock 프레임워크를 DSL로 만들지 말자. 복잡해진다.
- 문자열을 메소드 대신에 사용하지 말자.
- 읽기 어려운 anonymous inner 클래스를 사용하지 말자.
- 리팩토링이 어려워서는 안 된다.

- Mockito 프레임워크의 차별점은 무엇인가? 

- 테스트 그 자체에 집중한다.
- 테스트 스텁을 만드는 것과 검증을 분리시켰다.
- Mock 만드는 방법을 단일화했다.
- 테스트 스텁을 만들기 쉽다.
- API가 간단하다.
- 프레임워크가 지원해주지 않으면 안되는 코드를 최대한 배제했다.
- 실패 시에 발생하는 에러추적(stack trace)이 깔끔하다.

## 기본 사용법

|    |           | 
|----------|-------------|
| CreateMock |  인터페이스에 해당하는 Mock 객체를 만든다.   | 
| Stub |    테스트에 필요한 Mock 객체의 동작을 지정한다(단, 필요 시에만).   | 
| Excercise | 테스트 메소드 내에서 Mock 객체를 사용한다. | 
| Verify | 메소드가 예상대로 호출됐는지 검증한다. | 


1. Mock 객체 만들기 

    ```java
    Mockito.mock(타깃 인터페이스); 
    ```

    example 

    ```java
    import static org.mockito.Mockito.*;
    List mokedList = mock(List.class);
    ```

    Mockito 클래스의 static 메소드인 mock을 이용해 인터페이스나 클래스를 지정한다. 

    이후 부터는 구현 클래스로 객체가 생성된 것처럼 동작한다. 

2. 예상값 지정

Mockito는 예상값을 지정하는 것이 아니라, 필요 시에 테스트 스텁만 만들고 나중에 호출 여부를 확인하는 방식을 사용한다. 그래서 예상값 지정 부분이 없다. 

3. 테스트에 사용할 스텁 만들기 

```java
when(Mock_객체의_메소드).thenReturn(리턴값);
when(Mock_객체의_메소드).thenThrow(예외);
```

단, 스텁은 필요할 때만 만드는 것이 원칙이다. 

```java
List mockedList = mock(List.class);

//Mock 객체를 사용한다. 
mockedList.add("item");
mockedList.clear();

//검증
verify(mockedList).add("item");
verify(mockedList).clear(); 

// Stub 만들기
when(mockedList.get(0)).thenReturn("item");
when(mockedList.size()).thenReturn(1);
when(mockedList.get(1)).thenThrow(new RuntimeException());
System.out.println(mockedList.get(0));
System.out.println(mockedList.size());
System.out.println(mockedList.get(2));
System.out.println(mockedList.get(1));
```

4. 검증

```java
verify(Mock_객체).Mock_객체의_메소드;
verify(Mock_객체, 호출횟수지정_메소드).Mock_객체의_메소드;
```

Mock 객체의 특정 메소드가 호출됐는지 확인한다. 

호출 횟수도 지정해서 검증 할 수가 있다. 

호출 횟수 지정 메소드 종류
|    |           | 
|----------|-------------|
| items(n) |  n번 호출 됐는지 확인. n=0은 items를 지정하지 않았을 때와 동일하다.   | 
| never |    호출되지 않았어야 함   | 
| atLeastOnce | 최소 한 번은 호출됐어야 함 | 
| atLeast(n) | 적어도 n번은 호출됐어야 함 | 
| atMost(n) | 최대 n번은 호출되면 안 됨 | 

```java
verify(mockedList).add("item");
verify(mockedList, times(1)).add("item");
verify(mockedList, times(2)).add(box);
verify(mockedList, never()).add(car);
verify(mockedList, atLeastOnce()).removeAll();
verify(mockedList, atLeast(2)).size();
verify(mockedList, atMost(5)).add(box);
```

a.  Argument Matcher

인자는 특정하지 하고 메소가 n번 사용됐기만 하면 될 때 사용하는 것이 Argument Matcher이다. 

|    |           | 
|----------|-------------|
| any |  어떤 객체가 됐든 무방   | 
| any 타입 |    anyInteger( ), anyBoolean 등 Java 타입에 해당하는 any 시리즈가 있다. 이런 시리즈들은 null이거나 해당 타입이면 만족한다.   | 
| anyCollection anyCollectionOf | List, Map, Set 등 Collection 객체이면 무방. anyCollectionOf는 anyCollection과 동일하다. 자연스런 문장을 위해 사용한다. | 
| argThat(HamcrestMatcher) | Mockito도 Hamcrest Matcher를 사용하고 있다. Hamcrest에서 사용하던 Matcher를 그대로 이 부분에서 사용할 수 있다. | 
| eq | Argument Matcher가 한번 사용된 부분에선 Java의 타입을 그대로는 더 이상 쓸 수가 없다. verify(mock).add(anyString(), “item”); 즉, 위와 같이는 쓸 수 없다. 이럴 때 아래와 같이 eq를 이용한다. verify(mock).put(anyString(9), eq(“item”)); | 
| anyVarargeq | 여러 개의 인자를 지칭할 때 사용한다. 예) // verification mock.foo(1, 2); mock.foo(1, 2, 3, 4); verify(mock, times(2)).foo(anyVararg( )); // Stubbing: when(mock.foo(anyVararg( )).thenReturn(100); | 
| matches(String regex) | 정규식 문자열로 인자(argument) 대상을 지칭한다. | 
| startswith(String) endWith(String) | 특정 문자열로 시작하거나 끝나면 OK | 
| anyList anyMap anySeteq | anyCollection의 좀 더 디테일한 버전이다. 해당 타입이기만 하면 OK | 
| isA(Class) | 해당 클래스 타입이기만 하면 된다 | 
| isNull | Null이면 OK | 
| isNotNull | null만 아니면 OK | 




b. 순서 검증

만일 Stub으로 만들어진 Mock 객체 메소드의 호출 순서까지 검증하고 싶다면 InOrder클래스를 이용한다. 

```java
List firstMock = mock(List.class);
List secondMock = mock(List.class);
firstMock.add("item1");
secondMock.add("item2");
InOrder inOrder = inOrder(firstMock, secondMock);
inOrder.verify(firstMock).add("item1");
inOrder.verify(secondMock).add("item2");
```

## Mockito의 특징적인 기능

1. void 메소드를 Stub으로 만들기 

    ```java
    doThrow(예외).when(Mock_객체).voidMethod();
    ```

2. 콜백으로 Stub 만들기: thenAnswer

    특정 Mock 메소드에 대해 실제 로직을 구현하고자 할 때 콜백 기법을 사용한다. 

    ```java
    when(rs.getInt("no")).thenAnswer( new Answer<Integer>(){
    public Integer answer(InvocationOnMock invocation) throws Throwable {
    if (currentRecord = = null)
    throw new SQLException("access fields is empty");
    return ((Integer)currentRecord[0]).intValue();
    }
    });
    ```

3. 실체 객체를 Stub으로 만들기: SPY

    Mockito는 실 객체도 Mock으로 만들 수 있다. Mockito의 강력한 기능 중 하나인데, 너무 강력해서 오히려 문제가 될 수도 있는 기능이다

    ```java
    ArrayList<String> realList = new ArrayList<String>();
    realList.add("Hello");
    System.out.println(realList.get(0));
    List mockedList = spy( realList );
    when(mockedList.get(0)).thenReturn("item");
    System.out.println(mockedList.get(0));
    -----실행 결과-----
    Hello
    item
    ```

4. 똑똑한 NULL 처리: SMART NULLS

필요에 따라, 좀 더 유용한 값이 기본 값으로 찍히게 만드는 것이 SMART NULLS라는 방식이다. 

```java
List mockedList = mock(List.class);
List smartMockedList = mock(List.class, RETURNS_SMART_NULLS);
System.out.println(mockedList.toArray());
System.out.println(smartMockedList.toArray());
-----실행 결과-----
null
[LJava.lang.Object;@3eca90
```

SMART NULLS 규칙 
- 기본형 래퍼(primitive wrapper) 클래스는 해당 기본형 값으로 바꾼다. 
- String은 ""로 바꾼다. 
- 배열은 크기 0인 기본 배열 객체로 만들어준다.
- Collection 계열은 빈 Collection 객체로 만든다. 

5. 행위 주도 개발(BDD) 스타일 지원

Mockito는 [//given](//given) [//when](//when) [//then](//then) 식의 행위 주도 개발(Behabiour-Driven Development, BDD) 스타일로 테스트 케이스 작성할 수 있게 지원해준다. BDD 스타일을 사용하려면 Mockito 클래스 대신 BDDMockito를 static import 한다. 

```java
import static org.mockito.BDDMockito.*;
Seller seller = mock(Seller.class);
Shop shop = new Shop(seller);
public void shouldBuyBread() throws Exception {
	// given
	given(seller.askForBread()).willReturn(new Bread());
	// when
	Goods goods = shop.buyBread();
	// then
	assertThat(goods, containBread());
}
```