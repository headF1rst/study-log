계산기 프로그램에서 사칙연산자가 주어졌을때 올바른 사칙연산자로 연산을 
수행하는지에 대한 테스트를 작성하고자 합니다.

`@Test` 어노테이션을 사용하면 다음과 같이 덧샘, 뺄샘, 곱샘, 나눗샘 별로 모든 
테스트 코드를 만들어 줘야 합니다.

```java
@DisPlayName("덧샘 연산자의 경우 테스트")
@Test
void plusOperatorTest() {
		assertThat(Operator.findOperator("+").getSymbol()).isEqualTo("+");
}

@DisPlayName("뺄샘 연산자의 경우 테스트")
@Test
void minusOperatorTest() {
		assertThat(Operator.findOperator("-").getSymbol()).isEqualTo("-");
}

...
```

위의 케이스의 경우에, 테스트 메서드에 인자값을 주지 못하는 `@Test` 를 
사용하게 되면 중복되는 부분이 생기게 됩니다.

중복되는 테스트 메서드가 많아지는 것 또한 가독성에 좋지 않습니다.

오늘은 JUnit5에서 제공하는 `@ParameterizedTest` 를 사용해서 하나의 
메서드로 다른 인자를 주어 테스트를 하는 방법에 대해 기록하고자 합니다.

```java
@DisplayName("올바른 사칙연산자의 경우 테스트")
@ParameterizedTest
@CsvSource(value = {"+ : +", "- : -", "* : *", "/ : /"}, delimiter = ':')
void correctOperators(String input, String output) {
    assertThat(Operator.findOperator(input).getSymbol()).isEqualTo(output);
}
```

`@ParameterizedTest` 어노테이션을 사용하면 하나의 테스트 메서드로 여려 
개의 파라미터에 대해서 테스트가 가능합니다. 

## Dependency

먼저 라이브러리를 사용하기 위해 의존성을 추가해 주었습니다.

### Gradle - build.gradle

```java
dependencies {
	testImplementation("org.junit.jupiter:junit-jupiter-params:5.4.2")
}
```

### Maven - pom.xml

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
```

## @ValueSource

리터럴 값의 배열에 대한 접근을 제공하는 어노테이션입니다.

쉽게 말해서 배열에 담긴 원소들이 하나씩 함수 인자값으로 주어지게 됩니다.

short, byte, int, long, float, double, char, boolean, string, 
class 배열을 제공합니다.

```java
@Target({ ElementType.ANNOTATION_TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@API(status = STABLE, since = "5.7")
@ArgumentsSource(ValueArgumentsProvider.class)
public @interface ValueSource {

	short[] shorts() default {};
	byte[] bytes() default {};
	int[] ints() default {};
	long[] longs() default {};
	float[] floats() default {};
	double[] doubles() default {};
	char[] chars() default {};
	boolean[] booleans() default {};
	String[] strings() default {};
	Class<?>[] classes() default {};
}
```

이제 `@ValoueSource` 를 사용해서 짝수 여부를 확인하는 메서드의 테스트 
코드를 작성해보도록 하겠습니다.

```java
@DisplayName("짝수 여부 테스트")
@ParameterizedTest
@ValueSource(ints = {2, 4, 12})
void isEvenTest(int input) {
		assertThat(Numbers.isEven(input)).isTrue();
}
```

## @CsvSource

앞의 @ValueSource의 경우 input은 다르지만 모든 input의 output은 항상 
같았습니다.

만약 **입력값에 따라 output이 다른 경우**를 테스트 하려면 `@CsvSource` 
를 사용할 수 있습니다.

CSV(Comma Separated Values)는 이름에서 알 수 있듯이 콤마(,)를 
디폴트값으로 하여 input과 output을 구분합니다.

구분자는 `delimiter` 를 직접 정의하여 변경 가능합니다.

```java
@DisplayName("짝수 여부 테스트")
@ParameterizedTest
@CsvSource(value = {"1 : false", "2 : true", "13 : false"}, delimiter = ':')
void isEvenTest(int input, boolean output) {
		assertThat(Numbers.isEven(input)).isEqualTo(output);
}
```

## @NullAndEmptySource

테스트 메서드의 인자로 null값과 empty value 모두 전달하기 위해서 
`@NullAndEmptySource` 어노테이션을 사용할 수 있습니다.

`@ValueSource` 등의 어노테이션과 함께 사용이 가능합니다.

```java
@DisplayName("input이 Null 혹은 Empty면 IllegalArgumentException 발생 여부 테스트")
@ParameterizedTest
@NullAndEmptySource
void checkExceptionWhenInputIsNullOrEmpty(String input) {
      assertThatExceptionOfType(IllegalArgumentException.class)
            .isThrownBy(() -> {
                    InputParser.checkNullOrEmpty(input);
             });
    }
```

### @NullSource

테스트 인자로 null값을 전달합니다.

primitive data의 경우 null 값을 허용하지 않기 때문에 primitive 
인자값에는 사용할 수 없습니다.

### @EmptySource

테스트 인자로 빈 값을 전달합니다. “"

## @MethodSource

보다 복잡한 인자값을 가독성 좋게 테스트하기 위해서는 `@MethodSource` 
어노테이션을 사용합니다.

`@MethodSource` 에 설정하는 이름은 테스트 하고자 하는 메서드 이름입니다.

테스트를 위한 별도의 메서드를 구현해 주어야 하는데, 덕분에 다른 테스트 간에 
인자를 공유하는데 유용합니다.

만약 클래스 단위 생명 주기가 아닌 경우, static 메서드여야 합니다.

다음은 `@MethodSource` 를 사용하여 Member 도메인 유효성 테스트를 진행한 
예제입니다.

```java
@DisplayName("Member 생성 유효성 테스트")
@ParameterizedTest
@MethodSource("invalidParameters")
void MemberEntityCreateTest(String email, String password, String message, String exceptionMessage) {
		assertThatExceptionOfType(IllegalArgumentException.class)
				.isThrownBy(() -> {
							new Member(email, password);
					}).isEqualTo(exceptionMessage);
}

private static Stream<Arguments> provideStringsForCalculate() throws Throwable {
    return Stream.of(
        Arguments.of(VALID_NAME, VALID_EMAIL, "p@ssw0rd", "password 대문자 제외", PASSWORD_NOT_MATCH_MESSAGE),
        Arguments.of(VALID_NAME, VALID_EMAIL, "P@SSW0RD", "password 소문자 제외", PASSWORD_NOT_MATCH_MESSAGE),
        Arguments.of(VALID_NAME, "asdfasd", VALID_PASSWORD, "email 양식 틀림", EMAIL_NOT_MATCH_MESSAGE)
    );
}
```

**참고 자료 📚**

[parameterizedTest - baeldung](https://www.baeldung.com/parameterized-tests-junit-5)

[[JUnit] JUnit5 사용법 - Parameterized Tests - 
Heee's Development Blog](https://gmlwjd9405.github.io/2019/11/27/junit5-guide-parameterized-test.html)

[JUnit 5 Parameterized Tests 사용하기](https://dublin-java.tistory.com/56)