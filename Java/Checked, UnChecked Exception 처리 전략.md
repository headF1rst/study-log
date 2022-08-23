# Checked, unChecked Exception 처리 전략

## Checked, Unchecked Exception 차이

![스크린샷 2022-08-23 오후 11.48.29.png](https://i.imgur.com/cl8wEyF.png)

### Error - 시스템 레벨의 오류

Error는 시스템이 비정상적인 상황에서 발생한다.

시스템 레벨에서 발생하기 때문에 개발자가 미리 예측할 수 없으며 처리할 방법도 없다.

따라서 어플레케이션 단에서는 `OutOfMemoryError` 나 `ThreadDeath` 같은 Error에 대한 처리를 고려하지 않아도 된다.

### Checked, Unchecked Exception

Checked, Unchecked는 개발자들이 구현한 어플리케이션 코드에서 예외가 발생한 경우에 사용한다.

상속 구조에서 알 수 있듯이 두 Exception을 구분하는 포인트는 다음과 같다.

- `Unchecked Exception` - Runtime Exception을 상속한다.
- `Checked Exception` - Runtime Exception을 상속하지 않는다.

## Unchecked Exception

- 명시적인 예외 처리를 강제하지 않기 때문에 Unchecked Exceptoin이라 한다.
- catch로 잡거나 throw로 호출한 메서드로 예외를 던지지 않아도 상관 없다.

## Checked Exception

- 반드시 명시적으로 처리해야하기 때문에 Checked Exception이라 한다.
- `try catch` 를 통해 에러를 잡든 `throw` 를 통해서 호출한 메서드로 예외를 던져야 한다.
    - throw로 예외를 던진다면 예외를 처리할 객체의 책임은 어떻게 판별할까?

```java
@Test
public void throws_던지기() throws JsonProcessingException {
  final ObjectMapper objectMapper = new ObjectMapper();
  final Member member = new Member("yun");
  final String valueAsString = objectMapper.writeValueAsString(member);

}

@Test
public void try_catch_감싸기() {
  final ObjectMapper objectMapper = new ObjectMapper();
  final Member member = new Member("yun");
  final String valueAsString;
  try {
    valueAsString = objectMapper.writeValueAsString(member);
  } catch (JsonProcessingException e) {
    e.printStackTrace();
    throw new RuntimeException();
  }
}
```

위의 `JsonProcessingException` 은 IOException Exception을 상속하는 **Checked Exception** 이다.

그렇기 때문에 `throws` 로 상위 메서드로 예외를 넘기던지 자신이 `try catch` 해서 `throw` 를 던지든 해야한다.

반면 **Unchecked Exception** 은 명시적인 예외 처리를 하지 않아도 된다.

### Rollback 여부 - 스프링의 트랜잭션 처리 기준

```java
@Service
@RequiredArgsConstructor
@Transactional
public class MemberService {

  private final MemberRepository memberRepository;

  // (1) RuntimeException 예외 발생
  public Member createUncheckedException() {
    final Member member = memberRepository.save(new Member("sanha"));
    if (true) {
      throw new RuntimeException();
    }
    return member;
  }

  // (2) IOException 예외 발생
  public Member createCheckedException() throws IOException {
    final Member member = memberRepository.save(new Member("channie"));
    if (true) {
      throw new IOException();
    }
    return member;
  }
}
```

- (1) RunTimeException 예외를 발생 시키면 sanha라는 member는 `rollback` 이 진행된다. (unchecked)
- (2) 반면 IOExcetion 예외가 발생 되더라도 channie는 `rollback` 되지 않으며 `DB 트랜잭션이 commit` 까지 완료된다.

checked, unchecked 예외에 따라서 rollback을 할지 말지 여부는 개발자가 정한다.

하지만 스프링에서는 `RunTimeException` 계열의 예외가 발생하면 기본적으로 rollback을 진행한다.

반면 Unchecked Exception의 경우 rollback을 하지 않는다.

기본적으로는 이렇지만 스프링은 rollback 전략 옵션을 제공한다.

따라서 개발자가 원한다면 RunTimeException 계열 중에서도 특정 예외는 rollback을 하지 않게끔 설정하는 것도 가능하다.

그 반대로 Unchecked Exception이더라도 rollback을 하게끔 설정할 수 있다.

## Checked Exception이 Rollback 되지 않는 이유는?

Checked Exception은 복구가 가능하다는 매커니즘을 갖고 있다.

예를 들어, 특정 이미지 파일을 찾아서 전송해주는 함수에서 이미지를 찾지 못할 경우엔 디폴트 이미지를 전송하는 등의 복구 전략을 가질 수 있다.

```java
public void sendFile(String fileName){

    File file;
    try {
        file = FileFindService.find(fileName);
    } catch (FileNotFoundException e){ // FileNotFoundException은 IOException으로 checked exception이다.
        // 파일을 못찾았으니 기본 파일을 찾아서 전송 한다
        file = FileFindService.find("default.png");
    }

    send(file);
}
```

`try catch` 문을 사용하여 Checked Exception이 발생했을시에 복구 전략을 개발자가 정의해 주었다.

하지만 이런 식의 예외는 `try catch` 로 복구하는 것이 아니라 일반적인 코드의 흐름으로 제어하는 것이 바람직 하다.

```java
public void sendFile(String fileName){
    if(FileFindService.existed(filename)){
        // 파일이 있는 경우 해당 파일을 찾아서 전송
        send(FileFindService.find(fileName));    
    }else{
        // 파일이 있는 없는 경우 기본 이미지 전송
        send(FileFindService.find("default.png"));    
    }
}
```

사실 현실적으로 Checked Exception이 발생했을때 개발자가 복구 전략을 통해서 복구 할 수 있는 경우는 많지 않다.

고유한 이메일 값이 중복되어서 SQLException이 발생할 경우 어떻게 복구 할 수 있을까?

현실에서는 RuntimeException을 발생시켜서 rollback을 시키고 입력을 다시 유도하는 방법을 선택한다.

이때 해당 Exception을 발생시킬 때 명확하게 어떤 예외가 발생해서 Exception이 발생했는지 정보를 전달해주는게 중요하다.

이메일 중복의 경우 **DuplicateEmailException** (`Unchecked Exception` )을 발생시키는 것이 바람직하다.

Checked Exception을 만나면 **더 구체적인 Unchecked Exception**을 발생시켜서 정확한 정보를 전달하고 로직의 흐름을 끊어야 한다.

JPA의 구현체를 사용하더라도 Checked Exception을 직접 처리하지 않고 있는 이유 또한 적절한 RuntimeException으로 unchecked 예외를 던져주고 있기 때문이다.

## Checked Exception 처리 전략

```java
public class ObjectMapperUtil {

  private final ObjectMapper objectMapper = new ObjectMapper();

  // 예외처리를 throws를 통해서 위임하고 있습니다.
  public String writeValueAsString(Object object) throws JsonProcessingException {
    return objectMapper.writeValueAsString(object);
  }

  // 예외처리를 throws를 통해서 위임하고 있습니다.
  public <T> T readValue(String json, Class<T> clazz) throws IOException {
    return objectMapper.readValue(json, clazz);
  }
}
```

`writeValueAsString()` 과 `readValue()` 메서드는 Checked Exception을 발생시키기 때문에 반드시 예외 처리를 해야만 한다.

하지만 두 메서드는 예외를 해당 메서드를 호출한 상위 메서드로 던져버리기 때문에 메서드를 호출한 부분에서는 다시 throw를 하던지 try catch를 통해서 예외를 처리해야만 한다.

이처럼 무의미하고 반복적으로 예외를 던지는것은 상위 메서드에 더 많은 책임을 부여하기 때문에 좋지 않다.

### 더 구체적인 Unchecked Exception을 대신 발생 시켜라

```java
public String writeValueAsString(Object object) {
    try {
      return objectMapper.writeValueAsString(object);
    } catch (JsonProcessingException e) {
      throw new JsonSerializeFailed(e.getMessage());
    }
  }

  public <T> T readValue(String json, Class<T> clazz) {
    try {
      return objectMapper.readValue(json, clazz);
    } catch (IOException e) {
      throw new JsonDeserializeFailed(e.getMessage());
    }
  }
```

앞서 Checked Exception을 발생 시켰던 두 메서드를 `try catch` 문으로 감쌌다.

그 다음 `RuntimeException` 을 상속받은 커스텀 Exception을 작성하여 Checked Exception 대신 Unchecked Exception을 던지도록 하였다.

덕분에 메서드를 사용하는 곳에서는 아무런 예외처리를 진행하지 않아도 된다.

해당 예외가 왜 발생했는지에 대해서 에러 메시지 뿐만 아니라 구체적인 정보를 전달해 주는것이 더욱 바람직하다.

## 마치며

예외 복구 전략이 가능하고 명확하다면 Checked Exception을 `try catch` 로 잡고 복구를 진행하는 것이 좋다. 하지만 이러한 경우는 흔치 않다.

- Checked Exception이 발생하면 더 구체적인 Unchecked Exception을 발생시킨다
    - 예외에 대한 구체적인 메시지를 전달한다.
    
- 무책임하게 상위 메서드로 예외를 던지는(`throw` ) 것은 지양한다.
    - 상위 메서드들의 책임이 그만큼 증가하기 때문이다.