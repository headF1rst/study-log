프로그래밍을 하다 보면 사물을 복합물로 표현하는 것이 유용한 경우가 있다.

예를 들어 x, y로 이루어진 2차원 좌표, 시작 날짜와 끝 날짜로 이루어진 기간 등이 있다.

**VO**란 이와 같이 도메인에서 한 개 이상의 속성들을 묶어서 특정 값을 나타내는 객체$($복합물)을 의미한다.

## VO vs Entity

VO는 도메인 객체의 일종이며 식별 값을 갖는 Entity와 구별해서 사용한다.

VO는 어떤 조건들에 의해 엔티티와 구별되는지 알아보도록 하자.

### equals & hash code 메서드를 재정의해야 한다.

만약 두 객체의 타입과 내부 속성값이 모두 일치한다면 같은 객체로 취급되어야 한다.

하지만 실제로 두 객체의 동일성 비교를 해보면 둘을 서로 다른 객체로 구별한다.

```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

@Test
void equals() {
    Point point1 = new Point(1, 2);
    Point point2 = new Point(1, 2);

    // 동일성 비교
    assertThat(point1 == point2).isFalse(); // True
}
```

두 point 객체의 타입과 가리키는 위치가 동일한데 다른 위치를 가리키고 있다고 판단한다.

이는 **동일성** 비교$($==)의 경우, 객체가 참조하는 메모리 주소값을 비교하기 때문이다.

point1과 point2가 참조하는 주소값은 다르고 같게 만들수 없다.

따라서 객체의 속성값을 기준으로 객체를 비교할 때는 **동등성** 비교 $($equals)를 통해 객체를 비교해야 한다.

이때 어떤 속성값들을 기준으로 동등성 비교를 할 것인지는 equals 메서드와 hashCode 메서드를 재정의 해서 정한다.

hashCode를 재정의 해줌으로서 특정 값을 기준으로 같은 해시 코드를 얻을 수 있고, 이는 해시값을 사용하는 컬렉션 등에서 객체를 비교하는 용도로 사용된다.

```java
@Override
public boolean equals(final Object o) {

    if (this == o) {
        return true;
    }

    if (o == null || getClass() != o.getClass()) {
        return false;
    }

    final Point point = (Point) o;
    return x == point.x && y == point.y;
}

@Override
public int hashCode() {
    return Objects.hash(x, y);
}
```

equals와 hashCode 재정의를 통해서 VO를 사용할 때 속성값이 같은 객체는 같은 객체임을 보장할 수 있게 되었다.

### VO는 Setter가 없는 불변 객체.

엔티티는 내부 속성값들이 변경되더라도 식별 값을 갖고 있기 때문에 같은 객체로 인식하고 추적 가능하다.

하지만 속성값이 식별 값인 VO의 경우, 속성값이 바뀌면 추적이 불가하다.

때문에 VO를 불변 객체로 만들어야 한다.

Setter가 없는 불변 객체로 VO를 만들고 **생성자**를 통해 객체가 생성되면서 값이 한 번 주입되는 순간 이후로는 속성 값이 변경되어서는 안된다.

VO에 Setter를 사용한 안좋은 예시를 한번 보도록 하자.

```java
@Getter @Setter
@EqualsAndHashCode
public class Order {

    private String food;
    private String spicy;
    private int quantity;
}

public static void main(String[] args) {

    Order firstOrder = new Order();
    firstOrder.setFood("엽기 떡볶이");
    firstOrder.setSpicy("덜 매운맛");
    firstOrder.setQuantity(2);

    Order secondOrder = firstOrder; // !!!
    secondOrder.setSpicy("오리지널 매운맛");
}
```

Order라는 VO를 구현하였다.

엽떡 매장에 엽기 떡볶이 주문 2건이 들어왔다.

2건 모두 덜 매운맛 2개를 주문해서 첫번째 주문을 생성하고 두번째 주문은 첫번째 주문을 복사하였다.

이때 두 번째 주문한 손님 중 한 분이 엽떡은 오리지널이라며 오리지널 매운맛으로 변경하였다.

첫번째 손님은 주문한 떡볶이를 받았지만 너무 매워서 한입도 먹지 못했다. 알고 보니 오리지널 매운맛이 나온것이다.

이게 도대체 어떻게 된 일 일까?

두 번째 주문은 첫 번째 주문의 속성 값을 복사한 것이 아닌 참조하고 있는 **메모리의 주소값을 복사**했기 때문에 두 번째 주문의 맵기를 변경한 결과로 첫 번째 주문도 변경된 값을 가리키게 된 것이다.

이러한 오류를 방지하고자 VO에는 Setter를 사용하지 않으며 VO인 Order 객체를 불변으로 만들어야 한다.

다음과 같이 생성자를 통해서 값을 받고 Order객체는 불변이 되어야 한다.

```java
@Getter
public class Order {

    private String food;
    private String spicy;
    private int quantity;

    public Order(String food, String spicy, int quantity) {
        this.food = food;
        this.spicy = spicy;
        this.quantity = quantity;
    }
}

public static void main(String[] args) {

    Order firstOrder = new Order("엽기 떡볶이", "덜 매운맛", 2);

    Order secondOrder = new Order("엽기 떡볶이", "덜 매운맛", 2);

    secondOrder = new Order("엽기 떡볶이", "오리지널 매운맛", 2); // 주문 변경
}
```

VO의 값을 변경할 일이 생긴다면 생성자를 통해 객체를 새로 생성하고 재할당해 주어야 한다.

### VO를 사용하는 이유

VO를 사용하여 도메인을 설계하면 객체가 생성될 때 해당 객체안에 제약 사항을 추가할 수 있다.

뿐만 아니라 생성될 인스턴스가 정해져 있는 경우, 미리 인스턴스를 생성해놓고 캐싱하여 성능을 높이는 것도 가능하다.

엔티티의 원시 값들을 VO로 포장하면 엔티티가 지나치게 커지는 것을 방지할 수도 있어서, 테이블 관점이 아닌 객체 지향적인 관점으로 프로그래밍을 하는것이 가능하다.

때문에 우리는 VO와 엔티티의 역할을 명확히 나누어서 보다 객체 지향적으로 설계하도록 노력해야 한다.

**참고 자료** 📚

- https://tecoble.techcourse.co.kr/post/2020-06-11-value-object/