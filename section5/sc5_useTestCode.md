# 테스트 코드를 이용한 테스트  
  
지금까지 작성한 코드가 올바르게 동작한지 확인하기 위해서 
1. HTTP 요청에 올바르게 동작했는지 확인한다
2. HTTP 응답이 예상한 결과로 만들어 졌는지 확인한다.  

### Request
+ Request Line: Method, Path, HTTP Version
+ Headers
+ Message Body
### Response
+ Status Line: HTTP Version, Status Code, Status Text
+ Headers
+ Message Body  
  
요청 객체와 응답 객체에 대한 정보를 눈으로 확인했습니다. 이제는 테스트 코드를 작성해 눈으로 하나하나 확인하지 않고 예상한 결과와 일치하는지 결과만 받을 수 있도록 하겠습니다.  
  
1. 웹 요청 테스트를 해야하기 때문에 스프링 컨테이너를 띄운다.  
    기존 스프링 컨테이너를 실행하거나 테스트 스프링 컨테이너를 실행해도 된다.
2. 웹 Api 요청을 코드로 작성하는데 사용하라고 스프링이 만들어준 간단한 클래스를 활용한다.
    1. RestTemplate
   2. TestRestTemplate  
하지만 `TestRestTemplate`을 사용할 예정입니다. 서버에서 처리하다가 문제가 있어서 400번대나 500번때 응답 코드를 보내주는 경우에 `RestTemplate`은 예외를 던집니다.  
   순수하게 예외를 검증하는 방법도 있지만 응답 자체를 순수하게 다 가져와서 상태코드와 컨텐트 타입,바디를 확인하는 방식으로 테스트 코드를 작성하려면 `TestRestTemplate`을 활용하는게 편합니다.  

#### 전체 테스트 코드
```java
@SpringBootTest
public class HelloApiTest {

    @Test
    void helloApi(){
        // 스프링 부트가 동작해야한다.
        // http localhost:8080/hello?name=spring
        TestRestTemplate rest = new TestRestTemplate();
        ResponseEntity<String> response = 
                rest.getForEntity("http://localhost:8080/hello?name={name}", String.class, "Spring");
        // 응답 3가지 검증
        // status, header, Hello Sring
        assertThat(response.getStatusCode()).isEqualByComparingTo(HttpStatus.OK);
        assertThat(response.getHeaders().getContentType().getType()).isEqualTo(MediaType.TEXT_PLAIN.getType());
        assertThat(response.getHeaders().getFirst(HttpHeaders.CONTENT_TYPE)).startsWith(MediaType.TEXT_PLAIN_VALUE);
        assertThat(response.getBody()).isEqualTo("Hello Spring");
    }
}
```
```java
ResponseEntity<String> response =
                rest.getForEntity("http://localhost:8080/hello?name={name}", String.class, "Spring");
```  
`getForEntity`는 `URL`,응답 타입,`{}`에 넣을 값을 매개변수로 전달합니다.  
해당 오브젝트는 그 결과를 response로 반환하고 이 오브젝트로 테스트를 진행하면 됩니다.  
  
#### 파라미터 타입에 대해서  
Java에서는 메서드로 호추할 때 파라미터 이름을 넣지 않습니다. 
이렇게 기본형 타입으로 작성하면 `IDE`가 파라미터 이름도 같이 보여줍니다. 
단점은 코드가 길어져서 한줄인지 두줄인지 계산하기가 어려울 수 있고, 코드리뷰 툴이나 git에 올라갔을 때 
`IDE`가 보여주는 정보도 사라집니다. 리뷰하는 다른 사람들은 메소드 호출을 보고 파라미터 타입이 많고 타입도 비슷하면
헷갈릴 수도 있다고 지적한다고 합니다.
  
#### MockMvc로 테스트 해보기  
```java
@WebMvcTest(controllers = {HelloController.class})
public class MockTest {

    @Autowired
    protected MockMvc mockMvc;

    @MockBean
    protected HelloService helloService;

    @Test
    @DisplayName("확인하기")
    void test() throws Exception {
        //given
        given(helloService.sayHello(anyString())).willReturn("Hello Spring");

        // when // then
        mockMvc.perform(
                        get("/hello")
                            .param("name","String")
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("Hello Spring"));
    }
}
```  
사실 `Mock`으로 테스트를 할 때에 `Controller`계층에서 `@Valid`나 검증에 대한 
예외 상황을 테스트한다. 서비스 -> 레포지토리 테스트는 별개로 테스트를 분리해서 테스트를 한다.  
하지만 이런 방식으로도 테스트를 할 수 있다는걸 기억하려고 작성했다.  
  
