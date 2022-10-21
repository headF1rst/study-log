**null** 이라는 개념을 고안했던 영국의 컴퓨터 과학자 Tony Hoare는 당시에 
"존재하지 않는 값"을 표현할 수 있는 가장 편리한 방법을 null 참조라고 생각하였으나 
이는 "10억불 짜리 큰 실수" 였다고 토로하였다.

[Null References: The Billion Dollar Mistake - Tony Hoare](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)

특히, **null** 참조로 인한 NPE$($NullPointerException)는 프로그램 곳곳에서 
컴파일 타임에 숨어있다가 런타임에 문제를 일으켜 자바 개발자들을 잠 못 이루게 하는 
골칫덩어리 같은 녀석이다.

Java8 이전에는 NPE의 위험에 노출된 코드가 있다면 null값인지를 확인해 주는 조건문을 
사용하여 NPE를 예방하였다.

하지만 null 체크를 위한 조건문과 return문이 코드에 도배되면서 핵심 비즈니스 로직을 
파악하기 어려워지고 코드 가독성과 유지 보수성이 현저히 낮아지는 문제가 존재하였다.

## Java8 - Optional의 등장

Java8 부터 도입된 **Optional**을 통해서 우리는 다루기 위험하고 까다로운 null을 `Optional`이라는 안전한 상자에 담아서 처리할 수 있게 되었다.

### Optinal이란?

**Optional** - null이 될 가능성이 있는 객체를 감싸는 일종의 래퍼 클래스.

Optional로 객체를 감싸게 되면 다음과 같은 이점이 있다.

- null 체크를 직접 하지 않아도 된다.
- 명시적으로 해당 변수가 null일 가능성이 있다는 것을 표현할 수 있다. $($불필요한 방어 로직을 줄일 수 있다.)

## Optional 까서 안의 객체 접근하기

Optional이 감싸고 있는 객체에 접근하기 위한 방법에는 다양한 메서드가 존재한다.

- **`get()`**

Optional 객체가 비어있다면 `NoSuchElementException`을 던진다.

- **`orElse(T other)`**

Optional 객체가 비어있다면 넘어온 인자를 반환한다.

- **`orElseGet(Supplier<? extends T> other)`**

Optional 객체가 비어있다면 파라미터로 넘어온 함수를 통해 생성된 객체를 반환한다.

- **`orElseThrow(Supplier<? extends X> exceptionSupplier)`**

Optional 객체가 비어있다면 파라미터로 넘어온 함수를 통해 생성된 예외를 던진다.

이렇게 Optional을 사용하면 null 처리를 Optional 클래스에 위임하였기 때문에 **Optional 적용 후에 null 체크를 할 필요가 없다.**

## DAO 계층에서의 Optional 사용

DAO 계층에서는 특정 필드값을 기준으로 데이터를 DB로 부터 조회한다.

이때 해당 데이터가 DB에 존재할지, 안할지 확신이 없는 경우라면 null 참조를 반환할 가능성이 존재한다. $($NPE 발생 위험)

```java
@Override
public Optional<User> getUserById(Integer userId) {
    return Optional.ofNullable((User) sessionFactory.getCurrentSession()
    .createCriteria(User.class),add(Restrictions.eq("userId", userId))
    .uniqueResult());
}
```

null 값일 가능성이 있는 객체를 정적 펙토리 메서드 `.ofNullable()`를 통해서 Optional 클래스로 감싸주었다.

덕분에 getUserById 메서드는 반환값이 존재하지 않더라도 null이 아닌 빈 `Optional` 객체를 반환하게 된다.

## Service 계층에서의 Optional 처리

Service 계층에서는 DAO 계층으로 부터 전달받은 Optional 객체에 값이 존재하는지 안하는지에 따라 처리를 다르게 해야한다.

Optional 객체에 값이 존재한다면 그 값을 반환한다.

Optional 객체가 비어있다면 클라이언트에게 적절한 예외를 던져줘야 한다.

```java
@Override
@Transactional
public void deleteUser(Integer userId) throws BaseException, ResourceNotFoundException {

    if (userId == null) {
        throw new BaseException(" 인자는 null값이 될 수 없습니다. " + userId); 
    }

    final User user = userDao.getById(userId)
        .orElseThrow(() -> new UnauthorizedRequestException("존재하지 않는 사용자입니다."));
    user.setEnabled(false);
    userDao.update(user);
}

@Override
@Transactional(readOnly = true)
public User getUserById(Integer userId) throws BaseException {

    if (userId == null) {
        throw new BaseException(" 인자는 null값이 될 수 없습니다." + userId);
    }

    return userDao.getById(userId)
        .orElseThrow(() -> new ResourceNotFoundException(Integer.valueOf(userId)));
}
```

위의 예제에서는 `Optional.orElseThrow()`를 통해서 Optional이 비었을 경우 예외를 던져주었다.