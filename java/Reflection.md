# 자바 reflection

*리플랙션 - 거울에 반사된 이미지. 실체가 아닌 거울에 비쳐진 나의 모습을 보고 모습을 확인하거나 옷매무세를 수정하는등*

프로그래머가 작성한 모든 클래스는 JVM의 클래스 로더가 클래스 정보를 읽어와서 해당 정보를 메모리에 저장해 놓는다. (클래스 정보 - 각 클래스가 어떠한 생성자, 메서드, 필드들을 포함하고 있는지 나타낸다)

이때 클래스 로더가 읽어온 클래스 정보가 각각의 클래스의 리플랙션(거울에 비친 내 모습) 이라고 볼 수 있다.

리플랙션을 사용하여 “특정한 어노테이션이 붙은 클래스인가?” 등을 확인 할 수 있다.

혹은 어노테이션에 추가로 제공된 정보를 바탕으로 추가적인 일을 처리하는 것 또한 가능하다.

자바 Reflection은 컴파일 시간이 아닌 런타임에 동적으로 특정 클래스의 정보를 추출할 수 있는 프로그래밍 기법이다.

리플렉션은 구체적인 클래스 타입을 알지 못하더라도 (접근 제어자가 private 이더라도) 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 Java API 이다.

## 대표적으로 언제 사용할까?

- 동적으로 클래스를 사용해야할 때 사용.
- 프로그램 작성 시점에는 어떤 클래스를 사용할지 모르지만 런타임 시점에서 클래스를 가져와서 실행해야 하는 경우.
- ex) Spring의 어노테이션 같은 기능들이 리플렉션을 이용하여 런타임에 동적으로 클래스의 정보를 가져와서 사용한다.

## 리플렉션을 사용하여 가져올 수 있는 클래스 정보

다음과 같은 클래스 정보들을 리플렉션을 사용하여 가져올 수 있다.

정보들을 가져와서 객체를 생성하거나 메서드를 호출하거나 변수의 값을 변경할 수 있다.

- Class
- Constructor
- Method
- Field

## Java Reflection 주요 API

### java.lang.Class

- `String getName()` - 패키지 + 클래스 이름을 반환.
    
    ```java
    public class Test {
    		Class<?> aClass = Class.forName("me.sanha.helloService");
    		System.out.println(aClass.getName()); // me.sanha.helloService
    ```
    

- `int getModifiers()` - 클래스의 접근 제어자를 숫자로 반환.
- `Field[] getFields()` - 접근 가능한 **public** 필드 목록을 반환.
- `Field[] getDeclaredFields()` - 모든 필드 목록을 반환.
- `Constructor[] getConstructors()` - 접근 가능한 public 생성자 목록을 반환.
- `Constructor[] getDeclaredConstructors()` - 모든 생성자 목록을 반환.
- `Method[] getMethods()` - 부모 클래스, 자신 클래스의 접근 가능한 public 메서드 목록을 반환.
- `Method[] getDeclaredMethods()` - 모든 메서드 목록을 반환

### java.lang.reflect.Constructor

- `String getName()` - 생성자 이름을 반환
- `int getModifiers()` - 생성자의 접근 제어자를 숫자로 반환.
- `Class[] getParameterTypes()` - 생성자 파라미터의 데이터 타입을 반환.

### java.lang.reflect.Field

- `String getName()` - 필드 이름을 반환.
- `int getModifiers()` - 필드의 접근 제어자를 숫자로 반환.

### java.lang.reflect.Method

- `String getName()` - 메서드 이름을 반환
- `int getModifiers()` - 메서드의 접근 제어자를 숫자로 반환.
- `Class[] getParameterTypes()` - 메서드 파라미터의 데이터 타입을 반환.

---

## 클래스 객체를 받아오는 3가지 방법

```java
public class ReflectionTest {
	
  private String name;
  private String gender;
  private Tool use;

  public enum Tool{
    INTELLIJ, ECLIPSE
  }
  
  public ReflectionTest() { 
  
  }
  
  private ReflectionTest(Tool use) { //private
    setUse(use);
    System.out.println("Tool : " + getUse());
  }
  
  private void setNameAndUse(String name, Tool use) { //private
    setName(name);
    setUse(use);
  }
  
  public Tool getUse() {
    return use;
  }

  public void setUse(Tool use) {
    this.use = use;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getGender() {
    return gender;
  }

  public void setGender(String gender) {
    this.gender = gender;
  }
}
```

`ReflectionTest` 객체를 가져오는 방법에는 3가지 방법이 있다.

1. Class clazz = ReflectionTest.class;
    
    `.class` 를 붙이는 방법으로 해당 클래스를 바로 참조한다.
    
