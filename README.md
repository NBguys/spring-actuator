# 액츄에이터
액츄에이터가 제공하는 프로덕션 준비 기능을 사용하려면 스프링 부트 액츄에이터 라이브러리를 추가해야 한다.
참고로 앞의 프로젝트 설정에서 추가해두었다.
build.gradle - 추가 
```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator' //
actuator 추가 
```

동작 확인
* 기본 메인 클래스 실행( ActuatorApplication.main() )
* http://localhost:8080/actuator 실행

액츄에이터 기능을 웹에 노출
application.yml - 추가 
```yaml
management:
endpoints:
web:
exposure:
include: "*" 
```

엔드포인트 활성화
application.yml - shutdown 엔드포인트 활성화 
```yaml
management:
  endpoint:
    shutdown:
      enabled: true
  endpoints:
    web:
      exposure:
        include: "*" 
```
특정 엔드포인트를 활성화 하려면 management.endpoint.{엔드포인트명}.enabled=true 를 적용하면 된다.  
이제 Postman 같은 것을 사용해서 HTTP POST로 http://localhost:8080/actuator/shutdown 를 호출  
하면 다음 메시지와 함께 실제 서버가 종료되는 것을 확인할 수 있다.
```json
{"message": "Shutting down, bye..."} 
```

엔드포인트 노출
```yaml
management:
endpoints:
jmx:
exposure:
include: "health,info" 
```
* jmx 에 health,info 를 노출한다.
```yaml
management:
 endpoints:
 web:
 exposure:
 include: "*"
 exclude: "env,beans" 
 ```
* web 에 모든 엔드포인트를 노출하지만 env , beans 는 제외한다.

### 다양한 엔드포인트
각각의 엔드포인트를 통해서 개발자는 애플리케이션 내부의 수 많은 기능을 관리하고 모니터링 할 수 있다.  
스프링 부트가 기본으로 제공하는 다양한 엔드포인트에 대해서 알아보자. 다음은 자주 사용하는 기능 위주로 정리했다.  

#### 엔드포인트 목록
* beans : 스프링 컨테이너에 등록된 스프링 빈을 보여준다.
* conditions : condition 을 통해서 빈을 등록할 때 평가 조건과 일치하거나 일치하지 않는 이유를 표시한다.
* configprops : @ConfigurationProperties 를 보여준다.
* env : Environment 정보를 보여준다.
* health : 애플리케이션 헬스 정보를 보여준다.
* httpexchanges : HTTP 호출 응답 정보를 보여준다. HttpExchangeRepository 를 구현한 빈을 별도로 등록해야 한다.
* info : 애플리케이션 정보를 보여준다.
* loggers : 애플리케이션 로거 설정을 보여주고 변경도 할 수 있다.
* metrics : 애플리케이션의 메트릭 정보를 보여준다.
* mappings : @RequestMapping 정보를 보여준다.
* threaddump : 쓰레드 덤프를 실행해서 보여준다.
* shutdown : 애플리케이션을 종료한다. 이 기능은 기본으로 비활성화 되어 있다.

> 전체 엔드포인트는 다음 공식 메뉴얼을 참고하자.  
> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints

### 헬스 정보
헬스 정보를 사용하면 애플리케이션에 문제가 발생했을 때 문제를 빠르게 인지할 수 있다.
http://localhost:8080/actuator/health
기본 동작 
```json
{"status": "UP"}
 ```
헬스 정보는 단순히 애플리케이션이 요청에 응답을 할 수 있는지 판단하는 것을 넘어서 애플리케이션이 사용하는 데이  
터베이스가 응답하는지, 디스크 사용량에는 문제가 없는지 같은 다양한 정보들을 포함해서 만들어진다.  
헬스 정보를 더 자세히 보려면 다음 옵션을 지정하면 된다.

management.endpoint.health.show-details=always
```yaml
management:
 endpoint:
   health:
     show-details: always
```