## DI와 단위 테스트
테스트 코드가 테스트하는 대상은 `HelloController.hello()`메소드 입니다. 컨트롤러는 `HelloService`를 의존하고 런타임에 `SimpleHelloService`를 주입받아서 정상적으로 동작하는지 테스트를 했습니다.  
이 방법보다 웹 요청을 처리하는 API와 비즈니스 로직을 처리하는 Service 계층을 고립시켜 테스트를 진행을 하면 스프링 컨테이너를 전부 띄우지 않고도 빠른 시간내로 테스트가 가능합니다.  
  
비즈니스 로직을 처리하는 Service 계층에 있는 `SimpleHelloService` 클래스는 순수 자바 클래스입니다. 
그러면 테스트 코드를 작성할 때 자바 코드로만 테스트 하는 것이 속도도 더 빠르고 다양한 테스트를 할 수 있습니다.  
```java
public class SimpleHelloServiceTest {
    @DisplayName("인사를 하면 메아리로 돌아온다")
    @Test
    void sayMooyahooo(){
        //given
        String requestName = "둘리야";
        HelloService helloService = new SimpleHelloService();
        //when
        String sayHello = helloService.sayHello(requestName);
        //then
        Assertions.assertThat(sayHello).isEqualTo(String.format("Hello %s", requestName));
    }
}
```  
그러면 반대로 `HelloController`의 검증 로직을 테스트 해보기 전에 기존 코드를 수정하겠습니다.  
```java
@GetMapping("/hello")
public String hello(String name) {
    if (!StringUtils.hasText(name)) {
        throw new IllegalArgumentException("값이 비어있습니다.");
    }
    return helloService.sayHello(name);
}
```  

이제 컨트롤러가 검증하는 로직을 테스트하는 코드를 작성하겠습니다.
```java
class HelloControllerTest {
    @DisplayName("이름이 null이거나 공백이면 예외가 발생한다.")
    @Test
    void nameIsEmptyThrowEx(){
        //given
        String name = null;
        HelloController helloController = new HelloController(requestName -> requestName);
        //when//then
        Assertions.assertThatThrownBy(() -> helloController.hello(name))
                .isInstanceOf(IllegalArgumentException.class);
    }
}
```  
해당 코드를 이해하기 쉽게 풀어서 작성하면 아래와 같습니다.
```java
@DisplayName("이름이 null이거나 공백이면 예외가 발생한다.")
@Test
void nameIsEmptyThrowEx(){
    //given
    String name = null;
    HelloController helloController = new HelloController(new HelloService() {
        @Override
        public String sayHello(String requestName) {
            return requestName;
        }
    });

    //when//then
    Assertions.assertThatThrownBy(() -> helloController.hello(name))
            .isInstanceOf(IllegalArgumentException.class);
}
```
지금은 `HelloService`가 메소드를 하나만 가지고 있어서 람다식으로 구현해서 전달했습니다. 
이렇게 테스트 코드를 작성할 수 있는 환경은 거의 없다고 생각하지만 여기서 설명하는 건 
하나의 단위로 고립시켜 동작하게 하여 컨트롤러만 테스트를 하면 속도가 빨라지고 다양한 테스트를 할 수 있다는 걸 보여줍니다.  

`SimpleHelloService`클래스는 들어온 데이터를 그대로 반환하는 의존성 주입을 위해 생성한 인스턴스입니다. 
이렇게 테스트용도로 사용하는 어떤 `stub`은 가짜 객체를 만들어서 테스트하는 클래스에 의존성을 주입해주는 것 
이거를 `수동 DI`라고 이야기하기도 합니다.  

Dependency Injection 이라는 것은 두 개의 의존 관계가 있는 오브젝트를 제 3의 존재, 
`Assembler`가 런타임 시에 그 관계를 맺어주는 것, 의존성 주입을 통해서 관계를 맺어주는 것, 
이게 Dependency Injection 인데 그 원리가 테스트 코드에도 적용되었습니다.  
  
Spring Container 대신에 우리가 만든 테스트 코드가 `Assembler`역할을 해준겁니다. 
  
이제 전체 로직을 테스트해보겠습니다.  
```java
@Test
@DisplayName("이름이 없는 경우 500 에러가 발생한다.")
void helloApiWithoutName(){
    // 스프링 부트가 동작해야한다.
    // http localhost:8080/hello?name=spring
    TestRestTemplate rest = new TestRestTemplate();
    ResponseEntity<String> response =
            rest.getForEntity("http://localhost:8080/hello?name=", String.class);
    // 응답 3가지 검증
    // status, header, Hello Sring
    assertThat(response.getStatusCode()).isEqualByComparingTo(HttpStatus.INTERNAL_SERVER_ERROR);
}
```
    
