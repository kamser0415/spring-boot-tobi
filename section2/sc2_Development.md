# HelloController
```java
package toby.hello;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(String name) {
        return "Hello "+name;
    }
}
```
컨트롤러라는 거는 스프링 컨테이너안에 마치 웹 컨테이너 안에 있는 웹 컴포넌트처럼 웹의 요청을 받아서  
그 결과를 리턴해주는 건데 여기서는 `@RestController`를 사용합니다.  
  
`REST`방식을 이용을 해서 `HTML`을 통째로 리턴하는 대신에 API 요청에 대한 응답을  
HTTP body에 특정한 타입으로 인코딩해서 보내는 컨트롤러를 만들때 사용하는 방식입니다.  

약속된 규칙에서 행위(`method`)를 `GET`으로 되어있는 것만 받겠다는 의미입니다.  
`PATH`는 "/hello"로 시작하는 URL만 처리하겠다는 것을 이렇게 지정합니다.  

그리고 파라미터를 하나 받을 겁니다.  
파라미터를 전달 받으러면 `hello()`메소드에 매개변수를 지정하면 됩니다.  
이제 `name` 이라는 쿼리스트링을 받아서 반환값으로 전달할 수 있습니다.  

# Hello API 테스트  
"/hello"라는 URL로 시작하는 경로뒤에 Query String으로 Name이라는 파라미터를 주고  
파라미터 값에 따라 리턴되어 웹 페이지 화면에 출력되는 것을 확인할 수 있습니다.  
  
웹 브라우저를 통해서 우리가 만든 컨트롤러 기능이 우리가 기대했던 대로 정상적으로 동작하는지 확인할 수 있습니다.  
이런 테스트를 했지만, 과연 우리가 만드는 웹 컨트롤러의 기능을 이렇게 브라우저 화면을 통해서만 확인할 필요는 없습니다.

_**그리고 웹 브라우저로 확인하는 테스트가 충분한지 생각해봐야합니다.**_
 
우리가 작성한 코드는 컨트롤러 코드입니다.  
이건 API라고 할 수 있습니다. HTML View를 리턴한게 아닙니다.  
그래서 API 테스트를 하는 방법이 브라우저를 이용하는 방법이 되게 간단하고 쉽게 할 수 있지만  

그런데 이보다 더 세밀하게 테스트를 해야될 경우가 있습니다.  
웹 애플리켙이션은 기본적으로 웹 클라이언트에서 전달하는 HTTP 요청, 리퀘스트를 받아서  
다이나믹한 컨텐츠를 생성하고 그것을 HTTP Response 응답으로 리턴하는 기능을 수행하는데  
이 요청과 응답이 우리가 생각한 대로 바르게 전달되고  
우리가 작성한 웹 컴포넌트(빈)가 기능을 수행한 뒤에 응답이 만들어지는지  
이 모든 과정을 살펴봐야 API의 모든 기능을 테스트 했다고 볼 수 있습니다.  

> 우리가 브라우저에서 눈으로 화면에 출력된 것만 확인하는 것이 아니고  
> 눈에 보이지 않지만 사실 그 뒤에서 동작하는 내용들이 어떤 것들이 있는지 살펴보는 게 필요합니다.  

HTTP API를 테스트하는데 우리가 사용하는 도구들이 많이 있지만,  
흔히 사용할 수 있는 것중에 하나가 웹 브라우저 개발자 도구를 이용하는 겁니다.  
여기서 웹 브라우저가 어떤 요청을 보내고, 서버가 어떤 응답을 하는지 확인할 수 있습니다.  
요청할 때에 어떤 요청을 갖고 어떤 헤더 값들이 전달되고 서버는 어떤 헤더 값을 받았는지   
바디가 어떻게 왔는지 그 타입은 무엇인지 이런 추가적인 정보들을 확인할 수 있습니다.  
* 자세한 방법은 크롬 개발자 툴을 확인합니다.  
  