참고 - 자세한 헬스 기본 지원 기능은 다음 공식 메뉴얼을 참고하자
> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.auto-configured-health-indicators
> 참고 - 헬스 기능 직접 구현하기

원하는 경우 직접 헬스 기능을 구현해서 추가할 수 있다. 직접 구현하는 일이 많지는 않기 때문에 필요한 경우 다음 공식 메뉴얼을 참고하자
> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.writing-custom-health-indicators

## 애플리케이션 정보
info 엔드포인트는 애플리케이션의 기본 정보를 노출한다.

기본으로 제공하는 기능들은 다음과 같다.
* java : 자바 런타임 정보
* os : OS 정보
* env : Environment 에서 info. 로 시작하는 정보
* build : 빌드 정보, META-INF/build-info.properties 파일이 필요하다.
* git : git 정보, git.properties 파일이 필요하다.

application.yml - 내용 추가 
```yaml
management:
  info:
    java:
      enabled: true
    os:
      enabled: true 
```
management.info.<id>.enabled 의 값을 true 로 지정하면 활성화 된다.

## env
이번에는 env 를 사용해보자.  
Environment 에서 info. 로 시작하는 정보를 출력한다.  
application.yml - 내용 추가 
```yaml
management:
  info:
    env:
      enabled: true
info:
  app:
    name: hello-actuator
    company: yh 
```
management.info.env.enabled 를 추가하고, info.. 관련 내용을 추가했다.

build.gradle - 빌드 정보 추가 
```groovy
springBoot {
    buildInfo()
}
```

이렇게 하고 빌드를 해보면 build 폴더안에 resources/main/META-INF/build-info.properties 파일을 확인할 수 있다. 
```properties
build.artifact=actuator
build.group=hello
build.name=actuator
build.time=2023-01-01T00\:000\:00.000000Z
build.version=0.0.1-SNAPSHOT 
```
build 는 기본으로 활성화 되어 있기 때문에 이 파일만 있으면 바로 확인할 수 있다

### git
앞서본 build 와 유사하게 빌드 시점에 사용한 git 정보도 노출할 수 있다. git 정보를 노출하려면
git.properties 파일이 필요하다.
build.gradle - git 정보 추가 
```groovy
plugins {
...
  id "com.gorylenko.gradle-git-properties" version "2.4.1" //git info
}
```
물론 프로젝트가 git 으로 관리되고 있어야 한다. 그렇지 않으면 빌드시 오류가 발생한다.

git 에 대한 더 자세한 정보를 보고 싶다면 다음 옵션을 적용하면 된다.
application.yml 추가 
```yaml
management:
  info:
    git:
      mode: "full"
```
info 사용자 정의 기능 추가
> info 의 사용자 정의 기능을 추가 하고 싶다면 다음 스프링 공식 메뉴얼을 참고하자  
> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.info.writing-custom-info-contributors

로거
loggers 엔드포인트를 사용하면 로깅과 관련된 정보를 확인하고, 또 실시간으로 변경할 수도 있다. 코드를 통해서 알아보자.
### LogController.java 참고

application.yml 설정
```yaml
logging:
  level:
    hello.controller: debug 
```
hello.controller 패키지와 그 하위는 debug 레벨을 출력하도록 했다. 이제 앞서 만든 LogController 클
래스도 debug 레벨의 영향을 받는다.

실행
http://localhost:8080/log

실행
http://localhost:8080/actuator/loggers

* 로그를 별도로 설정하지 않으면 스프링 부트는 기본으로 INFO 를 사용한다. 실행 결과를 보면 ROOT 의
  configuredLevel 가 INFO 인 것을 확인할 수 있다. 따라서 그 하위도 모두 INFO 레벨이 적용된다.
* 앞서 우리는 hello.controller 는 DEBUG 로 설정했다. 그래서 해당 부분에서 configuredLevel
  이 DEBUG 로 설정된 것을 확인할 수 있다. 그리고 그 하위도 DEBUG 레벨이 적용된다.


