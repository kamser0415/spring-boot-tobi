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
스프링 부트는 스프링 컨테이너에서 사용할 빈 정보를 전달해주면 IOC와 DI를 통해서 오브젝트를 대신 관리해줍니다.  
그런데 지금 코드는 톰캣과 디스패처 서블릿을 팩토리매소드로 사용하고 있습니다.  
이 코드는 작성하지 않아도 원래 프로젝트에서는 동작한다는 것을 볼때에 스프링 부트가 클래스와 구성 정보를 가진 클래스를 개발자가 신경쓰지 않아도 전달한다는 것을 확인할 수 있습니다.

## `@Configuration`과 `@ComponentScan`에 대해서 주저리 
+ [토비 강사님의 링크](https://www.inflearn.com/questions/1082553/configuration-%EA%B3%BC-componentscan)  
  
지금 코드를 보면 스프링 컨테이너에 register 매소드를 실행할 때 매개변수로 구성 정보가 담긴 클래스를 전달 했습니다.  
`HelloApplication.class`는 `@Bean`이나 `@Component`가 없어도 `register()`를 활용하여 직접 빈으로 등록했습니다.  
그러면 `@Configuration`이 없어도 `@ComponentScan`이 동작하기 때문에 `@Configuration`이 왜 필요한지 궁금증이 생겼습니다.  
  
결론부터 말씀드리면
1. 애노테이션은 기능이 아니라 `주석`으로 생각해보자.
2. 멀티 모듈로 나누어지고 모듈의 루트가 아닐때 해당 클래스에 대한 `설명`을 나타낸다.  

자바의 애노테이션은 한국어로 `주석`을 의미합니다.
위키백과에서 주석은 아래와 같이 설명합니다.  
주석은 문서의 특정 지점 또는 기타 정보와 관련된 추가 정보입니다. 주석이나 설명이 포함된 메모일 수 있습니다. 주석은 때때로 책 페이지 여백에 표시됩니다.   
  
자바의 애노테이션도 기본적으로 주석이라 그 자체로 어떤 기능을 동작하지 않습니다.  
```java
public class Member {
    @Override
    public String toString() {
        return //..
    }
}
```  
`@Override`처럼 동작하는 코드에 추가되어 해당 메소드에 대한 추가 정보를 나타냅니다.  
해당 애노테이션이 없다고 하더라도 오버라이드를 못하는건 아닙니다. 이건 코드를 읽는 사람들을 위해서 상위에서 정의된 메소드를 오바라이드하는 메소드라는 일종의 코멘트를 붙인 겁니다.  
  
그런데 `@Configuration`,`@ComponentScan`과 같은 스프링 기술을에서 쓰는 애노테이션은 단순 각주 이상으로 런타임에 코드의 동작을 관여합니다. 
주로 프레임워크가 참고하는 정보를 나타내죠. 그 자체로 명령을 하는건 아니지만, 메타데이터로 프레임워크가 해당 애노테이션을 발견하면 추가적인 기능을 수행하게 만드는 것입니다. 
이렇게 런타임까지 유지되는 애노테이션은 프레임 워크의 참고할 수 있는 정보를 주는 용도로 활용됩니다.  
  
`@Configuration`은 우선 `@Component`를 메타 애노테이션으로 가지고 있습니다.  
1. 자신이 스프링의 빈 오브젝트로 관리될 대상임을 각주로 표시합니다.  
2. 프레임워크에게는 `@ComponentScan`대상이 될수 있는 정보를 표시합니다.  
  
스프링 부트의 관례에 따르면 `HelloApplication`은 그 자체로 스프링의 빈으로 등록되야합니다. 
[스프링 부트-@Configuration](https://docs.spring.io/spring-framework/reference/core/beans/java/composing-configuration-classes.html#beans-java-injecting-imported-beans)  
그래서 최소한 `@Component`가 있어야하고, 해당 클래스가 구성 정보를 가지고 있는 클래스이기 때문에 `@Bean`과 같은 팩토리 매소드를 넣기도 합니다. 그 중에서 `@Configuration`을 붙이는 것이 관례입니다. 
이건 `@SpringBootApplication`이라는 부트가 만든 합성 애노테이션을 보면 알 수 있습니다.
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```
해당 어노테이션은 하나 이상의 `@Bean` 메서드를 선언하고 **자동 구성**과 **컴포넌트 스캐닝을 트리거**하는 **구성 클래스**를 나타냅니다. 
이는 @SpringBootConfiguration, @EnableAutoConfiguration 및 @ComponentScan을 선언하는 것과 동일한 편의성 어노테이션입니다.  
  
사실 애플리케이션의 부트스트래핑을 해주는 클래스를 register로 직접 등록하는 것은 아주 특별한 경우입니다. 
부트의 xxxApplication으로 끝나는 클래스 정도 뿐이고, 기능적으로 보면 HelloApplication에 `@Configuration`을 
붙이지 않아도 정상적으로 동작이 됩니다. 그러면 스프링 부트가 제공하는 애노테이션(`StringBootApplication`) 애노테이션이 붙은 클래스도 
register 매서드 매개변수로 등록이 되는데 거기에는 왜 또 @Configuration이 들어가 있을까요?  
  
1. 주석으로 이해한다.  
`HelloApplication`클래스는 스프링 빈으로 관리하고, 그 중에서도 구성 정보를 자바 코드로 다루는 것을 나타냅니다. 
애플리케이션 구조 전반에 적용되는 `@ComponentScan`을 `@Controller` 웹 요청 클래스에 붙이지 않는 것처럼 자연스럽게 `@Configuration` 빈으로 정의된 클래스에
나오게 됩니다. 그래서 스프링 부트의 애노테이션도 부트스트래핑으로 직접 등록되는 빈 클래스라고 하더라도 
`@Configuration`을 붙이게 됩니다.
2. 만약 HelloApplication이 모듈의 루트 클래스가 아닐 경우  
하위 패키지에서 빈 클래스를 등록시키기 위해서 `@ComponentScan`이 붙더라도 부트의 부트스트래핑 클래스가 아니게 될 수 있습니다. 
물론 이때도 `@Import`나 자동 구성에 의해서 등록될 수 있지만 register로 등록되는건 아닙니다. 
1번의 부가적인 설명을 해주신거 같아요  
  
자바 코드로 빈을 등록하는 방식을 선택했다면, 명시적으로 빈이 되는 클래스에는 `@Component`류의 애노테이션을 붙이는 것을 권장합니다. 
다른 개발자들이 읽을 때에 주석으로서 활용이 될 수 있기 때문입니다.  
  
