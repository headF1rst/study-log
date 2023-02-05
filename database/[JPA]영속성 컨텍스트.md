영속성 컨텍스트 : 논리적으로 엔티티를 영구 저장하는 환경. **EntityManager.persist(entity);**

- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근
- persist(entity) → entity를 영속성 컨텍스트에 저장.

## 엔티티의 생명주기

![IMG_BED44C4043A4-1.jpeg](https://blog.kakaocdn.net/dn/XPxaN/btqYJ9Z3PMB/svGLPEkkDrNFfvLUd3bwe1/img.png)

<small>출처 - https://huisam.tistory.com/entry/persistContext</small>

### 비영속(new / transient)

방금 막 객체를 생성한 상태. 영속성 컨텍스트와 전혀 관계 없다.

```java
Owner owner = new Owner(); // 객체가 생성된 상태
owner.setId = ("owner1");
owner.setName = ("관리자1");
```

### 영속(managed)

영속성 컨텍스트에 관리되는 상태 

em.persist(entity);

```java
EntityManager em = emf.createEntityManager(); // 엔티티 펙토리가 각 요청마다 엔티티 매니저 생성
em.getTransaction().begin();

em.persist(owner); // owner 객체가 영속 상태가 된다.
```

### 준영속(detached)

영속성 컨텍스트에 저장되었다가 분리된 상태

```java
em.detach(owner); // owner 객체를 영속성 컨텍스트에서 분리. 준영속 상태
```

### 삭제(removed)

삭제된 상태

```java
em.remove(owner); // owner 객체를 삭제한 상태
```

## 영속성 컨텍스트 사용 이유

영속성 컨텍스트는 애플리케이션과 DB 중간에 존재한다.

이로인해 얻을수 있는 이점은 다음과 같다.

- 1차 캐시
- 동일성 (identity) 보장
- 트랜잭션을 지원하는 쓰기 지연
- 변경 감지 (Dirty checking)
- 지연 로딩 (Lazy loading)

### 1. 1차 캐시에서 엔티티 조회

객체는 영속상태가 되면서 객체의 식별값 (Id)을 key로 하여 key - value 형태로 1차 캐시에 저장된다.

엔티티 메니저를 통해, 식별자 값이 “owner1”인 객체를 조회하는 요청을 보내면, 바로 DB에서 값을 조회하는게 아니라 1차 캐시를 먼저 조회한다.

1차 캐시에 요청한 식별자 값이 존재하면 1차 캐시에서 데이터를 가져오기 때문에 `select` 쿼리가 발생하지 않는다.

```java
Owner owner = new Owner();
owner.setId("owner1");

em.persist(owner); // 1차 캐시에 저장됨
Owner findOwner = em.find(Owner.class, "owner1"); // DB가 아닌 1차 캐시에서 조회
```

![IMG_89F166A44F6F-1.jpeg](https://lar542.github.io/img/post_img/JPA-2019-08-01-1.png)

만약 1차 캐시에 존재하지 않는값 “owner2”를 조회하게 된다면, DB를 조회해서 데이터를 1차캐시에 key - value 형태로 저장한 다음, 데이터를 가져온다.

때문에 em.persist(); 뿐만 아니라 데이터를 조회하는 em.find()를 통해서도 객체를 영속 상태로 만든다.

이때, 유의할 점은 엔티티메니저는 DB 트랜젝션 단위로 생성, 삭제 되기 떄문에 여러명이 1차 캐시를 공유하지는 않는다.

### 2. 영속 엔티티의 동일성 보장

동일한 트랜젝션 내에서 동일한 식별자 값으로 두 객체를 조회하였을때, 같은 객체임을 보장한다.

```java
Owner a = em.find(Owner.class, "owner1");
Owner b = em.find(Owner.class, "owner1");

System.out.println(a == b); // 동일성 비교 결과 : true
```

### 3. 트랜잭션을 지원하는 쓰기 지연

em.persist(entity);를 통해 엔티티를 영속성 컨텍스트 안에 저장만 해놨다가, 트랜잭션 커밋 시점에 DB에 INSERT 쿼리를 날려서 영속성 컨텍스트 내의 엔티티들을 한번에 DB에 반영(flush)시킨다.

![IMG_AEA94296A4B1-1.jpeg](https://hongchangsub.com/content/images/2021/10/Screen-Shot-2021-10-21-at-12.01.08-AM.png)

### 4. 변경 감지

객체가 처음 조회되어 영속성 컨텍스트의 1차 캐시에 저장될 때, JPA는 초기 객체의 상태인 **스냅샷을 같이 저장해 놓는다.**

객체의 데이터를 수정하고 나서, 트랜잭션이 커밋 되는 시점에, JPA는 1차 캐시의 엔티티값과 스냅샷을 비교하는데 만약 둘이 다르면, Update 쿼리를 생성해서 쓰기 지연 저장소에 저장해 놓는다.

그다음 쓰기 지연 저장소에 지연 쓰기 되어있던 쿼리들과 함께 한번에 DB에 flush 되어 변경사항이 DB에 반영되게 된다.

참고) 조회 (select) 쿼리는 트랜잭션 커밋 전에 바로 DB에 뿌려진다.

![IMG_C41E262A4D5D-1.jpeg](https://hongchangsub.com/content/images/2021/10/Screen-Shot-2021-10-22-at-11.49.15-PM.png)

### flush 란?

영속성 컨텍스트의 변경내용을 DB에 반영.

flush가 발생 → 변경 감지 발생 → 수정된 엔티티 쓰기 지연 저장소에 등록 → 쓰기 지연 저장소의 모든 쿼리를 DB에 전송

flush가 발생하더라도 영속성 컨텍스트는 비워지지 않는다. (1차 캐시의 내용 유지)

flush 방법

- em.flush() - 직접 호출
- transaction.commit() - 자동 호출
- JPQL 쿼리 실행 - 자동 호출

## 준영속 상태로 만드는 방법

영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태.

JPA가 관리하지 않으므로 트랜젝션 커밋 되도 아무 일이 일어나지 않는다.

- em.detach(entity) - 특정 엔티티만 준영속 상태로 전환.
- em.clear() - 영속성 컨텍스트 완전히 초기화
- em.close() - 영속성 컨텍스트 종료
