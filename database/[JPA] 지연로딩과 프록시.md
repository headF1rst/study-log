Member와 Team 엔티티가 다대일 연관관계로 맵핑되어있다고 생각해 보겠습니다.

이때 회원 정보를 조회하면서 회원의 이름만 필요한 경우가 있을테고 회원 이름과 해당 회원이 속한 팀 정보 모두 필요한 경우가 있을것 입니다.

- Case1. 회원과 팀 함께 조회
- Case2. 회원만 조회

```java
public void printUserInfo(String memberId) {
	
	Member member = em.find(Member.class, memberId);
	Team team = member.getTeam();

	// Case1. 회원과 팀 함께 출력
	System.out.println("회원 이름: " + member.getName());
	System.out.println("소속팀: " + team.getName());

	// Case2. 회원만 출력
	System.out.println("회원 이름: " + member.getName());

}
```

두 케이스 모두 `회원 엔티티`를 DB로 부터 가져와서 영속성 컨텍스트에 올리게 됩니다.

이때 회원 엔티티와 팀 엔티티가 맵핑 되어있기 때문에 해당 회원과 연관된 팀 엔티티도 같이 DB로 부터 조회되어서 영속성 컨텍스트에 올라옵니다. 

만약 회원의 이름만 조회하는 경우에는 불필요하게 팀을 가져오는 쿼리가 발생하는 문제가 발생합니다.

때문에 우리는 `지연 로딩` 을 사용하여 회원 엔티티를 조회하면서 가짜$($프록시) 팀 객체를 생성하여 DB 조회를 미루다가 팀 정보가 필요해지면 실제 엔티티를 조회하는 작업을 할 수 있습니다.

## 프록시 객체 생성

JPA를 사용해서 객체를 조회할 때 우리는 `em.find()` 를 사용하였습니다.

진짜 객체가 아닌 프록시 객체를 조회하기 위해서는 `em.getReference()` 를 사용합니다.

 `em.getReference()` - 데이터베이스 조회를 미루는 가짜$($프록시) 엔티티 객체 생성

### 프록시의 특징

- 프록시 객체는 실제 클래스를 상속 받아서 만들어집니다.
    - 타입 체크시 `==` 대신 `instance of` 사용.
- 프록시 객체는 실제 객체의 참조$($target)를 보관합니다.
- 프록시 객체는 처음 사용할 때 한 번만 초기화됩니다.
- DB에 쿼리가 나가지 않아도 객체를 조회하는게 가능합니다.
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메서드를 호출합니다.
    - 프록시 객체가 초기화 되면, 프록시 객체가 실제 엔티티로 바뀌는 것이 아니라 프록시 객체를 통해서 실제 엔티티에 접근이 가능해 지는것.
- 영속성 컨텍스트에 찾는 엔티티가 있으면 `em.getReference()` 호출해도 실제 엔티티를 반환합니다.
- 영속성 컨텍스트의 도움으 받을 수 없는 준영속 상태일 때 프록시를 초기화 하면 LazyInitializationException 발생

```java
Member findMember = em.getReference(Member.class, member.getId());
System.out.println("findMember.getId() = " + findMember.getId());

// DB에 쿼리 날려서 실제 객체의 메서드 호출 .getName()
System.out.println("findMember.getName() = " + findMember.getName());
```

회원 이름은 DB에 접근해야만 얻을수 있는 값이기 때문에 getName 메서드를 호출할 때 DB에 쿼리가 나가게 됩니다.

![Untitled](https://i.imgur.com/PohfO4v.png)

## 지연 로딩 - FetchType.LAZY

앞서 설명한 프록시 객체를 만들어서 실제 값이 필요할 때 까지 DB 조회를 미루는 것을 `지연 로딩` 이라고 합니다.

만약 회원 엔티티만 조회해서 회원 정보만 쓰는게 대부분이라면, 팀 엔티티는 지연 로딩을 사용하여 프록시 객체로 갖고 팀 정보가 필요할 때 프록시에서 실제 객체의 메서드를 호출하며 이때 쿼리가 나가게 됩니다.

![Untitled](https://i.imgur.com/jmDka3O.png)

회원 엔티티를 조회할 시에는 회원 엔티티와 연관관계인 팀 엔티티는 프록시 객체로 조회해옵니다.

![스크린샷 2022-05-12 오후 1.18.43.png](https://i.imgur.com/p2MKTum.png)

팀 엔티티에 대한 정보가 필요하게 되면 프록시 객체를 통해서 실제 team 엔티티에 접근하며 DB로 부터 정보를 조회하는 쿼리가 발생합니다.

![스크린샷 2022-05-12 오후 1.21.24.png](https://i.imgur.com/lEU7dmN.png)

## 즉시 로딩 - FetchType.EAGER

만약 회원 엔티티와 팀 엔티티를 자주 함께 사용할 경우에는 즉시 로딩을 사용하는것이 효율적입니다.

![Untitled](https://i.imgur.com/NqMzEb6.png)

즉시 로딩을 통해서 회원 엔티티를 조회할때 해당 회원과 연관된 팀 엔티티를 함께 조회하게 됩니다.

![스크린샷 2022-05-12 오후 1.34.50.png](https://i.imgur.com/LD1WcZ8.png)

연관관계 맵핑에서 `xxxToOne` 는 기본이 `즉시로딩` , `xxxToMany` 는 기본이 `지연로딩` 으로 설정되어있습니다.

**참고 자료**

- 자바 ORM 표준 JPA 프로그래밍 - 김영한 저자