2. Class clazz = Class.forName(”com.reflection”);
    
    `forName()` 메서드를 이용하여 괄호 안에 기술된 클래스를 로드하는 방법. 
    
    동적인 코드를 작성하는데 더 나은 장점을 갖고있다.
    
3. ReflectionTest reflectionTest = new ReflectionTest();
    
    Class clazz = reflectionTest.getClass();
    
    객체 생성 후, `getClass()` 메서드를 이용하는 방법.
    

## 객체 생성을 위한 생성자 정보 가져오기

생성자의 정보를 가져오는 메서드는 크게 2가지가 있다.

- `getConstructors()`
    
    해당 클래스에 **public** 으로 선언된 모든 생성자를 배열의 형태로 반환.
    
- `getDeclaredConstructors()`
    
    해당 클래스에 선언된 모든 생성자를 배열의 형태로 반환.
    
    (**private** 으로 선언된 생성자도 포함)
    

## 메서드 정보 가져오기

- `getMethods()`
    
    **public** 으로 선언된 모든 메서드 목록과 해당 클래스에서 상속받은 메서드에 접근.
    
- `getDeclaredMethods()`
    
    해당 클래스에 선언된 메서드 정보 반환. (**private** 으로 선언된 메서드도 포함)
    

## 변수 정보 가져오기

- `getFields()`
    
    **public** 으로 선언된 변수 목록을 Field 타입의 배열로 반환.
    
- `getDeclaredFields()`
    
    해당 클래스에서 정의 된 모든 변수 목록을 배열의 형태로 반환합니다.
    

## 생성자를 이용한 객체 생성

필요한 생성자 하나에 접근하여 정보를 추출하는 방법에는 3가지가 있다.

- `getConstructor(parameterTypes)`
    
    public으로 선언된 생성자에 접근 (단일 매게 변수 필요)
    
- `getConstructor(new Class[] {parameterTypes, parameterTypes...})`
    
    public 으로 선언된 생성자에 접근 (복합 매게 변수 필요)
    
- `getDeclaredConstructor(parameterTypes)`
    
    모든 생성자에 접근 (단일 매게 변수 필요)
    
    단, private 생성자에 접근하기 위해서는 `setAccessible(true)` 가 필요.
    

```java
public class ReflectionTest {
		
		private ReflectionTest(Tool use) {
			setUse(use);
		}
		...
}
```

위의 ReflectionTest 클래스에 선언된 use를 매게 변수로 하는 private 생성자 정보를 가져와 보도록 하겠다.

```java
package com.reflection;

public class GetReflectionTest {
	public static void main(String[] args) {
			
			try {
				Class clazz = Class.forName("com.reflection");
			} catch (..) {
				...
			}
	
			try {
				Constructor constructor = clazz.getDeclaredConstructor(Tool.class);
				constructor.setAccessible(true);
				constructor.newInstance(Tool.ECLIPSE);
			} catch (...) {
				...
			}
	}
}
```

생성자의 매게 변수 타입이 `Tool` enum 타입이기 때문에 `getDeclaredConstructor()` 메서드 안에 Tool.class를 전달해 주었다.

`getDeclaredConstructor()` 는 private 생성자에 접근할 수 있지만 `setAccessible(true)` 메서드를 추가로 정의해 주어야한다.

그 다음, `newInstance(init args)` 메서드를 사용하여 **객체를 생성**해 주었다. 

## 메서드를 동적으로 실행하는 방법

특정 메서드 정보를 불러와서 동적으로 실행시키기 위해서는 `invoke(obj, args)` 함수를 사용한다.

invoke 메서드의 첫번째 인자인 obj는 해당 메서드를 실행하기 위해 필요한 객체,

두번째 인자인 args는 해당 메서드에 필요한 매게 변수이다.

![스크린샷 2022-08-20 오후 8.28.59.png](https://i.imgur.com/hxwXMSb.png)

## 변수의 값을 동적으로 제어하는 방법

![스크린샷 2022-08-20 오후 8.37.44.png](https://i.imgur.com/2W9SM3Q.png)

`Student` 객체 정보를 불러온 다음 인스턴스를 생성해 주었다.

그다음 “name” 이라는 필드 정보를 불러온 다음 수정을 위해서 `setAccessible(true)` 로 설정하여 private인 name 필드값에 접근할 수 있도록 설정하였다.

`set(obj, args)` 메서드를 사용하여 수정하고자 하는 필드값의 인스턴스와 새로 설정할 값을 인자로 넘겨주어 필드 값을 동적으로 변경하였다.