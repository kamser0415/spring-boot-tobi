# 컨트롤러 매핑과 바인딩  
> 현재코드
```java
ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
WebServer webServer = webServerFactory.getWebServer(
        servletContext -> {
            servletContext.addServlet("frontController", new HttpServlet() {
                @Override
                protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                    String requestURI = req.getRequestURI();

                    if(requestURI.equals("/hello") && req.getMethod().equals(HttpMethod.GET.name())){
                        String name = req.getParameter("name");
                        resp.setStatus(HttpStatus.OK.value());
                        resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
                        PrintWriter writer = resp.getWriter();
                        writer.println("hello "+name);
                    } else if (requestURI.equals("/member")){
                        //기타 로직                                
                    } else {
                        // 등록된 기능이 없다면
                        resp.setStatus(HttpStatus.NOT_FOUND.value());
                    }
                }
            }).addMapping("/*");
        }
);
webServer.start();
```  
프론트 컨트롤러는 모든 요청을 다 받아서 공통적인 기능을 처리하고 요청 정보를 이용해서  
여러가지 종류의 작업을 매핑해주는 코드를 만들었습니다.  

공통로직과 매핑이 끝나고 나서 그 로직을 처리하는 부분을 살펴보면  
`GET /hello`로 들어오는 요청을 처리하는 로직은 굉장히 간단합니다.  
파라미터에서 `name`을 추출하고 그 다음 문자열에 추출한 값을 body에 작성하는게 전부죠.  
  
이 부분이 복잡한 데이터베이스 처리도 하고, api 호출도 하고 많은 비즈니스 로직이 들어온다면  
해당 컨트롤러는 유지보수하기가 어려울 겁니다.  
  
요 부분을 분리해보면 더 유지보수 하기 쉬울거라 생각합니다.  
이제 `/hello`라는 로직을 처리하는 코드를 만들긴 해야하지만  
굳이 새로 만들 필요는 없습니다 이미 예제로 만든 `HelloController`가 있기 때문입니다.  
  
현재는 스프링 부트를 사용하지 않기 때문에 애노테이션을 제거해봅니다.  
```java
//@RestController
public class HelloController {

//    @GetMapping("/hello")
    public String hello(String name) {
        return "Hello "+name;
    }
}
```  
해당 클래스는 hello라는 매소드 하나를 가지고 있고 String 타입의 파라미터를 받아서  
로직을 처리하고 결과를 `String`타입으로 반환하는 평범한 Java 클래스입니다.  
  
이제 프론트 컨트롤러가 이 작업을 위임해서 처리할 백단에 있는 핸들러 컨트롤러 혹은 커맨드라 생각하고,  
해당 클래스를 사용합니다. 매 요청마다 새로운 인스턴스를 만들 필요가 없습니다. 
한번 만들어놓고 계속 재사용해도 문제가 없는 코드입니다.  
  
그래서 서블릿을 초기화하는 부분에서 controller의 instance를 만들어서 변수에 저장하고,  
이 인스턴스를 백단에서 사용하면 됩니다.  
  
여기서 name을 추출하는 것은 프론트 컨트롤러가 해주면 더 좋을거 같습니다.  
이유는 지금 구조에서 프론트 컨트롤러가 필요한 파라미터만 helloController에게 전달만 하면    
`helloController`는 서블릿 기술인 `Request & Response`를 알지 못해도 동작할 수 있습니다.  
  
```java
servletContext -> {
    HelloController helloController = new HelloController();
    servletContext.addServlet("frontController", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            String requestURI = req.getRequestURI();

            if(requestURI.equals("/hello") && req.getMethod().equals(HttpMethod.GET.name())){
                String name = req.getParameter("name");

                String result = helloController.hello(name);

                resp.setStatus(HttpStatus.OK.value());
                resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
                
                PrintWriter writer = resp.getWriter();
                writer.println(result);
            } else if (requestURI.equals("/member")){
                //기타 로직                                
            } else {
                // 등록된 기능이 없다면
                resp.setStatus(HttpStatus.NOT_FOUND.value());
            }
        }
    }).addMapping("/*");
}
```  
코드를 다시 살펴보면 이번 로직은 프론트 컨트롤러 안에서 로직이 동작한게 아니라  
HelloController의 hello 라는 메소드에 작업을 위임하고 얘가 처리한 결과를 리턴받아서  
메세지 바디에 넣어 응답을 반환했습니다.  
  
