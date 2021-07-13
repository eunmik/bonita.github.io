---
layout: post
title: "[Clean Code] 예외 처리"
date: 2021-07-12
excerpt: "Clean Code책에 나온 예외처리 내용"
tags: [Java, Exception, Try-Catch]
comments: true
---
# Table of contents

- [오류 코드보다 예외를 사용하라](#---)
- [Try-Catch-Finally 문 부터 작성하라](#try-catch-finally---)
- [미확인(unchecked) 예외를 사용하라](#unchecked--)
- [예외에 의미를 제공하라](#--)
- [호출자를 고려해 예외 클래스를 정의해라](#----)
- [정상 흐름을 정의하라.](#--)
- [null을 반환하지 마라](#null--)
- [null을 전환하지 마라](#null--)


예외 처리에 대해 알아보던 중 Clean Code라는 책에 예외 처리하는 구문이 정리된 블로그를 몇개 발견했다. 다음엔 내가 책 사서 다 읽고 정리해야지!! 

## 오류 코드보다 예외를 사용하라

오류 코드를 받아 처리 로직을 추가하는 것 보다 오류가 발생하면 예외를 던지는게 좋다. 

bad👎 : 함수를 호출한 즉시 오류를 확인하지 않으면 문제가 발생할 확률이 높음

```java
public class DeviceController{
	public void sendShuitDown(){
		DeviceHandle handle = getHandle(DEV1); 
		if(handle != DeviceHandle.INVALID) {
			retreiveDeviceRecord(handle);
			if(record.getStatus() != DEVICE_SUSPEND){
				pauseDevice(handle); 
				...
			} else {
				logger.log("Device suspend.");
			}
		} else {
			logger.log("Invalid handle for: ");
		}
	}
}
```

Good👍 : 오류가 발생하면 예외를 던지는 편이 낫다. 그러면 호출자 코드가 더 깔끔해진다. 

```java
public class DeviceController{
	...
	public void sendShutDown(){
		try{
			tryToShutDown();
		} catch (DeviceShutDownError e){
			logger.log(e);
		}
	}
	
	private void tryToShutDown() throws DeviceShutDownError {
		DeviceHandle handle = getHandle(DEV1);
		DeviceHandle recrod = retrieveDeviceRecord(handle); 
		
		pauseDevice(handle);
		...
	}
	
	private DeviceHandle getHandle(DeviceID id){
		...
			throw new DeviceShutDownError("Invalid handle for: ");
		...
	}
}
```

## Try-Catch-Finally 문 부터 작성하라

try-catch-finally 문으로 시작하면 try 블록에서 무슨일이 생기는지 호출자가 기대하는 상태를 정의하기 쉬워진다. 

try문은 transaction처럼 동작하는 실행코드로, catch문은 try문에 관계없이 프로그램을 일관적인 상태로 유지하도록 한다. 

### Step 1

```java
// Step 1: StorageException을 던지지 않으므로 이 테스트는 실패한다.

@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
  sectionStore.retrieveSection("invalid - file");
}

public List<RecordedGrip> retrieveSection(String sectionName) {
  // dummy return until we have a real implementation
  return new ArrayList<RecordedGrip>();
}
```

### Step 2

```java
// Step 2: 이제 테스트는 통과한다.
public List<RecordedGrip> retrieveSection(String sectionName){
  try{
    FileInputStream stream = new FileInputStream(sectionName);
  } catch (Exception e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

### Step 3

```java
// Step 3: Exception의 범위를 FileNotFoundException으로 줄여 정확히 어떤 Exception이 발생한지 체크하자.
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

## 미확인(unchecked) 예외를 사용하라

- 예전에는 메서드를 선언할 때는 메서드가 반환할 예외를 모두 열거 했다.
- 예외 처리에 드는 비용 대비 이득을 생각해봐야 한다.
- 확인된 예외를 사용하면 **OCP(Open Closed Principle)를 위반한다.**
    - 확인된 예외를 던졌는데 catch 블록이 세 단계 위에 있다면 그 사이 메서드 모두가 선언부에 해당 예외를 정의해야 한다.
    - 다시 빌드한 다음 배포해야 한다.
    - 상위 레벨 메소드에서 하위 레벨 메소드의 디테일에 대해 알아야 하기 때문에 캠슐화 또한 깨진다.

```java
public void get(){
	try{
		
	} catch (InvalidGetException e){
		logger.log(e);
	}
}

public void getById() throws InvalidGetException {
	call();
}

public void call() throws InvalidGetException {
	throw new InvalidGetException(); 
}
```

## 예외에 의미를 제공하라

호출 스택만으로 사용자가 의도를 파악하기 어려우므로 오류 메시지에 정보를 담아 예외와 함께 던져야 한다. 전후 상황을 충분히 덧붙이고 실패한 연산 이름과 실패 유형도 언급한다. 

## 호출자를 고려해 예외 클래스를 정의해라

프로그래머는 오류를 정의할 때 **오류를 잡아내는 방법**을 고려해야한다. 

외부 API의 다양한 예외를 직접 노출하지 않고 감싸기 기법을 통해 새로운 클래스를 만들어 캡슐화.

Bad👎 : 다른 종류의 예외를 처리 로직은 모두 같으므로 의미가 없음

```java
// 외부 라이브러리가 던질 예외를 모두 잡아 낸다. 
// 같은 에러 잡아 내는 코드가 많다. 
// 변경하기 어렵다.

ACMEPort port = new ACMEPort(12);
try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device response exception");
} finally {
  ...
}
```

Good👍 : 

```java
// 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환.
// Wrapper클래스 덕분에 의존성이 크게 감소.

LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}

public class LocalPort {
  private ACMEPort innerPort;
  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }

  public void open() {
    try {
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  ...
}
```

## 정상 흐름을 정의하라.

외부 API를 감싸 독자적인 예외를 던져 중단한 뒤 호출하는 코드에서 

처리를 정의해 중단된 계산을 처리하는 것은 대게 적합하지만 중단이 적합하지 않은 경우도 있다. 

이러한 경우 특수 사례 패턴(SPECIAL CASE PATTERN)을 적용해 개선한다. 

특수 사례 패턴이란?

반환할 값이 없을 때 예외를 던지는 것이 아니라 기본 값을 반환

Bad👎

```java
// 식비비용 조회를 실패하면 일일 기본 식비를 총계에 더한다.
// 특수 상황을 처리할 필요가 없다면 더 코드는 간결해진다.
try { 
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal(); 
} catch(MealExpensesNotFound e) {
   m_total += getMealPerDiem(); 
}
```

Good👍

```java
// caller logic.
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

public class PerDiemMealExpenses implements MealExpenses {
	private static final int MEAL_EXPENSES_DEFAULT = 3000;
  public int getTotal() {
    // 기본값으로 일일 기본 식비를 반환
		return MEAL_EXPENSES_DEFAULT;
  }
}
```

## null을 반환하지 마라

메서드가 null을 반환하면 해당 메서드를 사용하는 클라이언트는 null 체크 코드를 추가할 수 밖에 없다. 이것은 좋지 못한 습관이다. 이 과정에서 null 체크가 누락되면 버그로 이어지고, null 체크 코드는 아름답지 못하다. 특수 사례 객체를 통해 이러한 문제를 해결할 수 있다. 

Bad👎

```java
public void registerItem(Item item) { 
  if (item != null) {
    ItemRegistry registry = peristentStore.getItemRegistry();
    if (registry != null) {
      Item existing = registry.getItem(item.getID());
      if (existing.getBillingPeriod().hasRetailOwner()) {
        existing.register(item);
      }
    }
  }
}

List<Employee> employees = getEmployees();
  if (employees != null) {
    for(Employee e : employees) {
      totalPay += e.getPay();
    }
  }
```

Good👍

```java
// 더 읽기 쉬운 코드.
  List<Employee> employees = getEmployees();
  for(Employee e : employees) {
    totalPay += e.getPay();
  }

  public List<Employee> getEmployees() {
    if( .. there are no employees .. )
      return Collections.emptyList();
    }
  }
```

## null을 전환하지 마라

메서드의 argument로 null을 전달하게 되면 메서드는 내부에서 null 체크 또는 assert를 이용하여 처리하는 로직을 추가할 수 밖에 없다. 이러한 로직을 통해 exception을 던지는 메서드가 되면 해당 메서드를 사용하는 쪽에서도 exception 처리를 추가해야 한다. 

호출자가 넘기는 null을 처리하는 방법은 없다. 정책적으로 null을 넘기지 못하게 하는 것이 가장 합리적이다. 

Bad👎

```java
// Parameter에 null값이 들어오면 NullPointerException 발생.
public class MetricsCalculator { 
  public double xProjection(Point p1, Point p2) { 
    return (p2.x – p1.x) * 1.5; 
  } 
}
```

Better

```java
public class MetricsCalculator { 
  public double xProjection(Point p1, Point p2) { 
    if(p1 == null || p2 == null){
      throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection"); 
    } 
    return (p2.x – p1.x) * 1.5; 
  } 
}
```

Good👍

```java
public class MetricsCalculator { 
  public double xProjection(Point p1, Point p2) { 
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    return (p2.x – p1.x) * 1.5; 
  } 
}
```