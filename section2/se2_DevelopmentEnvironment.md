# 스피링 부트 개발 환경  
> Spring Boot 2.7.6 기준  
  
스프링 부트로 개발을 시작하면, 스프링 부트 버전부터 결정해야합니다.  
스프링 부트 버전이 결정되면 JDK 버전과, 스프링 버전도 결정됩니다.  
그렇다고 하나로 제안하지 않고 부트가 사용하는 JDK 8,11,17 버전중에 하나를 사용할 수 있습니다.  

### 개발 환경  
#### JDK  
JDK 8,11,17 설치
+ 공개 JDK 다운로드 후 설치  
  + Eclipse Temurin
  + Microsoft OpenJDK
  + Amazon Corretto
  + Azul JDK
  + Oracle JDK  
```text
- WAF 커맨트라인 툴을 사용
SDKMAN 이라는 매니저 툴을 사용해서 버전을 관리할 수 있다.
```
+ https://sdkman.io/
+ https://github.com/shyiko/jabba  
#### IDE
+ IntelliJ IDEA : https://www.jetbrains.com/idea/download  
  + Ultimate
  + Community
+ STS : https://spring.io/tools
+ Visual Studio Code : https://code.visualstudio.com/  
### SpringBoot  
  Spring Boot CLI : https://docs.spring.io/spring-boot/docs/2.7.x/reference/htmlsingle/#getting-started.installing.cli  
```text
CLI를 설치해서 사용할 수 있습니다.  
SDK MAN을 통해서 설치할 수 있습니다.
```
### HelloBoot 웹 프로젝트 생성
#### 스프링 부트 프로젝트 생성
+ 웹 Spring Initializr - https://start.spring.io/
+ IDE의 Spring Initializr 프로젝트 생성 메뉴
+ Spring Boot CLI
#### 생성 옵션
+ Project: Gradle
+ Langauge: Java
+ SpringBoot Version: 2.7.6
+ Group Id: tobyspring
+ Name: hello
+ Packaging: Jar
+ Java Version: 11
+ Dependency: Web  

