# `@ComponentScan` 사용  
이전까지는 스프링 컨테이너에 빈을 등록하는 방법으로  
클래스 정보를 레지스터 빈에 `springContainer.register(class)`에 클래스를 넘겨주거나 혹은 
팩토리 메소드를 만들어 클래스 레벨에 `@Configuration`을 추가하여 그 클래스 정보를 넘겨주면 
스프링 컨테이너가 클래스 애노테이션을 읽고 `@Bean`이 붙은 팩토리 매소드를 실행하여 빈으로 등록했습니다.  
  
스프링 컨테이너에는 컴포넌트 스캐너가 있습니다.  
컴포넌트 스캐너는 `@Component` 애노테이션이 붙은 모든 클래스를 찾아서 빈으로 등록합니다.  
스프링 부트에 필요한 빈으로 인식해 달라는 표시로 클래스 레벨에 어노테이션을 선언합니다.  
  
스프링 컨테이너에 등록하는 첫 번째 클래스가 중요합니다.  
컨테이너를 구성하는데 필요한 여러 가지 정보와 힌트를 넣을 수 있습니다.  
그 중 지금 `@Component`가 붙은 클래스를 찾아서 빈으로 등록을 요청하는 애노테이션인 
`@ComponentScan`을 붙여서 레지스터에 전달합니다.  
```java
@Configuration
@ComponentScan
public class HelloApplication {
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
`@ComponentScan`이 붙은 클래스가 있는 패키지부터 시작하여 하위 패키지를 스캔하여 `@Component` 애노테이션이 붙은 
모든 클래스를 빈으로 등록을 합니다.  
빈으로 등록할 때 필요하다면 의존 오브젝트를 찾아서 생성자를 호출할 때 파라미터로 넘겨줍니다.  

이렇게 처음 등록하는 구성 정보 클래스가 `@ComponentScan`이 있으면, 매번 구성 정보 클래스를 다시 등록할 필요가 없습니다.  
빈으로 등록할 클래스에 `@Component`를 추가하면 됩니다.  

장점은 매우 편리하게 빈으로 등록할 수 있고, 단점으로는 빈으로 추가되는 클래스가 굉장히 많아지면 이 애플리케이션을 실행했을 때 정확하게 어떤 클래스가 빈으로 등록되었는지 확인해보려면 번거로울 수 있습니다.  
내가 클래스들을 다 찾아서 컴포넌트가 붙은 클래스들이 이런게 있으니 이번 어플리케이션을 실행하면 이런 클래스의 빈이 만들어져 사용되는지 체크해봐야합니다.  
`-` 토비  
강사님 말씀의 뜻은 내가 빈으로 설정 정보나 대체해서 사용할 빈을 Component로 등록하면 어떤게 사용되는지 확인해야하는 경우가 발생할 수 있다.  
라고 이해가 되었습니다.  
  
그리고 패키지 구성을 잘하고, 모듈을 잘 나누어서 개발하면 어렵지 않게 파악할 수 있어서 단점이 부각되지 않습니다.  
  
`@Component` 애노테이션을 메타 애노테이션으로 붙여도 컴포넌트 스캔 대상이 됩니다.  
메타 애노테이션이란 애노테이션 위에 붙은 애노테이션이라는 의미입니다.  
```java
@Retention(RetentionPolicy.RUNTIME) // 생명주기를 결정
@Target(ElementType.TYPE) // 애노테이션의 위치를 결정합니다.
@Component
public @interface MyComponent {}
```    
메타 애노테이션을 사용하는 이유는 
+ 명시적으로 의도나 목적을 나타냅니다.  
```java
@Controller // 웹 요청을 처리하는 빈을 나타냅니다.
@Service // 비즈니스 로직을 담당하는 빈을 나타냅니다.
@Repository // 데이터 엑세스를 담당하는 빈을 나타냅니다.
```  
다시 정리하면, @Controller 애노테이션은 디스패처 서블릿이 인식합니다.  
디스패처 서블릿이 스프링 컨테이너를 내부에 가지고 있고 웹 요청이 들어오면 클래스 레벨에 붙은 웹 요청 애노테이션을 찾아서 그 이하 메소드에 웹 요청을 처리하는 메소드도 같이 읽어서 처리합니다.  

웹 요청 어노테이션과 빈으로 등록하는 애노테이션은 별개입니다.  
스프링 컨테이너에 등록한 빈중에서 웹 요청 애노테이션이 있는 클래스와 내부 메소드를 읽어서 웹 요청을 매핑하는 것이 디스패처 서블릿의 역할이고  
스프링 컨테이너에 빈을 등록하는 방식을 자바 코드로 등록할 때 필요한게 기존보다 확장된 스프링 컨테이너인  
`AnnotationConfigServletWebApplicationContext`을 사용하는 겁니다.  
  
# Bean의 생명주기 메소드
```java
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
```  
스프링 컨테이너에 빈을 등록하는 방법은 간편한 방법인 `@CompnentScan`을 사용하는 것과 `@Configuration`과 `@Bean`을 활용하여 Factory 메소드를 이용하는 Java 코드 구성 정보를 클래스에 작성해서 등록하는 방법이 있습니다.  
지금 스프링 컨테이너에게 오브젝트 생성을 위임하여 만들어진 빈이 있고 그리고 우리가 직접 코드로 작성하여 오브젝트를 사용하는 `톰캣`과 `디스패처 서블릿`이 있습니다.  
이 두 가지는 애플리케이션의 기능을 제공하기 위해 만들어야하는 오브젝트는 아닙니다.  
하지만 없을 경우 애플리케이션을 시작할 수도 없습니다.  
`StandAlone`애플리케이션을 만들기 위해서 보이지 않는 곳에 인스턴스를 생성후 사용했습니다.  
  
이 두 가지 오브젝트도 빈으로 등록하면 굉장히 유연한 구성이 가능해집니다.  
  
이 두가지 오브젝트를 빈으로 등록하는 방법으로 `Factory`매소드를 활용하는 방법을 사용합니다.  
```java
@Configuration
@ComponentScan
public class HelloApplication {

