# equals와 hashCode 그리고 재정의 시점

`equals` 와 `hashCode` 는 Object 객체에 정의되어있다. 즉, 모든 자바 객체는 equals & hashCode 함수를 상속받고 있다.

equls & hashCode에 대해 알아보기 전에 값 비교에 일반적으로 사용하는 `==` 연산자의 결과를 알아보자

```java
int a = 1;
int b = 1;

println(a == b); // ture
```

보통 두 값의 동일 여부를 확인할때 `==` 연산을 사용한다.

== 연산자는 두 대상의 주소값을 비교하며 기본 타입인 Call by Value 값들은 주소값을 가지지 않는 형태이기 때문에 단순 값을 비교하여 같다는 결론이 나온다.

그렇다면 다음의 결과는 어떨까?

```java
String a = "abcd";
String b = "abcd";
String c = new String("abcd");
String d = new String("abcd");

System.out.println(a == b); // 1. true
System.out.println(a == c); // 2. false
System.out.println(c == d); // 3. false
```

2, 3번 결과의 경우, String 객체가 생성되면서 Heap 영역에 서로 다른 주소값을 할당 받기 때문에 false라는 결과가 당연하게 느껴진다.

반면 1번 결과의 경우, 기본 타입이 아닌 두 String 객체를 비교했는데 true가 나온게 이상하다.

두 객체 비교값이 true인 이유는, a, b String 변수를 `리터럴 방식` 으로 생성한 다음 비교했기 때문이다.

String 변수는 두가지 방법으로 생성할 수 있다.

- 리터럴을 이용한 방식
- new 연산자를 이용한 방식

### 리터럴을 이용한 String 생성

리터럴을 사용하여 String 객체를 생성하게 되면 내부적으로 String의 `intern()` 메서드가 호출됩니다. intern() 메서드는 주어진 문자열이 string constant pool에 존재하는지 검색하고 있다면 그 주소값을 반환하고 없다면 string constant pool에 넣고 새로운 주소값을 반환합니다.

즉, 1번의 경우 b를 생성하는 시점에서 “abcd”라는 값이 a에 의해 string constant pool에 존재하기 때문에 a의 주소값을 반환하게 됩니다. 때문에 `a == b` 의 결과가 같은 주소값에 의해서 true라는 결과가 나오게 됩니다.

### new 연산자를 통한 String 생성

new 연산자를 통해 String을 생성하면, Heap 영역에 고유의 주소값을 갖고 저장됩니다.

c, d는 각각 new 연산자에 의해 서로 다른 주소값을 할당 받았기 때문에 == 연산 결과 false가 나오게 됩니다.

## equals()

`equals()` 메서드는 두 객체가 동일한지 확인하기 위한 목적으로 사용된다.

Object 객체의 equals 메서드 구현 부분을 살펴보면 해당 객체와 매개 변수로 주어진 객체가 **동일한 메모리 주소를 가리키는지** 여부를 판단한다는 것을 알 수 있다.

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

그렇다면 String 객체간의 equals 연산도 내부적으로 == 연산을 사용하므로 false가 나와야겠지만 true가 나오는데, 그 이유는 String 객체의 equals 연산이 재정의되었기 때문이다.

```java
String a = "abcd";
String b = new String("abcd");
System.out.println(a.eqauls(b)); // ture -> 서로 다른 주소값을 가지는데 true이다.

// String 클래스에 재정의된 equals 메서드
public boolean equals(Object anObject) {
		if (this == anObject) {
				return true;
		}

		if (anObject instanceof String) {
				String aString = (String) anObject;
				if (coder() == aString.coder()) {
						return isLatin1() ?
								StrinLatin1.equals(value, aString.value) : StringUTF16.equals(value, aString.value);
				}
    }
return false;
}
```

equals 내부 구현코드를 살펴보니 내부 값을 비교해서 리턴 해주는 것을 알 수 있다.

## hashCode()

hashCode 메서드는 런타임에 객체의 유일한 integer 값을 반환한다.

Object 클래스의 경우, 객체의 메모리 주소를 반환하도록 되어있다.

```java
class Person {
		String name;
		int age;

		Person(String name, int age) {
			this.name = name;
			this.age = age;
		}
}
```

다음과 같은 `Person` 클래스가 존재할 때,

