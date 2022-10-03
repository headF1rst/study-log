## MVC 패턴이란

유지보수를 용이하게 하기 위해서 각 객체의 변경 라이프 사이클을 기준으로 책임을 나누는 것이 중요하다. (단일 책임 or SOLID에 대한 질문 대비)

MVC 패턴이란 어플리케이션의 기능을 수행하기 위한 책임을 세가지 구성요소인 Model, View, Controller에게 분배하여 유지보수를 용이하게 한 패턴을 말한다.

쉽게 말해서 어떤 특정한 역할들에 대해 역할 분담을 할 때 가이드라인을 제시하는 방법 중 하나이다.

![스크린샷 2022-08-22 오후 1.29.42.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/38439138-1232-4999-8890-aba8bb7c1939/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-08-22_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_1.29.42.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221003%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221003T053516Z&X-Amz-Expires=86400&X-Amz-Signature=d3ceeb42d642a31d72a7345bc2a0379cc58863fa61e9ff01335fcb394f6db29d&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-08-22%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25201.29.42.png%22&x-id=GetObject)

- `Controller`
    
    클라이언트로 부터 HTTP request를 받아서 파라미터를 **검증하고, 비즈니스 로직을 실행한다.** 
    
    뷰에 전달할 결과 데이터를 조회해서 **모델에 담는다**
    

- `Model`
    
    뷰에 출력할 데이터를 저장해둔다.
    
    덕분에 뷰는 비즈니스 로직과 데이터 접근을 몰라도 되며 렌더링에만 집중할 수 있다.
    
- `View`
    
    HTML을 생성한다. 모델에 담겨있는 데이터를 사용해서 화면을 렌더링한다.
    

## 스프링의 MVC 구조

MVC 패턴을 사용한 프레임워크 중 가장 대표적인 프레임워크는 바로 Spring이다.

![스크린샷 2022-10-03 오전 2.09.19.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b0ac79cc-04da-4e5f-8b4f-8863868a25ba/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-10-03_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_2.09.19.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221003%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221003T053553Z&X-Amz-Expires=86400&X-Amz-Signature=7cd5b2b38447bcbc3089cb32721972432b39d81eadd9a1799cb3d1993a4565ad&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-10-03%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%25202.09.19.png%22&x-id=GetObject)

MVC 구조의 진행 과정을 살펴보도록 하자

- 클라이언트가 url 요청하면, DispatcherServlet으로 HttpRequest가 전달된다.
- DispatcherServlet은 HandlerMapping을 통해 해당 url과 매핑되어 있는 Controller를 탐색하여 찾아낸다.
- 찾아낸 Controller로 request가 전달되고 전달하기 위해 필요한 Model을 구성한다.
- Model은 페이지 처리에 필요한 정보들을 데이터베이스에 접근하여 쿼리문을 통해 가져온다
- 데이터베이스를 통해 얻은 Model 정보를 Controller에게 response 해주면, Controller는 이를 받아 Model을 완성시켜 DispatcherServlet에게 전달한다.
- DispatcherServlet은 ViewResolver를 통해 request에 해당하는 View 파일을 탐색 후 받아낸다.
- 받아낸 View 페이지 파일에 Model을 보낸 후 클라이언트에게 보낼 페이지를 완성시켜 받아낸다.
- 완성된 View 파일을 클라이언트에 response하여 화면에 출력한다.

## 스프링 MVC의 구성  요소

- DispatcherServlet
    - 모든 HttpRequest에 공통 로직을 처리한다.
    - url에 알맞는 Controller를 매핑해서 호출한다.

- HandlerMapping
    - 요청 url과 매핑된 Contorller를 찾아서 Dispatcher Servlet에게 전달한다.
    - HandlerMapping은 인터페이스이고 구현체로는 어노테이션 기반으로 url을 매핑하는 RequestMappingHandlerMapping, SimpleUrlHandlerMapping 등이 있으며 순서대로 탐색된다.

- Controller
    - 비즈니스 로직을 처리한다.
    - Model의 처리 결과를 담아서 Dispatcher Servlet에 반환한다.

- View Resovler
    - 논리경로를 물리경로로 변환해준다. (prefix, surfix)