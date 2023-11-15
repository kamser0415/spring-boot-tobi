# DispatcherServlet  
> 이전 코드  
```java
public interface HelloService {
    String sayHello(String name);
}
```
```java
  public static void main(String[] args) {
        GenericApplicationContext applicationContext = new GenericApplicationContext();
        applicationContext.registerBean(HelloController.class);
        applicationContext.registerBean(SimpleHelloService.class);
        applicationContext.refresh();

        ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
        WebServer webServer = webServerFactory.getWebServer(
                servletContext -> {
                    servletContext.addServlet("frontController", new HttpServlet() {
                        @Override
                        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                            String requestURI = req.getRequestURI();
                            if(requestURI.equals("/hello") && req.getMethod().equals(HttpMethod.GET.name())){
                                String name = req.getParameter("name");
                                HelloController helloController = applicationContext.getBean(HelloController.class);
                                String result = helloController.hello(name);
                                resp.setContentType(MediaType.TEXT_PLAIN_VALUE);
                                PrintWriter writer = resp.getWriter();
                                writer.println(result);
                            } 
                        }
                    }).addMapping("/*");
                }
        );
        webServer.start();
    }
   ```  
스프링 컨테이너에 빈에 클래스를 등록할 때 인터페이스는 등록할 수 없습니다.  
빈 클래스로 인스턴스를 생성해서 관리하기 때문에 구현 클래스를 넣어야합니다.  
  
스프링 컨테이너가 클래스에 필요한 의존관계를 주입하기 위해서는 정보가 필요합니다.  
예전에는 명시적으로 xml 파일을 통해서 의존 클래스의 정보를 넣어 전달했습니다.  
  
자바 코드로 빈을 등록하는 경우에는 xml 파일이 아니라 스프링 컨테이너 내부에서  
의존하는 클래스의 빈 정보를 찾아 주입을 하고, 로딩만 되어있고 빈이 아닌 경우에는 빈으로 만들어 주입합니다.  
  
## 서블릿  
스프링 컨테이너를 생성하고, 필요한 클래스도 빈으로 등록하여 주입받았습니다.  
스프링 부트에서는 몇 줄의 코드만 추가를 하면 많은 작업이 개발자가 신경을 쓰지 않아도 잘 돌아가는 코드로 가야하는데  
오히려 이전보다 코드가 복잡해지고 있습니다.  
  
지금은 프론트 컨트롤러라는 하나의 서블릿을 직접 만들어 사용하고 있습니다.  
하지만, 스프링 부트는 컨테이너 리스로 서블릿 컨테이너를 신경쓰지 않고 싶습니다.  
  
코드를 보면 애플리케이션 로직과 긴밀하게 연결되어있는게 서블릿 코드 안에 등장합니다.  
+ `Mapping`  

매핑은 웹 요청 기술을 가지고 요청에 맞는 컨트롤러 메서드를 찾아 연결해주는 작업입니다.  
여기 코드에서는 웹 요청의 URL의 PATH 정보, HTTP 메소드를 가지고 필터링을 해서  
스프링 컨테이너가 가지고 있는 `HelloController` 클래스 타입의 오브젝트의 `hello()`라는 메소드가   
담당하게 한다, 이걸 결정해주는 작업을 하드코딩되어 있습니다.  
+ `Binding`  

두 번째에는 요청 파라미터의 URL의 쿼리스트링으로 넘어온 파라미터 값을 추출해서  
`hello()`메소드의 파라미터로 넘겨주는 작업을 하고 있습니다.  
웹 기술이 컨트롤러나 비즈니스 로직까지 전달되지 않게 하기 위해서  
바인딩이라는 작업을 하드코딩으로 하고 있습니다.  
  
매번 서블릿과 관련된 코드를 다 넣을 수 없습니다.  
  
직접 만들었던 서블릿을 제거하고 스프링이 제공하는 `DispatcherServlet`을 사용합니다.  
```java
public static void main(String[] args) {
    GenericApplicationContext applicationContext = new GenericApplicationContext();
    applicationContext.registerBean(HelloController.class);
    applicationContext.registerBean(SimpleHelloService.class);
    applicationContext.refresh();

    ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
    WebServer webServer = webServerFactory.getWebServer(
            servletContext -> {
                servletContext.addServlet("dispatcherServlet",
                         new DispatcherServlet()
                        ).addMapping("/*");
            }
    );
    webServer.start();
}
```  
  
지금까지 했던 매핑과 바인딩외 다양한 기능을 수행해주는 클래스입니다.  
웹 요청 정보가 매핑에 일치하면 우리가 만들었던 서블릿은 스프링 컨테이너에서  
필요한 빈을 꺼내서 사용했습니다.  
  
`DispatcherServlert`은 스프링이 제공하는 서블릿이니까 매핑이나 바인딩은 알아서 해줍니다.  
그 후에 필요한 빈의 정보를 스프링 컨테이너에서 가져와 실행을 해야합니다.  
`DispatcherServlet`이 스프링 컨테이너를 알고 있어야 가능할 거 같습니다.  
  
우리가 만들고, 정보를 넣은 `ApplicationContext`를 전달합니다.  
```java
 servletContext.addServlet("dispatcherServlet",
         new DispatcherServlet(applicationContext)).addMapping("/*");
```  

그러나 컴파일 에러가 발생합니다.  
`DispatcherServlet`은 웹 환경에 적합한 스프링 컨테이너 타입으로 전달해야합니다.  
`GenericWebApplicationContext`라는 걸 이용해서 전달합니다.  
  
#### 지금까지 정리  
직접 만든 프론트 컨트롤러 서블릿은 매핑과 바인딩을 직접해야했습니다.  
그러면 서블릿 컨테이너 기술을 몰라도 동작하는 컨테이너 리스인 스프링 부트와는 다릅니다.  
스프링에서는 디스패처 서블릿을 대신 넣어주면 됩니다.  
디스패처 서블릿은 매핑과 바인딩을 하고, 매핑된 클래스에게 요청을 위임합니다.  
 
테스트를 해보면 동작하지 않습니다.  
디스패처 서블릿에게 어떤 웹 요청이 왔을때 어떤 클래스의 로직을 실행해달라고  
정보를 전달하지 않았습니다.  
  
다양한 방법이 있지만, 예전에는 xml을 사용하여 처리할 빈을 명시해 전달했지만  
매핑 정보를 서블릿 코드에 직접 넣는 방법보다 요청을 처리할 컨트롤러 클래스 안에다  
매핑 정보를 집어 넣는 방법을 사용합니다 .
