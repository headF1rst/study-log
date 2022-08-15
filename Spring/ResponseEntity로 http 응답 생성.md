스프링 부트 프로젝트를 진행하면서 REST API 규약대로 클라이언트에게 데이터를 전송해 주게 되었습니다.

하지만 단순하게 데이터만 클라이언트에게 전달해 주기에는 서버에 전달해줄 데이터 뿐만이 아니라 상태 및 정보들이 많았습니다. 서버의 응답 정보를 종합해서 클라이언트에 보내주기 위해서 HTTP에 대해서 다시 상기하는 시간을 갖게 되었습니다.

네트워크 상에서 송수신 되는 데이터들은 웹 서비스들 간의 대화가 가능하도록 특정 규약을 기준으로 전송됩니다.

이때 데이터의 형식을 규약에 맞게 전송해야 하는데 이 규약을 `HTTP(HyperText Transport Protocol)` 이라고 합니다.

## HTTP

HTTP란 클라이언트와 서버 사이에 요청과 응답을 처리하기 위한 규약입니다.

- HTML 문서 및 여러 종류의 데이터를 교환하기 위한 프로토콜입니다.
- TCP/IP 기반으로 되어있습니다.
- 클라이언트와 서버의 request, response 구조로 되어있습니다.
- stateless - 각 요청/응답은 독립적입니다.
    - 진행과정의 저장이 필요하면 쿠키나 세션을 사용합니다.
- HTTP 요청, 응답 모두 세 가지 요소로 구성됩니다.
    - HTTP 요청/응답 구성 요소 - Start Line, Headers, Body

### HTTP 요청 요소 $($Start Line, Headers, Body)

1. ***Start Line - method, URL, HTTP Version***

서버에 요청을 받아들이는 첫 줄입니다.

1. ***Headers***

요청에 대한 접속 운영체제, 인증 정보와 같은 부가 정보를 담고 있습니다.

1. ***Body***

요청에 관련된 json, html과 같은 구체적 데이터가 포함됩니다.

### HTTP 응답 요소 $($Status Line, Headers, Body)

HTTP 응답은 요청과 같은 Headers와 Body 그리고 Status Line으로 구성됩니다.

1. ***Status Line***

HTTP 버전과 함께 요청에 대한 처리 상태를 나타낸다. 200, 404와 같은 숫자 코드로 상태를 나타냅니다.

스프링 부트에서는 이러한 응답 요소를 담은 `HTTP Response` 를 만든 다음 반환해서 REST API 규약을 지켜야합니다. 때문에 세 가지 요소를 포함하는 객체를 만들어 줘야만 하는데 이를 데이터로 받아서 자동으로 구성해주는 것이 `@ResponseBody` 와 `ResponseEntity` 입니다.

## @ResponseBody

`@ResponseBody` : HTTP 규격에 맞는 응답을 만들어주기 위한 어노테이션

HTTP `요청을 객체로` 변환하거나 `객체를 응답으로` 변환하는 `HtttpMessageConverter` 를 사용합니다. 

HttpMessageConverter는 해당 어노테이션이 붙은 대상을 ResponseBody에 직렬화를 하는 방식으로 동작합니다. 

따라서 Controller에서 반환할 객체나 메서드에 `@ResponseBody` 를 붙히는 것만으로 HTTP 규격에 맞는 값을 반환하는게 가능합니다.

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseBody
@ResponseStatus(HttpStatus.OK)
public MoveResponseDto move(@PathVariable String name, 
														@RequestBody MoveDto moveDto) {
  String command = makeMoveCmd(moveDto.getSource(), moveDto.getTarget());
    springChessService.move(name, command, new Commands(command));
    
  MoveResponseDto moveResponseDto = new MoveResponseDto(springChessService
      .continuedGameInfo(name), name);
      
  return moveResponseDto;
}
```

어노테이션을 메서드에 추가해주는 것만으로 HTTP 규격에 맞는 응답을 생성한다는 점이 `@ResponseBody` 의 장점입니다.

만약 해당 메서드를 가진 Controller 위에 `@RestController` 를 붙이면 `@ResponseBody` 를 생략하는게 가능합니다. 이미 해당 어노테이션에 명시가 되어있기 때문입니다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody //@ResponseBody 명시
public @interface RestController {
	@AliasFor(annotation = Controller.class)
	String value() default "";
}
```

