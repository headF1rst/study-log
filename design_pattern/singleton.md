싱글톤 패턴이란 인스턴스를 오직 한개만 제공하는 클래스를 말한다.

`Settings` 라는 클래스를 얼마든지 마음대로 생성할 수 있다.

![스크린샷 2022-08-08 오후 1.44.20.png](https://i.imgur.com/LzyhtVv.png)

이렇게 하나의 클래스로 부터 새롭게 생성된 인스턴스들은 서로 같지 않다.

- `new` 를 사용해서 인스턴스를 만들게끔 허용하면 싱글톤 패턴을 만족시킬수 없다.
    - `private` 생성자를 사용해서 객체 외부에서 new 키워드로 생성자를 호출 할 수 없게 해야한다.
    - 외부에서 인스턴스를 만들 수 없기 때문에 클래스 안에서 인스턴스를 만들어주는 방법을 `global` 하게 제공해 줘야 한다. - getInstance() 메서드를 `static` 으로 선언.

      ![스크린샷 2022-08-08 오후 1.46.49.png](https://i.imgur.com/WViQ6pt.png)




## 멀티 쓰레드 환경에서 안전하지 않은 싱글톤 코드

![스크린샷 2022-08-08 오후 2.13.19.png](https://i.imgur.com/p6hntr1.png)

위 코드가 `thread-safe` 하지 않은 이유는 다음과 같다.

- 두개의 각기 다른 쓰레드 A, B가 있다.
- 쓰레드 A가 `getInstance()` 메서드의 조건문을 통과한 순간, 컨텍스트 스위치가 발생한다.
- 쓰레드 B가 `getInstance()` 메서드의 조건문을 통과하고 Settings 인스턴스를 생성한다.
- 쓰레드 A가 다시 실행되며 쓰레드 A가 Settings 인스턴스를 생성한다.
- 쓰레드 A와 B는 서로 다른 인스턴스를 갖게된다.

## 멀티 쓰레드 환경에서 Thread-safe 하게 싱글톤 구현하기

### 1. 동기화 사용 - Synchronized 키워드 사용하기

![스크린샷 2022-08-08 오후 2.23.06.png](https://i.imgur.com/hI2WDWb.png)

`getInstance()` 메서드에 한번에 딱 하나의 쓰레드만 들어올 수 있게 `synchronized` 키워드를 적용한다.

단,

- 동기화 처리 작업때문에 synchronized 메서드를 실행할 때마다 성능상의 불이익이 생길 수 있다.

🤔

1. [**자바의 동기화 블럭 처리 방법은?**](https://parkcheolu.tistory.com/15)

1. **getInstance() 메서드 동기화시 사용하는 lock은 인스턴스의 lock인가 클래스의 lock인가?**

   getInstance() 메서드가 static 메서드이기 때문에 `클래스의 lock` 이다.

   이는 스태틱 메서드 동기화인데, 이 메서드를 가진 클래스의 클래스 객체를 기준으로 lock이 이루어진다.

   JVM 안에 클래스 객체는 클래스 당 하나만 존재할 수 있으므로, 같은 클래스에 대해서는 오직 한 쓰레드만 동기화된 스태택 메서드를 실행할 수 있다.


### 2. 이른 초기화 (eager initialization)

![스크린샷 2022-08-08 오후 2.34.16.png](https://i.imgur.com/8HurYem.png)

Settings 클래스가 로딩되는 시점에 Settings 인스턴스가 초기화 된다.

`getInstance()` 메서드는 클래스 로딩 시점에 이미 생성된 하나의 동일한 인스턴스를 리턴해 주기만 한다.

단,

- 인스턴스를 생성하는데 시간이 오래걸리고, 메모리를 많이 사용하게 된다면, 애플리케이션 로딩 시점에 많은 리소스를 사용해서 인스턴스를 생성했음에도 불구하고 객체를 사용하지 않는다면, 리소스 낭비이다.
    - `getInstance()` 메서드 호출 시점에 객체를 생성하는게 가장 이상적.


### 3. Double checked locking으로 효율적인 동기화 블럭 만들기

인스턴스에 대한 체크를 두번하기 때문에 `double check locking` 이라고 부른다.

![스크린샷 2022-08-08 오후 3.29.26.png](https://i.imgur.com/BMilpl8.png)

이때, `getInstance()` 메서드 동기화시 사용하는 락은 클래스의 락이다

- 두개의 각기 다른 쓰레드 A, B가 있다.
- 쓰레드 A가 `getInstance()` 메서드의 if 문을 확인하고 synchronized 블록에 들어온 다음, 컨텍스트 스위치가 발생한다.
- 쓰레드 B가 `getInstance()` 메서드의 if 문을 확인하고 synchronized 블록에 들어가려 하지만, 이미 쓰레드 A가 lock을 걸어 놨기 때문에 진입하지 못한다.
- 쓰레드 A가 인스턴스를 생성하고 리턴문을 실행한다.
- 쓰레드 B는 synchronized 블록에 lock을 걸고 들어가고 if 문을 확인해서 인스턴스를 생성하지 않고 나온다.

### 인스턴스 레벨의 락보다 double checked locking이 효율적인 이유

- 1번의 인스턴스 레벨의 락은 `getInstance()` 메서드를 호출할 때 마다 매번 `synchronized` 가 걸리지만 **double checked locking** 은 그렇지 않다.
    - 첫번째 조건문에서 인스턴스가 이미 있는 경우엔 synchronized 블록을 전혀 사용하지 않게 된다.
- 인스턴스를 필요한 시점에 생성할 수 있다.

### 4. static inner 클래스 사용 - 권장 ⭐️

![스크린샷 2022-08-08 오후 3.50.16.png](https://i.imgur.com/JqzlwtV.png)

멀티 쓰레드 환경에서도 thread-safe 하다.

`getInstance()` 메서드가 호출될 때, `SettingsHolder` 클래스가 로딩이 되기 때문에, **lazy initialization** 이라고 볼 수 있다.

다시말해, `static class` 는 static 이기 때문에 프로그램 실행 시점에 class loader에 의해서 jvm에 로드 되지만 아직 초기화 되어있지는 않다.

해당 static class를 호출하거나 접근할때서야 비로소 초기화가 된다.

때문에 `SettingsHolder` 클래스도 static이지만 `getInstance()` 메서드가 호출되면서 클래스에 접근 할때 서야 비로소 초기화가 되기 때문에 lazy initialization이다.

[[Java] static inner class 는 언제 로드가 될까? 로드와 초기화?](https://kdhyo98.tistory.com/70)

위처럼 싱글톤을 위한 코드를 작성하더라고, `reflection` 과 `serializable` 을 사용하면 싱글톤 패턴을 깨트리고 사용할 수 있다.

사용자 측에서 싱글톤을 깨트리지 못하게 하고 작성자의 의도대로 객체를 생성하게 하고 싶은경우에는 다음과 같은 방법을 사용한다.

### 5. enum을 사용하는 방법 - 권장 ⭐️

enum은 리플렉션에서 new 인스턴스를 할 수 없게 막아놨기 때문에 사용자가 설계자의 의도를 어기고 싱글톤을 깨트리지 못한다.

enum에 대한 인스턴스를 리플랙션으로 생성할 수 없다.

enum에는 이미 직렬화가 상속되어 있다.

enum의 serializable 같은 경우에는 별다른 장치를 추가하지 않더라도 안전하게 동일한 인스턴스로 역직렬화가 가능하다.

단, 단점으로는

- enum은 클래스를 로딩하는 순간, 클래스가 생성된다. (이른 초기화)
- 상속을 받지 못한다.
    - 만들고자 하는 싱글톤 클래스가 특정한 클래스를 상속 받아서 만들어야 한다면 static inner 클래스를 사용하는 방법 밖에 없다.

## 정리

### Q1. 자바에서 enum을 사용하지 않고 싱글톤 패턴을 구현하는 방법은?

### Q2. private 생성자와 static 메서드를 사용하는 방법의 단점은?

### Q3. enum을 사용해 싱글톤 패턴을 구현하는 방법의 장점과 단점은?

### Q4. static inner 클래스를 사용해 싱글톤 패턴을 구현하라.
