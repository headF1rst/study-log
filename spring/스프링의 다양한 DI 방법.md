**DI $($의존관계 주입) :** 애플리케이션 실행 시점에 외부에서 구현 객체를 생성하고, 그 참조값을 클라이언트로 전달해서 클라이언트와 서버의 의존관계가 연결 되는 것.

- 클라이언트 코드를 수정하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다. $($OCP 😊)

- DI를 통해서 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다.

## 정적인 클래스 의존관계
애플리케이션을 실행하지 않아도 의존관계를 분석할 수 있다. $($클래스 다이어그램으로 객체간의 의존관계를 표현)

단, 클래스 다이어그램을 통해서는 실행시에 실제로 어떤 객체가 주입 될지 알 수 없다.

## 동적인 객체 인스턴스 의존 관계
애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계. $($객체 다이어그램으로 표현)

## 다양한 의존관계 주입 방법

- 필드 주입
- 수정자 주입
- 생성자 주입 ⭐
    - lombok 라이브러리

### 필드 주입
테스트 코드에서만 사용된다.

`@Autowired`를 필드 값에 붙여준다.
필드에 바로 의존관계를 주입하는 방법.

외부에서 변경이 불가능하기 때문에 테스트가 어렵다.

```java
@Component
public class OrderServiceImpl implements OrderService {

  @Autowired private MemberRepository memberRepository;
  @Autowired private DiscountPolicy discountPolicy;
}
```

### 수정자 주입
필드 값을 변경하는 수정자 메서드$($setter)를 통해서 의존관계를 주입하는 방법.

- 선택, 변경 가능성이 있는 의존관계에 사용

단, set 메서드를 public으로 열어둬야 하기 때문에 필드값이 변경될 위험이 있다.

set 메서드로 인해 필드 값에 `final`을 선언해 주지 못한다.

때문에 생성자에 값이 설정되지 않은 오류를 컴파일 시점에 알 수 없다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;
    
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    
    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
```

### 생성자 주입
생성자를 통해서 의존관계를 주입 받는 방법.

생성자 위에 `@Autowired` 에너테이션을 추가해준다.

- 생성자 호출 시점에 딱 1번만 호출되는것이 보장된다.
- **불변, 필수** 의존관계에 사용.

주로 의존관계는 애플리케이션 종료 전까지 변하면 안된다.

생성자에 값이 설정되지 않는 요류를 컴파일 시점에 알기 위해서 팔드 값을 `final`로 선언한다.

```java
@Component
public class OrderServiceImpl implements OrderService {
	
		// 필드값을 final로 선언하여 값이 반드시 들어가는걸 보장해 주었다.
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

위의 예제와 같이 생성자가 1개 있으면 `@Autowired`를 생략할 수 있다.

`@Autowired`는 스프링 컨테이너에서 스프링 빈을 타입으로 조회한다.

생성자가 호출될 때, `@Autowired`가 있으면 스프링 컨테이너에서 필드 값과 일치하거나 하위 타입을 조회하여 파라미터에 주입시켜 준다.

번외
`@Component`를 붙인 객체는 `@ComponentScan`을 통해 스프링 컨테이너에 빈으로 등록된다.

#### lombok

`@RequiredArgsConstructor`

롬복 라이브러리가 제공하는 `@RequiredArgsConstructor`을 사용.

final이 붙은 필드를 모아서 생성자를 자동으로 만들어 준다.
$($생성자를 별도로 만들어 주지 않아도 된다.)

```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;
}
```

위의 예제에서 `@RequiredArgsConstructor`는 final이 붙은 memberRepository와 discountPolicy에 값을 주입하는 생성자를 실제로 생성한다.

```java
public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
  this.memberRepository = memberRepository;
  this.discountPolicy = discountPolicy;
}
```