하지만 Header에 대해서 유연한 설정을 가져가지 못한다는 것

그리고 메서드 밖에 상태 코드 어노테이션을 별도로 설정 해주어야 한다는 단점이 있습니다.

이는 `@ResponseBody` 만 사용시에 별도의 뷰를 제공하지 않고, 데이터만 전송하는 형식이기 때문입니다.

이와같은 문제들을 해결해 주는것이 바로 `ResponseEntity` 객체입니다.

## ResponseEntity 객체

`ResponseEntity` : HTTP 응답을 만들어주기 위한 객체

HTTP 응답으로 변환될 정보를 모두 담은 요소들을 객체로 만들어서 반환해줍니다.

객체의 구성요소에서 HttpMessageConverter는 응답이 되는 본문을 처리해준 다음, RESTTemplate에 나머지 구성 요소인 상태를 넘겨줍니다. 

이를 통해서 앞서 언급한 ResponseBody의 문제점을 해결할 수 있습니다.

다음은 ResponseEntity 객체의 구조입니다.

```java
// ResponseEntity 구조
public class ResponseEntity<T> extends HttpEntity<T> {

	private final Object status;
...
}
```

ResponseEntity 클래스는 다음과 같이 필드값으로 `status` 만 갖고있습니다.

덕분에 ResponseEntity에서 직접적으로 상태 코드를 지정할 수 있습니다.

나머지 부분은 ResponseEntity가 상속한 `HttpEntity` 에 구현되어 있습니다.

```java
//HttpEntity 선언 구조
public class HttpEntity<T> {
    public static final HttpEntity<?> EMPTY = new HttpEntity<>();
  
  
    private final HttpHeaders headers;
  
    @Nullable
    private final T body;
}
```

HttpEntity에서는 제네릭 타입으로 Body 필드 값을 가질 수 있습니다.

Body 필드가 제네릭 타입이기 때문에 바깥에서 Wrapping 될 타입을 지정하는게 가능합니다.

Wrapping된 객체들은 자동으로 HTTP 규격에서 Body에 들어갈 수 있도록 변환 됩니다.

또한 필드 타입의로 `HttpHeaders` 를 가지고 있습니다.

따라서 ResponseBody와 다르게 객체 안에서 Header를 설정해 줄 수 있습니다.

## Constructor 대신 Builder를 사용하자

ResponseEntity는 Constructor와 Builder 두 방법으로 사용하는 것이 가능합니다.

먼저 Constructor를 활용하여 ResponseEntity를 사용한 예는 다음과 같습니다.

```java
public ResponseEntity<MoveResponseDto> move(@PathVariable String name,
    @RequestBody MoveDto moveDto) {
    HttpHeaders headers = new HttpHeaders();
    headers.set("Game", "Chess");
    
    String command = makeMoveCmd(moveDto.getSource(), moveDto.getTarget());
    springChessService.move(name, command, new Commands(command));
    
    MoveResponseDto moveResponseDto = new MoveResponseDto(springChessService
        .continuedGameInfo(name), name);

    return new ResponseEntity<MoveResponseDto>(moveResponseDto, headers, HttpStatus.valueOf(200)); // ResponseEntity를 활용한 응답 생성
}
```

HTTP 응답 타입은 `ResponseEntity<반환할 타입>` 으로 지정합니다.

Constructor를 사용할 경우, Body, Header, Status를 인자로 넣어서 생성합니다.

위 예시에서는 moveResponseDto 객체가 Body 부분에 들어가서 응답으로 전송 됩니다.

이와 같이 HTTP 응답에 필요한 대표 요소들을 지정하여 응답을 만들 수가 있습니다.

단, Constructor 대신 Builder를 활용하여 ResponseEntity를 사용하는걸 권장하고 있습니다.

그 이유는 숫자로 된 상태 코드를 넣을 때, 잘못된 숫자를 넣을 수 있는 실수 때문입니다. 

따라서 Builder Pattern을 활용한다면 각 상태에 매칭되는 숫자 코드를 외울 필요 없이 적절한 Builder 메서드를 선택하면 됩니다.

```java
// Constructor 활용
return new ResponseEntity<MoveResponseDto>(moveResponseDto, headers, HttpStatus.valueOf(200));

// Builder 활용
  return ResponseEntity.ok()
        .headers(headers)
        .body(moveResponseDto);
```