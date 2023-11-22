# 자바 코드 구성정보  
  
+ 자바 코드로 구성 정보를 등록하기 전 코드  
```java
public class HelloApplication {
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
}
```  
스프링 컨테이너에 우리가 작성한 코드를 등록해야합니다.  
그리고 작성한 코드는 스프링 컨테이너에서 빈이라는 오브젝트로 관리를 하죠.  
클래스간 의존 관계를 xml파일로 만들어 스프링 컨테이너에게 전달하는 방법도 있고,  
지금은 스프링 컨테이너가 내부에서 스캔을 통해 의존 관계를 주입하는 방법을 사용합니다.  
  
단순한 의존 관계뿐만 아니라 어느 시점에 의존성을 주입해 줄지 정보를 줄수도 있습니다.  
스프링 컨테이너가 사용하는 다양한 구성 정보에 전달을 해야하죠.  
  
다양한 구성 정보를 전달하는 방법중에서 `Factory Method`를 사용하는 방법을 소개합니다.  

`FactoryMethod`는 인스턴스를 생성하는 메소드를 의미합니다.   
사용하는 이유는 생성자 함수와 다르게 의도나 목적을 나타내고 있기 때문이죠.  

클래스 정보만 전달하면 스프링 컨테이너가 알아서 빈을 생성하고 관리하는데  
팩토리 메소드를 통해서 개발자가 직접 필요한 의존성을 주입하고,  
생성자 함수로 인스턴스를 반환하는 코드를 작성하는 이유는  
빈 오브젝트를 만들고 초기화하는 작업이 상당히 복잡한 경우가 있습니다.  
  
이 복잡한 설정 정보를 나열하는 대신에 자바코드로 만들면 간결하고 이해하기가 쉽습니다.  

```java
@Configuration
public class HelloApplication {
    @Bean
    public HelloController helloController(HelloService helloService) {
        return new HelloController(helloService);
    }
    @Bean
    public HelloService helloService() {
        return new SimpleHelloService();
    }
    public static void main(String[] args) {
        AnnotationConfigServletWebApplicationContext applicationContext = new AnnotationConfigServletWebApplicationContext(){
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
            }
        };
        applicationContext.register(HelloApplication.class);
        applicationContext.refresh();
    }
}
```  
스프링 컨테이너에 빈 정보를 전달할 때 클래스 레벨에 애노테이션을 붙여서 목적을 나타냅니다.  
`@Controller`가 클래스 레벨에 붙은 경우 해당 클래스는 웹 요청을 처리하는 빈이라는 걸 알리는 거죠.  
`@Configuration`가 클래스 레벨에 붙으면 스프링 컨테이너에게  
빈 오브젝트를 가진 팩토리 메소드가 있는 클래스라는 걸 인지시킵니다.  
  
그리고 자바 코드를 통해서 구성정보로 활용하는 기능을 추가한 스프링 컨테이너를 사용해야합니다.  
`AnnotationConfigServletWebApplicationContext`  
```java
@Configuration-annotated classes, but also plain @Component classes 
        and JSR-330 compliant classes using javax.inject annotations.
```
`@Bean`이 붙은 팩토리 메소드를 가지고 스프링 컨테이너에게  인지하기 위한  
어노테이션 `@Configuration`가 붙은 클래스를 스프링 컨테이너에 전달합니다.  
  
사실 중요한건 `@Configuration`가 붙은 클래스가  
`AnnotationConfig`를 이용하는 애플리케이션 컨텍스트에 처음 등록됩니다.  
  
왜냐하면 `@Configuration`이 붙은 클래스는 `BeanFactoryMethod`를 가지는 것 이상으로  
전체 애플리케이션을 구성하는데 필요한 정보를 많이 넣을 수 있기 때문입니다.   

### 정리  
`GenericWebApplicationContext`는 빈 오브젝트로 사용할 클래스를 직접 전달해야한다.  
자바 코드로 등록하는 것과, 자바 코드로 매핑 정보를 전달하는 건 차이가 있다.  
여기서 자바 코드로 등록하면서 매핑 정보까지 전달하는 애노테이션이 `@Controller`다.  
  
@Controller,@Configuration,@Component를 사용하려면  
`GenericWebApplicationContext`를 사용할 수 없고 하위 클래스인  
`AnnotationConfigServletWebApplicationContext`을 사용해야합니다.