```java
Person person1 = new Person("Sanha", 23);
Person person2 = new Person("Sanha", 23);
Person person3 = person2;

System.out.println(person1 == person2); // 1. false
System.out.println(person1.equals(person2)); // 2. false
System.out.println(person2.equals(person3)); // 3. true
```

서로 주소값이 다른 1, 2번의 경우 false가 나오고 3q번의 경우 person2와 3이 같은 주소를 참조하기 때문에 true를 반환한다.

hashCode의 결과값 또한 마찬가지이다.

```java
System.out.println(person1.hashCode()); // 12311411
System.out.println(person2.hashCode()); // 11111111
System.out.println(person3.hashCode()); // 11111111
```

hashCode는 해당 객체를 해시함수를 통과시킨 값을 리턴 해준다.

현실에서도 “Sanha”라는 이름의 23살 사람이 모두 동일 인물이라는 보장은 없기 때문에 두 객체의 equals 결과는 false가 나오는게 맞다.

하지만 주민등록 번호를 포함하는 경우를 생각해보자.

```java
class Person {
		int id;
		String name;

		Person(int id, String name) {
			this.id = id;
			this.name = name;
		}
}
```

주민등록 번호를 의미하는 `id` 값이 같은 경우엔 **동일 인물로 봐야한다.**

즉, 이처럼 객체의 값이 같은지 확인하고 싶을 때에 equals 메서드를 재정의 한다.

```java
@Override
public boolean equals(Object obj) {
		if (this == obj) {
				return true;
		}
		
		if (obj instanceof Person) {
				Person p = (Person) obj;
				return this.id == p.id && this.name == p.name;
		}
		return false;
}
```

## hashCode() Override의 필요성 - 해시 충돌

equlas 값이 같다면 hashCode 값도 같아야만 한다.

때문에 equals 메서드를 재정의 했다면, hashCode 메서드 또한 재정의 해줘야만 한다. (역도 마찬가지)

HashTable이나 HashSet, HashMap과 같은 자료구조는 저장하기 위한 위치를 선택하기 위해 `hashCode` 를 사용한다.

```java
Map<Person, Integer> map = new HashMap<>();

Person p1 = new Person(1, "sanha");
Person p2 = new Person(2, "phil");
Person p3 = new Person(1, "sanha");

map.put(p1, 1);
map.put(p2, 2);

System.out.println(map.get(p3)); // null
```

앞서 재정의한 `equals` 메서드에 의해서 p1과 p3는 같은 객체이며 p1을 HashMap에 저장하고 p3로 값을 불러왔을때 1이 나와야하지만 null값이 나오게 된다.

이는 p1과 p3의 해시값이 다르기 때문이며 p3의 해시값으로 저장된 key가 없으므로 null이 출력된 것이다.

따라서 hashCode를 equals 메서드와 함께 재정의 해주어야 한다.

```java
@Override
public int hashCode() {
		return Objects.hash(a, b);
}

public static int hash(Object...values) {
		return Arrays.hashCode(values);
}
```

만약 hashCode는 재정의 했지만 equals를 재정의 하지 않았다면 어떨까?

마찬가지로 null이 나오게 된다.

해시값이 같을때 해시충돌이 발생하면 자바의 HashMap은 `chaining` 기법을 사용한다.

이는 해당 해시값이 같은데 key값이 동일하지 않을 때에 해당 해시값의 버킷이 LinkedList를 가지며 값들을 가지게 된다. (동일할 때는 값을 덮어쓰며 LinkedList 크기가 8 이상일 경우에는 Tree 형태를 띄게 된다.)

이 동일 유무를 equals를 통해 알 수 있다.

```java
Map<Person, Integer> map = new HashMap<>();

Person p1 = new Person(1, "sanha");
Person p2 = new Person(1, "sanha");

map.put(p1, 1);

System.out.println(map.get(p2)); // null
```

hashCode만 재정의 해놓은 경우, p2는 p1과 같은 해시값을 갖기 때문에 map의 인덱스에는 잘 찾아가게 된다.

하지만 해당 해시값의 버킷에서 list또는 tree를 탐색할 때 key값이 같은 객체를 리턴해야 하는데 equals 메서드가 재정의 되지 않은 상태라면 동일성 유무를 판별할 수 없게된다.

즉, equals와 hashCode 메서드는 재정의를 같이 하거나 둘 다 하지 않아야 한다.