크롬 개발자 도구를 통해서 API 테스트를 한다면,  
일단 헤더를 먼저 확인해봅니다. 요청하는 헤더는 어떤 정보가 날아가는지 알아야합니다.  
이 헤더 정보도 중요한 요청의 구성 요소로 헤더가 어떻게 구성되느냐에  
따라서 웹 애플리케이션이 동작하는 방식이 달라집니다.   

HTTP request Header 입니다.
```text
GET /hello?name=doll HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,
        image/avif,image/webp,image/apng,*/*;q=0.8,
        application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: ko,en;q=0.9
Cache-Control: max-age=0
Connection: keep-alive
Cookie: Idea-27f53e72=c3f2bd74-ed99-4df0-a229-7b5a789e8c6c
Host: localhost:8080
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
sec-ch-ua: "Google Chrome";v="119", "Chromium";v="119", "Not?A_Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
```  
특별한 form이나 api 자바스크립트 코드 같은 것이 동작해서 요청이 간게 아니라면 url로 넘어간 것은  
request method, http method라고 하죠 get이라는 method 타입으로 넘어갑니다.  

응답은 
```text
HTTP/1.1 200
Content-Type: text/html;charset=UTF-8
Content-Length: 10
Date: Sat, 11 Nov 2023 10:33:14 GMT
Keep-Alive: timeout=60
Connection: keep-alive
```  
상태코드는 200(ok)가 왔습니다. 문제없이 결과를 처리했고 응답을 만들었다는 걸 확인할 수 있습니다.  
그리고 응답에도 헤더들이 많이 붙는데, 헤더들 중에서 눈여겨볼 만한 건 컨텐츠 타입이라는 겁니다.  
화면에 뿌려진게 `HTML`인지 `JSON`인지 아니면 단순 `text/plain`인지 알아야하는데  
이건 응답 헤더의 컨텐트 타입을 보면 됩니다. 이런 상세한 정보들이 우리가 이 애플리케이션을 만들 때  
컨트롤러 API를 호출할 때 기대했던 것인가, 그 기대했던 대로 동작을 한 것인가  
이것까지 확인해보는 게 웹의 기능을 테스트하는데 사실은 필요합니다.  

크롬 브라우저의 개발자 도구같은 브라우저 도구를 이용해 테스트 하는 것도 가능한데  
이게 생각보다 사용이 불편할 수 있습니다. 그래서 보통 많이 쓰는건 커맨드라인에서 사용하는 도구들을 이용합니다.  
#### 웹 브라우저 개발자 도구  
+ curl
+ HTTPie
+ Intellij IDEA Ultimate- http request
+ Postman API Platform
+ JUnit Test
+ 각종 API 테스트 도구
> 강사님은 HTTPie를 사용하는걸 추천합니다.  
> 이유는 간단한 명령으로 HTTP 요청과 응답을 확인해 볼 수 있기 때문입니다.
  
그리고 눈으로 확인하는 테스트가 아니라 자동으로 결과를 확인할 수 있는 Junit Test 도구도 있습니다.  
+ httpie  
```text
C:\Users\yousd>http -v ":8080/hello?name=Spring"
GET /hello?name=Spring HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:8080
User-Agent: HTTPie/3.2.2

HTTP/1.1 200
Connection: keep-alive
Content-Length: 12
Content-Type: text/plain;charset=UTF-8
Date: Sat, 11 Nov 2023 11:18:25 GMT
Keep-Alive: timeout=60

Hello Spring
```  
웹 브라우저에서 확인할 수 있는 것보다 더 많은 정보를 볼 수 있습니다. 
1. 응답 코드 확인하기
2. content-type 확인하기
3. 그외  

원하는 응답코드가 내려왔는지 확인해야합니다.  
그 다음에 중요한 건 content-type 입니다.  
HTTP body에 들어오는 값이 plain text인지, 아니면 html인지, json이나 xml인지  
또 다른 인코딩인지 타입을 확인할 수 있는 정보입니다.  
이렇게 HTTP 요청과 응답을 확인하는 것을 모두 맞춰야 API에 대한 최종 테스트가 마무리됩니다.  
