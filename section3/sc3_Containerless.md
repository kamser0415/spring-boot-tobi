# Container 개발 준비  
> 스프링 부트의 두가지 특징 중에 하나인 컨테이너리스 라는게 어떤 식으로 만들어졌고  
> 어떻게 동작하는지 알아야합니다.  
  
사실 컨테이너 리스를 제안한다는 것은 서블릿 컨테이너와 관련된 번거롭고 복잡한 작업들과  
그것을 하기위해서 필요한 지식들을 개발자들이 더 이상 신경쓰지 않고  
스프링 컨테이너에 올라가는 컴포넌트 빈을 만드는 거에만 집중해서 애플리케이션을 개발하도록 합니다.  
  
XML, 서블릿 컨테이너 설치, 배포 이런 작업들 전부 필요 없이 Stand alone으로 동작하는  
메인 메소드를 실행하는 간단한 방법으로 스프링 어플리케이션을 동작시킨다는 의미입니다.  
  
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(String name) {
        return "Hello "+name;
    }
}
```  
스프링 부트에서 컨트롤러를 만들고 결과를 확인할 때까지 우리는 톰캣을 설치하고 배포하기 위해서  
빌드 스크립트를 작성하지 않았습니다. 단지 Spring Boot가 처음 만들어준 `main()`메소드를 실행만 했는데  
서블릿 컨테이너인 Tomcat이 동작하고,그리고 Spring과 관련된 어떠한 설정도 하지 않았는데 `Spring Container`도 같이 떴습니다.  
  
컨트롤러로 만들어 놓은 저 코드가 스프링 위에 올라갔기 때문에 우리가 이 컨트롤러의 기능을 동작시킬수 있었습니다.  
```java
@SpringBootApplication
public class HelloApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }

}
```  
그런 복잡한 작업들이 간단한 `main()`메소드 코드안에서 다 일어나는 겁니다.  
파라미터로 클래스(`HelloApplication.class`)를 전달했습니다.  

그 다음에 눈에 띄는게 파라미터로 전달한 클래스위에 붙어있는 `@SpringBootApplication`라는 겁니다.  
이 두 가지만 있을 뿐인데 컨테이너와 관련된 모든 작업을 포함해서 스프링이 기동되게 만드는 모든 작업들이  
다 알아서 진행이 되고 개발자는 스프링 컨테이너에 등록할 컨트롤러 코드만 만들면 됩니다.  
  
여기서 어떤 일이 일어나는지 이해하기 위해서 `Spring Boot`가 없다고 생각하고  
```java
public class HelloApplication {
    public static void main(String[] args) { }
}
```  
스프링 부트의 코드를 제거하고 메인 메소드를 실행하면 아무 일이 일어나지 않습니다.  
  
Tomcat이 올라왔다는 로그도 안보이고, port 8080에 요청을 보내도 서버가 존재하지 않는다고 나오겠죠  
이 상태에서 예전처럼 잘 동작하게 만드는 작업을 시작해보겠습니다.