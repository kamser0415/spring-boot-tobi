# 서블릿 요청 처리  
> 이전 장에서는 서블릿 컨테이너를 별다른 설정없이 스프링 부트가 제공하는 기본 설정이 포함된  
> 자바 코드로 실행을 했습니다. 그 후 서블릿 컨테이너에 특정 URL로 들어오면 응답하는 서블릿을 등록했습니다.  
 
+ 초기 서블릿 코드
    ```text
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            resp.setStatus(200);
            resp.setHeader("Content-Type","text/plain");
            PrintWriter writer = resp.getWriter();
            writer.println("hello");
        }
    }).addMapping("/hello");
    ```   
`Response HTTP 3 요소`가 모두 포함되어 있습니다.  
하지만 헤더에서 `Content-Type`과 그 value인 `text/plain`을 `String` 문자로 작성했습니다.  
문자열을 하드코딩을 해서 넣는건 많이 사용하는 방법이지만, 오타가 발생하거나 빼먹는 문자가 발생한다면  
원하는 결과를 웹 클라이언트가 받지 못할 수 있습니다.  
  
_**수정 사항**_
+ http 속성을 `String`-> `enum`으로 교체해 오타및 생략 문제를 제거합니다.  
  

+ 수정후 서블릿 코드  
   ```java
  import org.springframework.http.HttpHeaders;
  import org.springframework.http.HttpStatus;
  import org.springframework.http.MediaType;
  
   servletContext.addServlet("hello", new HttpServlet() {
      @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.setStatus(HttpStatus.OK.value());
          resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
          PrintWriter writer = resp.getWriter();
          writer.println("hello");
      }
  }).addMapping("/hello");
   ```  
  
이제 오타나, 문자열 실수를 줄일 수 있는 코드로 리팩토링했습니다.  
이 `enum`타입은 서블릿뿐만 아니라 우리가 사용하는 HTTP requset/response를 직접 관리하는  
`Spring MVC` 애플리케이션에서도 얼마든지 사용할 수 있습니다.  

코드를 변경하고 다시 `httpie`를 통해서 원하는 `http`로 요청/응답된지 확인합니다.  
```text
$ http -v :8080/hello
GET /hello HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:8080
User-Agent: HTTPie/3.2.2

HTTP/1.1 200
Connection: keep-alive
Content-Length: 7
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 12 Nov 2023 07:11:41 GMT
Keep-Alive: timeout=60

Spring
```  
원하는 응답 코드와 컨텐트 타입, 바디의 내용이 포함되어있습니다.  
  
### 요청 정보를 컨트롤 하기  
현재까지는 응답에 대한 `http`에 대해서 작성했습니다.  
요청으로 들어온 객체 `request`를 어떻게 우리가 컨트롤 할 수 있을까?  
요청 중에서 URL로 들어온 부분은 어느 Sevlert을 사용할지 매핑하는데 이미 사용되었고,  
그거 말고 처음 만들었던 `@RestController`예제처럼 `Query String`에 name이라는  
파라미터를 전달받아 동적인 응답을 만들어내는 코드를 작성해보겠습니다.  

> 요구사항
+ 웹 클라이언트가 요청한 `http`를 컨트롤해서 'Query String'에 name이라는 파라미터로 전달받아서  
  동적인 응답을 만들어내는 코드로 수정합니다.   

파라미터로 넘어오는 것은 `httpSevletRequest`타입의  `getParameter` 이 메소드를 이용합니다.  
```java
String name = req.getParameter("name");
```  
이 코드를 서블릿 내에서 작성하면 파라미터로 넘어오는 요청 정보중에서  
쿼리 스트링중에서 `name` 문자열이 키인 데이터를 가져옵니다.  
그 결과를 응답 바디에 작성합니다.  
```bash
$ http -v :8080/hello?name=padoc
GET /hello?name=padoc HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:8080
User-Agent: HTTPie/3.2.2

HTTP/1.1 200
Connection: keep-alive
Content-Length: 13
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 12 Nov 2023 07:26:42 GMT
Keep-Alive: timeout=60

hello padoc
```
