---
layout: post
title: "MyBatis와 @Builder 어노테이션 사용 시 주의해야할 점"
date: 2025-01-14
categories: ["memoir, troubleshooting"]
tags: ["memoir, mybatis, builder, annotation"]
comments: true
---

**2025/01/15** 

🟠  이슈를 발견 하게 된 계기 
- 기존에 있던  VO(aka. Board VO) 에는 Class 레벨에 @Data 어노테이션이 달려있었다.
    - 참고로 @Data 어노테이션 안에는 
    **@ToString, 
    @EqualsAndHashCode, 
    @Getter on all fields, 
    @Setter on all non-final fields, and 
    @RequiredArgsConstructor** 이 존재한다.
- Board VO를 resultMap에 사용하는 Select 쿼리문이 있었다.
- 내가 Board VO에 빌더 패턴을 사용하기 위해서 @Builder 어노테이션을 추가했다.
- 여기서 문제 발생!!
- Board VO를 사용하는 기존 Select 쿼리문에서 에러 발생
    
    ```bash
    	java.sql.SQLDataException: value 'ThisIsTextNotLong' cannot be decoded as Long
    	at org.mariadb.jdbc.client.column.StringColumn.decodeLongText(StringColumn.java:232)
    	at org.mariadb.jdbc.client.result.rowdecoder.TextRowDecoder.decodeLong(TextRowDecoder.java:127)
    	at org.mariadb.jdbc.client.result.Result.getLong(Result.java:511)
    	at org.mariadb.jdbc.client.result.Result.getLong(Result.java:703)
    	at net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy.getLong(ResultSetSpy.java:2853)
    .........(생략)
    	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:223)
    	at jdk.proxy3/jdk.proxy3.$Proxy91.getPushApiList(Unknown Source)
    ........(생략)
    ```
    

🟠 원인 

- @Builder 어노테이션 class에 작성된 주석을 보면
    
    ```java
    /**
     * The builder annotation creates a so-called 'builder' aspect to the class that is annotated or the class
     * that contains a member which is annotated with {@code @Builder}.
     * <p>
     * If a member is annotated, it must be either a constructor or a method. If a class is annotated,
     * then a package-private constructor is generated with all fields as arguments
     * (as if {@code @AllArgsConstructor(access = AccessLevel.PACKAGE)} is present
     * on the class), and it is as if this constructor has been annotated with {@code @Builder} instead.
     * Note that this constructor is only generated if you haven't written any constructors and also haven't
     * added any explicit {@code @XArgsConstructor} annotations. In those cases, lombok will assume an all-args
     * constructor is present and generate code that uses it; this means you'd get a compiler error if this
     * constructor is not present.
    ```
    
    - 클래스에 builder 어노테이션이 적용 되었다면, 모든 필드를 매개변수로 가지는 package-private 레벨의 생성자가 생성된다.
    - 이 생성자는 @Builder 어노테이션이랑 같이 적용된 것처럼 동작한다.
    - 이 생성자는 사용자가 어떤 생성자도 작성하지 않고, 다른 생성자 관련 어노테이션(@XArgsConstructor)을 추가하지 않았을 때 만 생성된다.
    - 이런한 경우, lombok은 모든 매개변수가 있는 생성자가 존재한다고 가정하고, 해당 생성자를 사용하는 코드를 생성한다. 따라서, 이 생성자가 없다면 컴파일 에러가 발생할 것이다.
- @Builder 어노테이션을 사용하게 되면 기본 생성자(매개변수가 없는 빈 생성자)가 없이 모든 매개변수를 가지고 있는 생성자만 존재 하게 된다.
- MyBatis는 기본 생성자가 없으면 VO에 정의된 필드를 순서대로 Select 쿼리에서 조회한 데이터와 맵핑을 한다.
    - 예를 들어:
        
        [Car.java](http://Car.java) 라는 VO가 있을 때 
        
        ```java
        @Builder
        @Data
        public class Car {
        	private String type;
        	private Long price; 
        	private String plateNumber;
        }
        ```
        
        아래 처럼 같은 순서로 데이터를 조회하면 문제가 발생하지 않는다. BUT!! 
        
        CarMapper.xml (문제없음 🙆‍♀️)
        
        ```java
        <select id="getCars" resultMap="Car">
        	SELECT 
        			type 
        			, price
        			, plateNumber
        	FROM CAR 
        </select>
        ```
        
        VO에 정의된 데이터 타입의 순서가 String, Long, String 인데 
        
        Select 문으로 가져오는 데이터는 String, String Long 이기 때문에 
        
        SQLDataException이 발생하게 된다. 
        
        CarMapper.xml (문제있음 🙅‍♀️) 
        
        ```java
        <select id="getCars" resultMap="Car">
        	SELECT 
        			type 
        			, plateNumber
        			, price
        	FROM CAR 
        </select>
        ```
        

🟠 해결 : 

- 해결 방법은 Default Constructor를 추가해주는 것이다.
    
    ```java
    @Builder  
    @NoArgsConstructor  
    @AllArgsConstructor  
    @Data
    ```
    

🟠 분석 : 

- 롬복 어노테이션을 추가하고 빌드된 VO의 class에 만들어진 생성자를 비교해 보았다.
    
    @Data 또는 @Getter, @Setter 만 있을 때 
    
    ```java
    public Board() {} //빈생성자
    ```
    
    @Data, @Builder만 있을 때 
    
    ```java
    Board(전부 다) //public이 아님
    public static BoardBuilder builder() { //빌더 메서드
            return new BoardBuilder();
        }
    public static class BoardBuilder { //빌더패턴 클래스
    	필드 변수 다 있음
    	... 
    	BoardBuilder() {} //빌더패턴 클래스의 빈 생성자
    	...
    }
    ```
    
    @Data, @Builder, @NoArgsConstructor, @AllArgsConstructor 다 있을 때 
    
    ```java
    public Board(전부 다) 
    
    Public Board() //빈 생성자
    
    public static BoardBuilder builder() { //빌더 메서드
            return new BoardBuilder();
        }
    public static class BoardBuilder { //빌더패턴 클래스
    	필드 변수 다 있음
    	... 
    	BoardBuilder() {} //빌더패턴 클래스의 빈 생성자
    	...
    }
    ```
    
- 가장 큰 차이는 역시 빈 생성자가 있냐 없냐!
- MyBatis에서 데이터를 맵핑할때 기본생성자가 있냐 없냐에 따라 타는 로직이 다르다.
- DefaultResultSetHandler.java 안에 createResultObject() 메소드에서
- final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);
- metaType안에 디폴드 생성자가 있는지 확인 한다.
    
    if (resultType.isInterface() || metaType.hasDefaultConstructor()) 
    
    - 없으면 (중간 과정은 생략되었음)
        - applyConstructorAutomapping() → applyColumnOrderBasedConstructorAutomapping() 메소드에서 for문으로 자동으로 결과를 맵핑한다.
            
            ```java
            for (int i = 0; i < constructor.getParameterTypes().length; i++) {  
            		  Class<?> parameterType = constructor.getParameterTypes()[i];  
            		  String columnName = rsw.getColumnNames().get(i);  
            		  TypeHandler<?> typeHandler = rsw.getTypeHandler(parameterType, columnName);  
            		  Object value = typeHandler.getResult(rsw.getResultSet(), columnName);  
            		  constructorArgTypes.add(parameterType);  
            		  constructorArgs.add(value);  
            		  foundValues = value != null || foundValues;  
            	}
            ```
            
    - 있으면 (중간 과정은 생략되었음)
        - createAutomaticMappings() 메서드에서 이름을 찾아서 맵핑을 해준다.