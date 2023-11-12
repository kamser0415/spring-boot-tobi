# 프론트 컨트롤러로 전환
```java
servletContext.addServlet("hello", new HttpServlet() {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String name = req.getParameter("name");

        resp.setStatus(HttpStatus.OK.value());
        resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
        PrintWriter writer = resp.getWriter();
        writer.println("hello "+name);
    }
}).addMapping("/hello");
```  
`/hello`라는 URL에 매핑을한 서블릿을 하나 만들고 서블릿 컨테이너에 등록했습니다.  
이제 해당 서블릿은 `/hello`로 들어오는 요청만 위임받아서 처리하는 기능을 가진 서블릿입니다.  
  
이 서블릿을 프론트 컨트롤러  
즉, 모든 서블릿의 공통 로직을 수행하는 서블릿으로 만들려고 합니다.  
큰 의미는 없지만 이름부터 바꾸고, 더 중요한 매핑부분을 변경해야합니다.  

프론트 컨트롤러는 중앙화된 처리를 하기 위해서 모든 클라이언트의 요청을 다 받아야합니다.  
모든 URL의 요청을 받기 위해서 매핑을 `/*`로 변경합니다.
```java
servletContext.addServlet("frontController", new HttpServlet() {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String name = req.getParameter("name");

        resp.setStatus(HttpStatus.OK.value());
        resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
        PrintWriter writer = resp.getWriter();
        writer.println("hello "+name);
    }
}).addMapping("/*");
```  
이제 `frontController` 이름을 가진 프론트 컨트롤러 책임을 가진 서블릿이 되었습니다.  
클라이언트의 다양한 요청 Path`/hello`,`/member`등 다양한 요청이 모두 해당 `frontController`로 들어오게 됩니다.  
  
`frontController`는 여러 개의 서블릿의 공통 로직, 인증, 보안, 다국어 처리등을 처리합니다.  
지금 다 구현할 수 없으니 그걸 했다고 생각하고, 기존에는 `URL`을 직접 매핑해서 서블릿 컨테이너가 직접 위임을 했습니다.  
  

_**서블릿 컨테이너의 매핑역할을 `FrontController`가 담당합니다.**_  

매핑은 요청(request object)을 가지고 하는 겁니다.  
웹 요청이 들어오면 그 요청에 들어있는 3가지 요소가 있었죠  
### Request
+ Request Line: Method, Path, HTTP Version
+ Headers
+ Message Body  

`Request Line`의 Method,path url,headers,body 이 정보들이 다 매핑하는데 활용할 수 있습니다.  
제일 대표적으로 사용하는건 `path url`입니다.
```java
 servletContext.addServlet("frontController", new HttpServlet() {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String requestURI = req.getRequestURI();

        if(requestURI.equals("/hello")){
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
```
위에 코드는 `path url`이 `/hello`로 들어오는 것만 처리를 했으면 하는 겁니다.  
그래서 여기다가 request에서 패스 정보를 가져오는 `getRequestURI()`를 사용합니다.  
요청 url이 `/hello`,`/member`등이 오면 우리가 작성한 응답 객체를 반환하고  
매핑된 url이 없다면 상태 코드`404`를 반환하도록 수정했습니다.  

그런데 예전 `@RestController`를 보면 `@GetMapping("/hello")`로 메소드가 `GET`인 요청만 위임받았습니다.  
현재 코드는 모든 메소드를 다 위임받아 처리하는 형식이죠.  
이제 그 부분까지 처리를 할 수 있습니다.  
  
`request Object`에서 요청 메소드를 꺼내서 `GET` 메소드이고, 요청 url이 `/hello`인 경우만 허용하도록 수정합니다.  
```java
if(requestURI.equals("/hello") && req.getMethod().equals(HttpMethod.GET.name()))
```  
정리하면, `/hello`로 요청이 들어왔지만 요청 Method가 `GET`이 아닐 경우에는  
처리 할 수 없고 아니면 다른 처리하는 코드를 찾고, 없으면 `404`를 반환하는 로직이 되었습니다.

+ GET 요청 테스트
```text
$ http -v :8080/hello?name=padoc
GET /hello?name=padoc HTTP/1.1

HTTP/1.1 200
Connection: keep-alive
Content-Length: 13
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 12 Nov 2023 07:26:42 GMT
Keep-Alive: timeout=60

hello padoc
```  
+ POST 요청 테스트  
```text
POST /hello?name=padoc HTTP/1.1

HTTP/1.1 404                       
Connection: keep-alive             
Content-Length: 0                  
Date: Sun, 12 Nov 2023 09:40:16 GMT
Keep-Alive: timeout=60  
```
요청 URL은 `/hello`로 동일하지만 요청 메소드가 다를 경우에는  
`404`를 반환하는 것을 확인했습니다.  
  
이렇게 프론트 컨트롤러를 만들고 보니 생각해보면 실제적인 웹 애플리케이션 로직을 담당하는 부분은  
다른 오브젝트한테 위임을 해야합니다.  
  
이게 프론트 컨트롤러가 동작하는 방식입니다.  
  
여기서 이제 다른 컨트롤러한테 또 `HttpRequest`,`HttpResponse` 요청,응답 객체를  
그대로 넘겨서 보낼 것이냐, 아니면 다른 방식으로 요청할 것인지 생각할 부분이 생겼습니다.
