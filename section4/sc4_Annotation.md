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
