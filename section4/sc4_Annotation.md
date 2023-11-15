# 애노테이션 매핑 정보 
서블릿 컨테이너 대신에 스프링 디스패처 서블릿을 등록하고  
디스패처 서블릿이 위임할 클래스 정보를 주는 스프링 컨테이너인  
`GenericWepApplicationContext`를 생성자 매개변수로 전달했습니다.  
  
```java
package toby.hello;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Objects;

@RestController
public class HelloController {

    private final HelloService helloService;

    public HelloController(HelloService helloService) {
        this.helloService = helloService;
    }

    @GetMapping("/hello")
    public String hello(String name) {

        // 방법 1
        if (name == null) {
            throw new IllegalArgumentException("값이 비어있습니다.");
        }
        // 방법 2 - null -> exception
        return helloService.sayHello(Objects.requireNonNull(name));
    }
}
```  
  
현재 스프링 컨택스트에는 `HelloController`의 클래스 정보가 들어있습니다.  
디스패처 서블릿은 스프링 컨택스트에서 웹 요청을 처리할 클래스와 매서드를 찾아서 매핑을 대신 해줍니다.  
다양한 매핑 정보를 디스패처 서블릿에 전달하는 방법이 있지만  
웹 요청 기능을 처리할 클래스에 애노테이션을 작성하는 방법을 많이 사용합니다   
  
클래스 레벨에 `@Controller`나 `@RequestMappin`이 들어가야합니다.  
(스프링 3.0 버전부터는 `@Controller`만 가능합니다. )  
디스패처 서블릿이 스프링 컨택스트에 등록된 빈을 조회할 때 클래스 레벨에 붙어있는 어노테이션을 보고 판단하기 때문입니다.  
메소드 레벨에서 찾는다고 하면 수천, 수만개가 되는 메서드에 있는  
애노테이션을 찾는 일은 매우 번거롭고 , 느리기 때문입니다.  
  
디스패처 서블릿이 어떤 정보를 필요로 하는데 생략이 가능한 경우  
스프링 MVC에 다양한 관례들이 많이 있습니다.  
  
**_웹 요청 처리를 하는 메소드를 보고 응답을 예상할 수 있어야합니다._**

스프링 컨테이너 및 웹 서버 실행 코드  
```java
public static void main(String[] args) {
    GenericWebApplicationContext applicationContext = new GenericWebApplicationContext();
    applicationContext.registerBean(HelloController.class);
    applicationContext.registerBean(SimpleHelloService.class);
    applicationContext.refresh();

    ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
    WebServer webServer = webServerFactory.getWebServer(
            servletContext -> {
                servletContext.addServlet("dispatcherServlet",
                         new DispatcherServlet(applicationContext)
                        ).addMapping("/*");
            }
    );
    webServer.start();
}
```  
현재 실행 코드를 보면 크게 두 파트로 나누어 집니다.  
+ `applicationContext.refresh()`  
+ `webServer.start()`  
  
앞 부분에서는 스프링 컨테이너를 생성하고 , 빈으로 사용할 클래스를 등록하고 초기화를 했습니다.  
이후에는 그렇게 만들어진 스프링 컨테이너를 활용해서 웹 서버를 생성하고 서블릿을 등록할 때  
디스패처 서블릿을 대신 등록하고 스프링 컨택스트를 전달해서 서버를 실행했습니다.  
  
이 코드를 하나로 동작해보려고합니다.  
```text
스프링 부트와 같이 컨테이너를 같이 띄우기 시작하게 된 건, 
아마도 클라우드의 영향으로 동시에 여러개의 서버를 사용하고, 
필요에 따라 이를 늘리기도 하는 등의 최근 배포 방식이 시작되면서인 듯합니다. 
마이크로서비스를 도입하거나, 그렇지 않더라도 작은 사양의 서버를 부하에 따라 
늘리거나 줄이거나 하는 상황에서 SE가 일일히 서버를 관리하기 힘들게 됐죠.

그래서 점차 서블릿 컨테이너당 웹 모듈 하나만 배포하는 것, 
그리고 재배포시 아예 서블릿 컨테이너를 재시작하는 방식도 늘기 시작했고요. 
아마 그런 변화가 일어나던 시점에 
스프링 부트 혹은 그에 영향을 준 다른 웹 기술 등이 등장을 했던 것 같습니다.
- 토비
```   
그러면 어느 시점에 컨테이너를 초기화하는 작업을 넣어야하는지 알아야합니다.  
스프링 컨테이너 초기화 작업은 `refresh()`라는 메서드에서 다 일어납니다.  
  