다시 정리해보면,  
프론트 컨트롤러에서 요구되는 공통적인 코드들을 서블릿 컨택스트 안에 인스턴스를 생성하고, 재사용 합니다.  
그리고 실제 클라이언트 요청을 받아서 처리하는 코드를 동작시키는 동안에 두 가지 중요한 작업이 수행합니다.  

_**매핑과 바인딩**_ 입니다.  

**매핑**은 웹 요청에 들어있는 정보를 활용해서 어떤 로직을 수행하는 코드를 호출할 것인가 결정하는 작업이고  
위 코드같은 경우에는 `GET /hello` 요청이 들어오면 `helloController`의 hello 메소드를 호출해서  
로직을 수행하고 그 결과를 리턴하게 합니다.
  
`helloController`는 웹 요청이나 웹 응답 객체를 직접 사용하지 않았습니다.  
평범한 자바 클래스에 웹 요청이나 응답 객체를 사용하면 웹 기술인 `Request`,`Response`의 메소드를  
내부 로직에서 파라미터를 꺼내고, 패스를 꺼내고, 상태값을 넣어주는 코드들이 추가됩니다.  
그 부분을 분리하는 거죠.
  
물론 필요하다면 사용할 수는 있습니다. 하지만 일반적으로 사용하지 않습니다.  
  
컨트롤러 로직을 작성할 때 웹 기술의 메소드나 정보를 알지 못해도  
웹 기술과 관련없는 순수 자바코드로 비즈니스 로직을 작성하고 반환만 하면 됩니다.  

바인딩이라는건 `method`를 호출할 때 여기 인자 값으로 넘겨주는 이 작업
```java
String name = req.getParameter("name");
// 바인딩
String result = helloController.hello(name);
```  
이 작업을 `binding` 이라고 합니다.  
클라이언트에서 서버에 전송한 웹 요청 정보를 컨트롤러내 메소드가 필요로 하는 형태  
즉, 기본 타입이나, DTO, JavaBean 형태의 오브젝트로 만들어서  
필요하다면 그 안에 데이터를 집어넣어서 이걸 처리하는 컨트롤러 메소드의 파라미터로 넘겨주는 작업들을  
바인딩 이라고 합니다.  
  
사실 웹 MVC에서 얘기하는 바인딩은 이것보다 훨씬 복잡합니다.  
알아야 할 것도 많고, 웹 MVC를 사용한다면 바인딩이 어떻게 일어나는지에 대해서  
많은 학습과 지식이 필요합니다.  
  
하지만, 기본적인 원리는 다 이렇습니다.  
우리가 웹 요청이 어떻게 생겼는지 알고 직접적으로 액세스하는 프론트 컨트롤러와 같은 코드에서  
웹 클라이언트가 요청하는 로직을 처리하는 오브젝트에서 평범한 데이터 타입으로 전환해서 넘겨주는 작업  
이정도가 바인딩이라고 생각하고 넘어가면 될 것 같습니다.  
  
지금까지 우리가 작업한 코드는 스프링을 전혀 사용하지 않고  
서블릿 컨테이너, 서블릿 기술만 사용해서 프론트 컨트롤러까지 만들어 봤습니다.  
이 과정 모두 서블릿 컨테이너를 따로 설치하거나 배포하는 작업을 하지 않고  
모든 과정이 메인 메소드 하나에서 일어난 일입니다.  
  
이렇게만 만들어도 독립적으로 실행 가능하고 설치등이 필요없는 코드는 만들어 졌지만,  
여기에 스프링 애플리케이션을 `stand alone`형식으로 서블릿 컨테이너와 관련된 코드들을  
애플리케이션 개발하는 핵심부와 완전히 분리된 형태로 동작하도록 작업할 겁니다.  