    @Bean
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }


    public static void main(String[] args) {
        AnnotationConfigServletWebApplicationContext applicationContext = new AnnotationConfigServletWebApplicationContext(){
            @Override
            protected void onRefresh() {
                super.onRefresh();
                ServletWebServerFactory serverFactory = this.getBean(ServletWebServerFactory.class);
                DispatcherServlet servlet = this.getBean(DispatcherServlet.class);
                WebServer webServer = serverFactory.getWebServer(servletContext -> {
                    servletContext.addServlet("dispatcherServlet", servlet).addMapping("/*");
                });

                webServer.start();
            }
        };
        applicationContext.register(HelloApplication.class);
        applicationContext.refresh();
    }
}
```

```java
@Bean
public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
}
```
반환 타입을 `ServletWebServerFatory`으로 하는 이유는 스프링 부트가 추상화한 웹서버 팩토리를 사용하면 `Tomcat`이 아닌 다른 서블릿 컨테이너도 사용할 수 있습니다.  
두 번째 `DispatcherServlet`을 빈으로 등록합니다. 디스패처 서블릿은 자신이 이용할 컨트롤러를 찾아야하기 때문에 스프링 컨테이너를 생성자를 통해서 넘겨줘야 합니다.  
그런데 팩토리 메소드에 애플리케이션 컨택스트를 어떻게 전달해야할 까요?  
  
이전에 오브젝트로 생성할 때는 자기 자신을 생성자에 넘겨주었는데 선언부가 밖에 있어서 전달하기가 어렵습니다.  
스프링 빈으로 등록된 톰캣 웹서버와 디스패처 서블릿을 `getBean()`으로 가져옵니다.  
  
```java
ServletWebServerFactory serverFactory = this.getBean(ServletWebServerFactory.class);
DispatcherServlet servlet = this.getBean(DispatcherServlet.class);
```  
팩토리 메소드에서는 생성자에 스프링 컨테이너를 전달하지 않고 생성했습니다.  
그래서 이미 생성된 디스패처 서블릿에 초기화를 합니다.  
```java
servlet.setApplicationContext(this);
```  
이 코드를 지우고 실행해도 정상 동작이 가능합니다.  
`ApplicationWebServletContext`를 주입하지 않았는데 동작이 되었습니다.  
스프링 컨테이너가 오브젝트를 생성할 때 해당 디스패처 서블릿은 애플리케이션 컨택스트가 필요하다고 알고 빈으로 등록할 때 주입해줍니다.  
  
이 개념을 이해하려면 빈의 생명주기 메소드라는 개념을 알아야합니다.  
해당 디스패처 서블릿의 상속과 구현정보를 확인하면 `ApplicationContextAware`를 구현하고 있습니다.  
```java
public interface ApplicationContextAware extends Aware {
    void setApplicationContext(ApplicationContext applicationContext) 
            throws BeansException;
}
```  
빈을 컨테이너에 등록하고 관리하는 중에 컨테이너가 관리하는 오브젝트를 빈에 주입해주는 라이프 사이클 메소드라는 걸 알 수 있습니다.  
해당 인터페이스를 구현한 빈으로 등록이 되면 그게 팩토리 메소드에서 만들어지든 설정 파일을 만들어지든 상관없습니다.  
컨테이너에 등록이 된 후에 이런 종류의 인터페이스를 구현하고 있으면 스프링 컨테이너가 이 인터페이스의 `set`메소드를 사용하여 주입을 합니다.  
  
그래서 명시적으로 생성자 또는 Setter 메소드를 직접 호출하지 않아도 ApplicationContext를 가지고 있게 됩니다.  
  
코드로 직접 적용해보겠습니다.
```java
@RestController
public class HelloController implements ApplicationContextAware {
    private ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        System.out.println(applicationContext);
    }
}
```  
  
이거는 스프링 컨테이너가 초기화 되는 시점에 이 작업이 일어납니다.  
그래서 서버를 띄우기만 해도 이렇게 실행되는 걸 확인할 수 있어야 됩니다.  
```java
toby.hello.HelloApplication$1@43a25848, started on Wed Nov 22 20:22:43 KST 2023
```  
  
내부에서 기능을 수행할 때 사용하니까 변수에 저장하고 사용하면 됩니다.  
`final`은 사용할 수 없습니다. 파이널로 정의하면 자바 필드는 생성자가 완료되는 시점까지는 초기화가 되어야합니다.  
해당 HelloController는 생성자를 통해서 인스턴스를 생성하고 이후에 호출되는 메소드이기 때문에 파이널로 만들 숫 없습니다.  
  
스프링 컨테이너도 빈으로 등록후 사용하기때문에 이렇게 주입이 가능합니다.  
최신 방법으로 생성자 주입을 통해서 주입을 받을 수 도 있습니다.  
```java
public HelloController(HelloService helloService,ApplicationContext applicationContext) {
    this.helloService = helloService;
    this.applicationContext = applicationContext;

```  
이 방법을 통해서 팩토리 메소드로 생성된 빈 오브젝트도 생성자를 통해서 주입하는 방법이 아니더라도  
내부 메소드를 통해서 주입할 수 있다는 걸 알았습니다.

# SpringBootApplication  
지금까지 작성한 코드를 메소드로 추출하여 리팩토링 해보겠습니다.  
`main`메소드에 있던 로직을 static method로 변경하고 별도의 클래스로 분리하겠습니다.
```java
public class MySpringApplication {
    public static void run(Class<HelloApplication> applicationClass,String... args) {
        AnnotationConfigServletWebApplicationContext applicationContext = new AnnotationConfigServletWebApplicationContext(){
            @Override
            protected void onRefresh() {
                super.onRefresh();
                ServletWebServerFactory serverFactory = this.getBean(ServletWebServerFactory.class);
                DispatcherServlet servlet = this.getBean(DispatcherServlet.class);
                WebServer webServer = serverFactory.getWebServer(servletContext -> {
                    servletContext.addServlet("dispatcherServlet", servlet).addMapping("/*");
                });

                webServer.start();
            }
        };
        applicationContext.register(applicationClass);
        applicationContext.refresh();
    }
}
```
```java
@Configuration
@ComponentScan
public class HelloApplication {

    @Bean
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }

    public static void main(String[] args) {
        MySpringApplication.run(HelloApplication.class,args);
    }
}
```
리팩토링된 코드를 정리해보겠습니다.  
매개변수에 구성 정보 클래스를 받아서 재사용 할 수 있도록 했습니다.  
넘겨주는 클래스는 중요한게 있습니다.  
+ `@Configuration`이 붙은 클래스  
+ `@ComponentScan`이 붙은 클래스  
  
스프링 컨테이너에게 애플리케이션 구성을 어떻게 할지 알려주는 정보를 가진 클래스여야합니다.  
그리고 메인 메소드에서 실행하기 때문에 CommendLine에서 argument가 넘어올 수 있으니 파라미터로 받겠습니다.   
다른 메인 클래스에서도 서블릿 컨테이너를 코드에서 자동으로 띄워주면서 스프링 컨테이너에 필요한 정보만 전달하면 기본적인 기능을 수행할 수 있도록 만드는 기초 작업들, 스프링 컨테이너 준비 작업들을 해주는데 이 메소드를 재사용할 수 있습니다.  
이 클래스를 `MySpringApplication`이라고 만듭니다.  
  
다른 클래스에서도 접근할 수 있도록 `public static`으로 지정합니다.  
  
## MySpringApplication 정리  
```java
public static void main(String[] args) {
    SpringApplication.run(HelloApplication.class,args);
}
```
정리하면 스프링어플리케이션의 run이라는 메소드에 스프링 컨택스트에서 사용할 구성 정보 클래스를 매개변수로 전달하면 
내부에서 해당 정보를 가지고 스프링 컨테이너에 오브젝트를 만들어 빈으로 관리하고, 스프링 부트가 지정해 놓은 설정으로 톰캣과 디스패처 서블릿이라는 프론트 컨트롤러를 실행시켜줍니다.  
  
실제 스프링 부트를 생성하면 톰캣을 빈으로 등록하거나 ,디스패처 서블릿을 빈으로 등록하지 않습니다.  
그런데 지금 코드를 보면 내부에서 직접 팩토리 매소드를 사용해서 빈을 등록하는 부분이 있습니다.  

`SpringApplication.run(HelloApplication.class,args);`  
클래스를 빈으로 사용하겠다고 @Configuration 과 @ComponontScan을 활용하여 구성 정보를 스프링 컨택스트에 전달하면 스프링 컨테이너가 내부에서 초기화하여 사용합니다.