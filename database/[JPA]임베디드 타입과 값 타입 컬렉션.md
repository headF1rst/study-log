JPA는 데이터를 두 개의 타입으로 분류한다.

하나는 식별자를 갖고, 데이터가 변해도 추적이 가능한  **엔티티 타입**. 
(@Entity로 정의하는 객체)

그리고 변경시 추적이 불가한 자바 기본 타입이나 객체인 **값 타입**이 있다.

이번에 포스트에서 다룰 임베디드 타입과 컬렉션 값 타입 둘 다 값 타입에 속한다.

## 임베디드 타입

임베디드 타입이란 좌표(x, y)를 값으로 정의하고 싶을때와 같이 개발자가 새로운 값 
타입을 직접 정의한 것을 말한다.

주로 x, y 좌표와 같이 기본 값 타입을 모아서 만들며 값 타입이기 때문에 데이터를 
변경해도 추적이 불가하다.

가게 엔티티에 가게 이름, 영업 시작시간, 영업 종료시간, 주소 도시, 주소 번지, 주소 
우편번호를 가진다고 예를 들어보겠다.

이때 영업 시작시간과 영업 종료시간은 “영업 시간"이라는 클래스로, 주소 도시, 주소 
번지, 주소 우편번호는 “가게 주소"라는 클래스로 재정의하는 것이 가능하다.

![store diagram](https://i.imgur.com/JRkAIu3.png)

### 임베디드 타입의 장점

임베디드 타입을 사용하면 테이블은 변경하지 않되, 객체와 테이블을 아주 세밀하게 
매핑하는 것이 가능하다.

재사용이 가능하고 해당 값 타입만 사용하는 의미 있는 메서드를 만드는것도 가능하다.

## 임베디드 타입 사용법

```java
@Entity
public class Store {
	
	...
	@Embedded
	private Address storeAddress;
```

```java
@Embeddable
public class Address {

	private String city;
	private String street;
	private String zipcode;

	public Address() {} // 기본 생성자
```

- `@Embeddable` : 값 타입을 정의하는 곳에 표시
- `@Embedded` : 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수

만약 한 엔티티에서 같은 값 타입을 사용하면 어떻게 해야 할까?

예를 들어 “회사원"이라는 엔티티에는 “집주소"와 “직장주소"가 존재한다.

이때 다음과 같이 작성을 하게 되면 컬럼 명이 중복된다.

```java
@Entity
public class officer {

	@Embedded
	private Address homeAddress;

	@Embedded
	private Address workAddress;
```

이러한 경우, `@AttributeOverrides` , `@AttributeOverride` 를 사용해서 
컬럼 명 속성을 재정의 해주어야 한다.

```java
@Entity
public class officer {

	@Embedded
	private Address homeAddress;

	@Embedded
	@AttributeOverrides({
		@AttributeOverride(name="city", column = 
		@Column(name = "WORK_CITY")),
		@AttributeOverride(name="street", column = 
		@Column(name = "WORK_STREET")),
		@AttributeOverride(name="zipcode", column = 
		@Column(name = "WORK_ZIPCODE")),
	})					
	private Address workAddress;
```

단, 임베디드 타입을 포함한 값 타입을 여러 엔티티에서 공유하면 side effect이 생길 
위험이 있다.

자바의 기본 타입의 경우 값을 대입하면 값을 복사하기 때문에 문제가 없지만 임베디드 
타입과 같은 객체 타입의 경우 참조를 전달하기 때문에 객체 a의 값을 바꾸면 의도와 달리 
객체 b의 값도 바뀌게 된다.

```java
// 기본 타입 
int a = 1;
int b = a; // 기본 타입은 값을 복사
b = 5; // b의 값을 바꿔도 a의 값은 1로 유지

//객체 타입
Address a = new Address("prev");
Address b = a;
b.setZipcode("new"); // a의 속성값도 "new"로 바뀜.
```

이러한 객체의 공유 참조는 피할 수 없기 때문에 애초에 생성 시점 이후 값을 변경할 수 
없는 불변 객체로 설계해야한다.

불변객체는 생성자로만 값을 설정하고 setter를 만들지 않아서 처음에 세팅된 값이 추후에 
변경될 가능성을 차단하면 된다.

(Integer, String은 자바가 제공하는 불변 객체)

## 값 타입의 비교

**동일성 비교** : 인스턴스의 참조 값을 비교. `==` 

**동등성 비교** : 인스턴스의 값을 비교. `equals()`

값 타입은 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야하기 때문에 동등성 
비교를 해야한다.

맥의 경우 cmd + N → Generate equals를 통해 값 타입의 equals() 메서드를 
적절하게 재정의 해서 사용할 수 있다.

## 값 타입 컬렉션

데이터베이스는 컬렉션 타입을 저장할 수 없다.

때문에 엔티티의 컬렉션은 일대다 관계의 별도의 테이블을 생성하여 관리해야 한다.

![collection](https://i.imgur.com/6VuGCew.png)

컬렉션을 데이터 베이스에 저장할 때 식별자 ID를 넣어서 기본키로 사용하면 값 타입이 
아닌 엔티티가 되어버린다.

```java
@Entity
public class Group {
	...
	@ElementCollection
	@CollectionTable(name = "GROUP_MEMBER", joinColumns = 
	@JoinColumn(name = "GROUP_ID"))
	private List<String> groupMember = new ArrayList<>();
```

값 타입 컬렉션의 생명 주기는 컬렉션이 선언된 엔티티에 의존한다.

따라서 값 타입 컬렉션을 별도로 값을 저장하거나 update 할 필요가 없다.

만약 값 타입 컬렉션에 변경 사항이 발생하면 주인 엔티티와 연관된 모든 데이터를 
삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.

체크박스 같이 값이 바뀌어도 괜찮은 케이스에만 값 타입을 사용하고 그 외의 상황에는 값 
타입 컬렉션 보다는 일대다 관계를 위한 엔티티를 만들고 여기에 값 타입을 사용하는 것이 
적합하다.

## 정리
- 값 타입은 변경 불가능하게 설계해야 한다.
- @Setter를 쓰지 말고 생성자에서 값을 모두 초기화해서 변경 불가능한 클래스를 만든다.
- 엔티티, 임베디드 타입은 기본 생성자를 **protected**로 설정.