더 자세히 조회하기  
다음과 같은 패턴을 사용해서 특정 로거 이름을 기준으로 조회할 수 있다.
http://localhost:8080/actuator/loggers/{로거이름}

실행
http://localhost:8080/actuator/loggers/hello.controller

### 실시간 로그 레벨 변경
개발 서버는 보통 DEBUG 로그를 사용하지만, 운영 서버는 보통 요청이 아주 많다. 따라서 로그도 너무 많이 남기 때문
에 DEBUG 로그까지 모두 출력하게 되면 성능이나 디스크에 영향을 주게 된다. 그래서 운영 서버는 중요하다고 판단되
는 INFO 로그 레벨을 사용한다.

loggers 엔드포인트를 사용하면 애플리케이션을 다시 시작하지 않고, 실시간으로 로그 레벨을 변경할 수 있다.  
다음을 Postman 같은 프로그램으로 POST로 요청해보자(꼭! POST를 사용해야 한다.)  
POST http://localhost:8080/actuator/loggers/hello.controller
POST로 전달하는 내용 JSON , content/type 도 application/json 으로 전달해야 한다.
```json
{
  "configuredLevel": "TRACE"
}
```
참고로 이것은 POST에 전달하는 내용이다. 응답 결과가 아니다.  
요청에 성공하면 204 응답이 온다.(별도의 응답 메시지는 없다.)  

### HTTP 요청 응답 기록
HTTP 요청과 응답의 과거 기록을 확인하고 싶다면 httpexchanges 엔드포인트를 사용하면 된다.
HttpExchangeRepository 인터페이스의 구현체를 빈으로 등록하면 httpexchanges 엔드포인트를 사용할 수 있다.  
(주의! 해당 빈을 등록하지 않으면 httpexchanges 엔드포인트가 활성화 되지 않는다)  
스프링 부트는 기본으로 InMemoryHttpExchangeRepository 구현체를 제공한다.  

InMemoryHttpExchangeRepository.java 참고

실행
http://localhost:8080/actuator/httpexchanges
실행해보면 지금까지 실행한 HTTP 요청과 응답 정보를 확인할 수 있다.  
참고로 이 기능은 매우 단순하고 기능에 제한이 많기 때문에 개발 단계에서만 사용하고, 실제 운영 서비스에서는 모니터  
링 툴이나 핀포인트, Zipkin 같은 다른 기술을 사용하는 것이 좋다.  

### 액츄에이터와 보안
#### 보안 주의
액츄에이터가 제공하는 기능들은 우리 애플리케이션의 내부 정보를 너무 많이 노출한다. 그래서 외부 인터넷 망이 공개  
된 곳에 액츄에이터의 엔드포인트를 공개하는 것은 보안상 좋은 방안이 아니다.  
액츄에이터의 엔드포인트들은 외부 인터넷에서 접근이 불가능하게 막고, 내부에서만 접근 가능한 내부망을 사용하는 것  
이 안전하다.  

#### 액츄에이터를 다른 포트에서 실행  
예를 들어서 외부 인터넷 망을 통해서 8080 포트에만 접근할 수 있고, 다른 포트는 내부망에서만 접근할 수 있다면 액  
츄에이터에 다른 포트를 설정하면 된다.  
액츄에이터의 기능을 애플리케이션 서버와는 다른 포트에서 실행하려면 다음과 같이 설정하면 된다. 이 경우 기존  
8080 포트에서는 액츄에이터를 접근할 수 없다.  

액츄에이터 포트 설정
```yaml
management:
  server:
    port: 9292
```

실행
http://localhost:9292/actuator

엔드포인트 경로 변경
엔드포인트의 기본 경로를 변경하려면 다음과 같이 설정하면 된다.  
application.yml
```
management:
 endpoints:
   web:
     base-path: "/manage"
```
/actuator/{엔드포인트} 대신에 /manage/{엔드포인트} 로 변경된다.