그 안에 코드를 살펴보면 전형적인 템플릿 메소드로 만들어 져있다는걸 알수 있습니다.
```java
// 컨텍스트 하위 클래스에서 빈 팩토리의 후처리를 허용합니다.
// 컨텍스트의 하위 클래스에서 빈 팩토리에 대한 추가 처리를 수행할 수 있습니다.
postProcessBeanFactory(beanFactory);

// Spring 컨텍스트 빈의 후처리 단계를 시작합니다.
// 컨텍스트에 등록된 빈 팩토리 프로세서를 호출합니다.
StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
invokeBeanFactoryPostProcessors(beanFactory);
// 등록된 빈 팩토리 프로세서가 여기에서 호출됩니다.

// 빈 생성을 가로채는 빈 프로세서를 등록합니다.
// 이러한 프로세서는 빈 생성을 가로채고 필요에 따라 수정할 수 있습니다.
registerBeanPostProcessors(beanFactory);
beanPostProcess.end();

// 이 컨텍스트의 메시지 소스를 초기화합니다.
// 이 컨텍스트에서 메시지를 해석하기 위한 메시지 소스를 설정합니다.
initMessageSource();

// 이 컨텍스트의 이벤트 다중 전송기를 초기화합니다.
// 애플리케이션 이벤트를 처리하기 위한 이벤트 다중 전송기를 설정합니다.
initApplicationEventMulticaster();

// 특정 컨텍스트 하위 클래스에서 다른 특수화된 빈을 초기화합니다.
// 특정 컨텍스트 하위 클래스에 대한 추가 초기화를 수행합니다.
onRefresh();

// 리스너 빈을 확인하고 등록합니다.
// 컨텍스트 내에서 빈으로 구성된 리스너를 등록합니다.
registerListeners();

// 지연 초기화되지 않은 모든 나머지 싱글톤을 인스턴스화합니다.
// 나머지 지연 초기화되지 않은 싱글톤을 초기화하고 인스턴스화합니다.
finishBeanFactoryInitialization(beanFactory);

// 마지막 단계: 해당 이벤트를 게시합니다.
// 컨텍스트 새로 고침이 완료되었음을 나타내는 해당 이벤트를 게시합니다.
finishRefresh();
```  
> 템플릿 메소드를 활용하다보면 그 안에 여러 개의 Hook 메소드를 주입하기도 합니다.  
그래서 템플릿 메소드 안에서 일정한 순서에 의해서 작업들이 호출되는데  
그중 서브클래스에서 확장하는 방법을 통해 특정 시점에 어떤 작업을 수행하게 해서  
기능을 유연하게 확장하도록 만드는 기능입니다.  
  
그 `hook`메소드 이름이 `onRefresh`입니다.  
refresh가 실행되면 스프링 컨테이너를 초기화하는 중 부가적으로 어떤 작업이 수행할 필요가 있다면  
이걸 사용하라고 만들어 놓은 메소드 입니다.  

템플릿 메소드 패턴은 상속을 통해서 기능을 확장하도록 만들고  
상속하기 때문에 부모 클래스에 대한 정보로 전부 알고 있습니다.  
  
`GenericWebApplicationContext` 클래스를 상속해 `onRefresh`를 오버라이딩 하면 됩니다.  
여러번 재사용하는 클래스가 아니기 때문에 클래스의 이름을 직접 정의하지 않고 상속하는 방법인  
익명 클래스를 사용해서 오버라이딩을 하도록 하겠습니다.  
```java
GenericWebApplicationContext applicationContext = new GenericWebApplicationContext(){
    @Override
    protected void onRefresh() {
        super.onRefresh();
        ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
        WebServer webServer = webServerFactory.getWebServer(
                servletContext -> {
                    servletContext.addServlet("dispatcherServlet",
                            new DispatcherServlet(this)
                    ).addMapping("/*");
                }
        );
        webServer.start();
    }};
applicationContext.registerBean(HelloController.class);
applicationContext.registerBean(SimpleHelloService.class);
applicationContext.refresh();
```
`refresh()`가 동작하는 중간에 실행이 되는 hook 메소드이기 때문에  
자기 자신을 참조해서 실행할 수 있게 `this`를 넣어서 디스패처 서블릿을 초기화하면 됩니다.

```java
public static void main(String[] args) {
    GenericWebApplicationContext applicationContext = new GenericWebApplicationContext(){
        @Override
        protected void onRefresh() {
            super.onRefresh();
            ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
            WebServer webServer = webServerFactory.getWebServer(
                    servletContext -> {
                        servletContext.addServlet("dispatcherServlet",
                                new DispatcherServlet(this)
                        ).addMapping("/*");
                    }
            );
            webServer.start();
        }};
    applicationContext.registerBean(HelloController.class);
    applicationContext.registerBean(SimpleHelloService.class);
    applicationContext.refresh();
}
```