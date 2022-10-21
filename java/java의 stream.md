# Java 8의 Stream

Steam은 컬렉션 저장 요소를 하나씩 참조하며 함수형 인터페이스를 적용하며 반복적으로 처리할 수 있도록 해주는 기능으로 자바 8 부터 추가되었습니다.

먼저 제가 그동안 작성해왔던 방식으로 컬렉션 요소를 순회하는 코드를 살펴보겠습니다.

``` java
// 김씨 성을 가진 회원수를 알아내는 코드
List<String> names = Arrays.asList("김채은", "김지민", "고산하", "이준석", "황승환");
Long count = 0

for (String name : names) {
    if (name.contains("김")) { // 들여쓰기 1번
        count += 1; // 들여쓰기 2번
    }
}
```

이름 컬렉션을 순회하면서 "김"씨 성의 이름이 몇개인지 찾는다고 가정한다면 `for문` 과 `if문`이 필요하며 총 두개의 들여쓰기가 필요합니다.

반면 이를 **Stream**으로 구현한다면 한 줄의 코드로 구현이 가능합니다.

``` java
List<String> names = Arrays.asList("김채은", "김지민", "고산하", "이준석", "황승환");
Long count = 0

count = names.stream().filter(x -> x.contians("김")).count();
```

이처럼 Stream은 불필요한 들여쓰기가 없기 때문에 직관적이고 가독성이 좋다는 장점이 있습니다.

## Stream의 구조

> 컬렉션 데이터.스트림생성().중개연산().최종연산();

### 중개 연산

**1. Filter**
조건에 맞는 경우만 통과한다.

``` java
List<String> names = Arrays.asList("김채은", "김지민", "고산하", "이준석", "황승환");
Stream<String> c = names.stream().filter(name -> name.contains("김"));
// 결과 -> 김채은, 김지민
```

위의 예시에서는 람다식을 이용해 `name`로 스트림의 요소를 받고 각 요소에 "김" 성을 가진 이름만을 걸러냅니다.

**2. Map**
스트림의 각 요소에 연산을 적용하는데 쓰인다.

``` java
List<String> names = Arrays.asList("김채은", "김지민", "고산하", "이준석", "황승환");
names.parallelStream()
        .map(name -> name.concat(" 17학번"))
        .forEach(name -> System.out.println(name));
```

**3. Sorted**
스트림의 요소들을 정렬하는데 쓰인다.

``` java
names.stream()
        .sorted()
        .collect(toList());
```

### 최종 연산

**1. collect**
스트림의 값들을 toMap, toList, toSet등의 컬랙션으로 변환해준다.

**2. forEach**
각 요소를 돌면서 처리할 수 있게 해준다.

``` java
names.parallelStream()
        .map(name -> name.concat(" 17학번"))
        .forEach(name -> System.out.println(name));
```

**3. allMatch, anyMatch, noneMatch**

- allMatch = 스트림의 모든 요소들이 조건을 만족하면 `true` 반환

- anyMatch = 스트림의 요소중 하나라도 조건 만족 여부 판단해 `boolean` 반환

- noneMatch = 스트림의 모든 요소들이 조건을 안 만족 하면 `true` 반환

---

스트림의 사용할때 주의해야할 몇가지가 존재합니다.

1. Stream은 재사용이 불가능하다.

<br>

2. 병렬 스트림은 여러 쓰레드가 작업한다.
`stream()` 이 아닌 `parallelStream()`으로 병렬 스트림을 만드는게 가능합니다.

여러 쓰레드가 처리해주기 때문에 성능면에서 유리해 보이지만 애플리케이션에서 사용하는 쓰레드가 많거나 스트림의 요소 수가 많지 않다면 쓰레드를 사용하는데 드는 오버헤드가 더 클 수 있습니다.

<br>

3. 중개 연산은 미리하지 않고 지연 연산을 한다.

```java
Stream<String> c = names.stream().filter(name -> name.contains("김"));
c.forEach(name -> System.out.println(name));
```

위의 코드와 같은 경우, 중개연산은 미리 계산되고 있지 않다가 최종연산이 적용될 때 같이 실행됩니다.

덕분에 두번 순회하면서 계산되지 않습니다.

**참고 자료 📚**

- https://jeong-pro.tistory.com/165