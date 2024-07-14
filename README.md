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

# 마이크로미터, 프로메테우스, 그라파나
한번 더 이야기 하지만 전투에서 실패한 지휘관은 용서할 수 있지만 경계에서 실패하는 지휘관은 용서할 수 없다 라는 말이 있다.   
이 말을 서비스를 운영하는 개발자에게 맞추어 보면 장애는 언제든지 발생할 수 있다. 하지만 모니터링(경계)은 잘 대응하는 것이 중요하다.   
서비스를 운영할 때는 애플리케이션의 CPU, 메모리, 커넥션 사용, 고객 요청수 같은 수 많은 지표들을 확인하는 것이필요하다. 
그래야 어디에 어떤 문제가 발생했는지 사전에 대응도 할 수 있고, 실제 문제가 발생해도 원인을 빠르게 파악해서 대처할 수 있다.  
예를 들어서 메모리 사용량이 가득 찼다면 메모리 문제와 관련있는 곳을 빠르게 찾아서 대응할 수 있을 것이다.  

세상에는 수 많은 모니터링 툴이 있고, 시스템의 다양한 정보를 이 모니터링 툴에 전달해서 사용하게 된다.
그라파나 대시보드, 핀포인트 툴 등이 있다.

그런데 중간에 사용하는 모니터링 툴을 변경하면 어떻게 될까?
기존에 측정했던 코드를 모두 변경한 툴에 맞도록 다시 변경해야 한다. 개발자 입장에서는 단순히 툴 하나를 변경했을뿐인데, 
측정하는 코드까지 모두 변경해야 하는 문제가 발생한다.
이런 문제를 해결하는 것이 바로 마이크로미터(Micrometer)라는 라이브러리이다.

마이크로미터 추상화
![스크린샷1](/src/main/resources/img/screenshot1.png)
![스크린샷2](/src/main/resources/img/screenshot2.png)

* 마이크로미터는 애플리케이션 메트릭 파사드라고 불리는데, 애플리케이션의 메트릭(측정 지표)을 마이크로미터가 정한 표준 방법으로 모아서 제공해준다.  
* 쉽게 이야기해서 마이크로미터가 추상화를 통해서 구현체를 쉽게 갈아끼울 수 있도록 해두었다.  
* 보통은 스프링이 이런 추상화를 직접 만들어서 제공하지만, 마이크로미터라는 이미 잘 만들어진 추상화가있기 때문에 스프링은 이것을 활용한다.  
  스프링 부트 액츄에이터는 마이크로미터를 기본으로 내장해서 사용한다.  
  * 로그를 추상화 하는 SLF4J 를 떠올려보면 쉽게 이해가 될 것이다.  
* 개발자는 마이크로미터가 정한 표준 방법으로 메트릭(측정 지표)를 전달하면 된다.   
  그리고 사용하는 모니터링 툴에 맞는 구현체를 선택하면 된다. 이후에 모니터링 툴이 변경되어도 해당 구현체만 변경하면 된다.   
  애플리케이션 코드는 모니터링 툴이 변경되어도 그대로 유지할 수 있다.  

마이크로미터가 지원하는 모니터링 툴은 다음과 같다. 
* AppOptics
* Atlas
* CloudWatch
* Datadog
* Dynatrace
* Elastic
* Ganglia
* Graphite
* Humio
* Influx
* Instana
* JMX
* KairosDB
* New Relic
* Prometheus
* SignalFx
* Stackdriver
* StatsD
* Wavefront

> 참고
> 각 모니터링 툴에 대한 자세한 내용은 마이크로미터 공식 메뉴얼을 참고하자
> https://micrometer.io/docs


### metrics 엔드포인트
metrics 엔드포인트를 사용하면 기본으로 제공되는 메트릭들을 확인할 수 있다.    
http://localhost:8080/actuator/metrics  

metrics 엔드포인트는 다음과 같은 패턴을 사용해서 더 자세히 확인할 수 있다.   
http://localhost:8080/actuator/metrics/{name}  

#### JVM 메모리 사용량을 확인해보자.  
실행  
http://localhost:8080/actuator/metrics/jvm.memory.used  

#### Tag 필터
availableTags 를 보면 다음과 같은 항목을 확인할 수 있다.
* tag:area , values[heap, nonheap]
* tag:id , values[G1 Survivor Space, ...]
해당 Tag를 기반으로 정보를 필터링해서 확인할 수 있다.  
tag=KEY:VALUE 과 같은 형식을 사용해야 한다.  
다음과 같이 실행해보자.  
* http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:heap
* http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:nonheap

#### HTTP 요청수를 확인
* http://localhost:8080/actuator/metrics/http.server.requests
HTTP 요청수에서 일부 내용을 필터링 해서 확인해보자.  
/log 요청만 필터 (사전에 /log 요청을 해야 확인할 수 있음)  
* http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/log
/log 요청 & HTTP Status = 200
* http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/log&tag=status:200

다양한 메트릭
마이크로미터와 액츄에이터가 기본으로 제공하는 다양한 메트릭을 확인해보자.
* JVM 메트릭
* 시스템 메트릭
* 애플리케이션 시작 메트릭
* 스프링 MVC 메트릭
* 톰캣 메트릭
* 데이터 소스 메트릭
* 로그 메트릭
* 기타 수 많은 메트릭이 있다.
* 사용자가 메트릭을 직접 정의하는 것도 가능하다. 뒤에서 예제로 만들어본다.

JVM 메트릭
JVM 관련 메트릭을 제공한다. jvm. 으로 시작한다.
* 메모리 및 버퍼 풀 세부 정보
* 가비지 수집 관련 통계
* 스레드 활용
* 로드 및 언로드된 클래스 수
* JVM 버전 정보
* JIT 컴파일 시간

시스템 메트릭
시스템 메트릭을 제공한다. system. , process. , disk. 으로 시작한다.
* CPU 지표
* 파일 디스크립터 메트릭
* 가동 시간 메트릭
* 사용 가능한 디스크 공간

애플리케이션 시작 메트릭
애플리케이션 시작 시간 메트릭을 제공한다.
* application.started.time : 애플리케이션을 시작하는데 걸리는 시간( ApplicationStartedEvent 로 측정)
* application.ready.time : 애플리케이션이 요청을 처리할 준비가 되는데 걸리는 시간( ApplicationReadyEvent 로 측정)

스프링은 내부에 여러 초기화 단계가 있고 각 단계별로 내부에서 애플리케이션 이벤트를 발행한다.
* ApplicationStartedEvent : 스프링 컨테이너가 완전히 실행된 상태이다. 이후에 커맨드 라인 러너가호출된다.
* ApplicationReadyEvent : 커맨드 라인 러너가 실행된 이후에 호출된다.

### 스프링 MVC 메트릭
스프링 MVC 컨트롤러가 처리하는 모든 요청을 다룬다.  
메트릭 이름: http.server.requests  

TAG 를 사용해서 다음 정보를 분류해서 확인할 수 있다.
* uri : 요청 URI
* method : GET , POST 같은 HTTP 메서드
* status : 200 , 400 , 500 같은 HTTP Status 코드
* exception : 예외
* outcome : 상태코드를 그룹으로 모아서 확인 1xx:INFORMATIONAL , 2xx:SUCCESS , 3xx:REDIRECTION , 4xx:CLIENT_ERROR , 5xx:SERVER_ERROR

#### 데이터소스 메트릭
DataSource , 커넥션 풀에 관한 메트릭을 확인할 수 있다.  
jdbc.connections. 으로 시작한다.  
최대 커넥션, 최소 커넥션, 활성 커넥션, 대기 커넥션 수 등을 확인할 수 있다.  
히카리 커넥션 풀을 사용하면 hikaricp. 를 통해 히카리 커넥션 풀의 자세한 메트릭을 확인할 수 있다.  

#### 로그 메트릭  
logback.events : logback 로그에 대한 메트릭을 확인할 수 있다.  
trace , debug , info , warn , error 각각의 로그 레벨에 따른 로그 수를 확인할 수 있다.  
예를 들어서 error 로그 수가 급격히 높아진다면 위험한 신호로 받아드릴 수 있다.  

#### 톰캣 메트릭
톰캣 메트릭은 tomcat. 으로 시작한다.  
톰캣 메트릭을 모두 사용하려면 다음 옵션을 켜야한다. (옵션을 켜지 않으면 tomcat.session. 관련 정보만 노출된다.)  

application.yml
```yaml
server:
 tomcat:
   mbeanregistry:
     enabled: true 
 ```
톰캣의 최대 쓰레드, 사용 쓰레드 수를 포함한 다양한 메트릭을 확인할 수 있다.

#### 기타
* HTTP 클라이언트 메트릭( RestTemplate , WebClient )
* 캐시 메트릭
* 작업 실행과 스케줄 메트릭
* 스프링 데이터 리포지토리 메트릭
* 몽고DB 메트릭
* 레디스 메트릭

### 사용자 정의 메트릭
사용자가 직접 메트릭을 정의할 수도 있다. 예를 들어서 주문수, 취소수를 메트릭으로 만들 수 있다.  
사용자 정의 메트릭을 만들기 위해서는 마이크로미터의 사용법을 먼저 이해해야 한다. 이 부분은 뒤에서 다룬다.  

정리
액츄에이터를 통해서 수 많은 메트릭이 자동으로 만들어지는 것을 확인했다. 그런데 이러한 메트릭들을 어딘가에 지속  
해서 보관해야 과거의 데이터들도 확인할 수 있을 것이다. 따라서 메트릭을 지속적으로 수집하고 보관할 데이터베이스  
가 필요하다. 그리고 이러한 메트릭들을 그래프를 통해서 한눈에 쉽게 확인할 수 있는 대시보드도 필요하다.  
> 참고: 지원하는 다양한 메트릭들은 다음 공식 메뉴얼을 참고하자
> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.supported


### 프로메테우스와 그라파나 소개
#### 프로메테우스
애플리케이션에서 발생한 메트릭을 그 순간만 확인하는 것이 아니라 과거 이력까지 함께 확인하려면 메트릭을 보관하는 DB가 필요하다. 
이렇게 하려면 어디선가 메트릭을 지속해서 수집하고 DB에 저장해야 한다. 프로메테우스가 바로 이런 역할을 담당한다.

#### 그라파나
프로메테우스가 DB라고 하면, 이 DB에 있는 데이터를 불러서 사용자가 보기 편하게 보여주는 대시보드가 필요하다.
그라파나는 매우 유연하고, 데이터를 그래프로 보여주는 툴이다. 수 많은 그래프를 제공하고, 프로메테우스를 포함한 다양한 데이터소스를 지원한다.

![스크린샷](/src/main/resources/img/screenshot3.png)
* 스프링 부트 액츄에이터와 마이크로미터를 사용하면 수 많은 메트릭을 자동으로 생성한다.
  * 마이크로미터 프로메테우스 구현체는 프로메테우스가 읽을 수 있는 포멧으로 메트릭을 생성한다.
* 프로메테우스는 이렇게 만들어진 메트릭을 지속해서 수집한다.
* 프로메테우스는 수집한 메트릭을 내부 DB에 저장한다.
* 사용자는 그라파나 대시보드 툴을 통해 그래프로 편리하게 메트릭을 조회한다. 이때 필요한 데이터는 프로메테우스를 통해서 조회한다.

### 프로메테우스 아키텍처
![스크린샷](/src/main/resources/img/screenshot4.png)

출처: https://prometheus.io/docs/introduction/overview/


### 프로메테우스 - 설치
https://prometheus.io/download/

### 프로메테우스 - 애플리케이션 설정
프로메테우스는 메트릭을 수집하고 보관하는 DB이다. 프로메테우스가 우리 애플리케이션의 메트릭을 수집하도록 연동해보자.
여기에는 2가지 작업이 필요하다.
* 애플리케이션 설정: 프로메테우스가 애플리케이션의 메트릭을 가져갈 수 있도록 애플리케이션에서 프로메테우스 포멧에 맞추어 메트릭 만들기
* 프로메테우스 설정: 프로메테우스가 우리 애플리케이션의 메트릭을 주기적으로 수집하도록 설정

build.gradle 추가 
```groovy
implementation 'io.micrometer:micrometer-registry-prometheus' //추가 
```
* 마이크로미터 프로메테우스 구현 라이브러리를 추가한다.
* 이렇게 하면 스프링 부트와 액츄에이터가 자동으로 마이크로미터 프로메테우스 구현체를 등록해서 동작하도록 설정해준다.
* 액츄에이터에 프로메테우스 메트릭 수집 엔드포인트가 자동으로 추가된다.
  * /actuator/prometheus

실행
http://localhost:8080/actuator/prometheus
포멧 차이
* jvm.info jvm_info : 프로메테우스는 . 대신에 _ 포멧을 사용한다. . 대신에 _ 포멧으로 변환된 것을 확인할 수 있다.
* logback.events logback_events_total : 로그수 처럼 지속해서 숫자가 증가하는 메트릭을 카운터라 한다. 프로메테우스는 카운터 메트릭의 마지막에는 관례상 _total 을 붙인다.
* http.server.requests 이 메트릭은 내부에 요청수, 시간 합, 최대 시간 정보를 가지고 있었다. 프로메테우스에서는 다음 3가지로 분리된다.
  * http_server_requests_seconds_count : 요청 수
  * http_server_requests_seconds_sum : 시간 합(요청수의 시간을 합함)
  * http_server_requests_seconds_max : 최대 시간(가장 오래걸린 요청 수)
* 대략 이렇게 포멧들이 변경된다고 보면 된다. 포멧 변경에 대한 부분은 진행하면서 자연스럽게 알아보자.

### 프로메테우스 - 수집 설정
이제 프로메테우스가 애플리케이션의 /actuator/prometheus 를 호출해서 메트릭을 주기적으로 수집하도록 설정 해보자.
프로메테우스 폴더에 있는 prometheus.yml 파일을 수정하자.
prometheus.yml
```yaml
scrape_configs:
  - job_name: "prometheus"
  static_configs:
    - targets: ["localhost:9090"]
  #추가
  - job_name: "spring-actuator"
    metrics_path: '/actuator/prometheus'
    scrape_interval: 1s
    static_configs:
    - targets: ['localhost:8080']
```
* 앞의 띄어쓰기 2칸에 유의하자
* job_name : 수집하는 이름이다. 임의의 이름을 사용하면 된다.
* metrics_path : 수집할 경로를 지정한다.
* scrape_interval : 수집할 주기를 설정한다.
* targets : 수집할 서버의 IP, PORT를 지정한다


#### 프로메테우스 연동 확인
프로메테우스 메뉴 Status Configuration 에 들어가서 prometheus.yml 에 입력한 부분이 추가되어 있는지 확인해보자.
http://localhost:9090/config
프로메테우스 메뉴 Status Targets 에 들어가서 연동이 잘 되었는지 확인하자.
http://localhost:9090/targets

* prometheus : 프로메테우스 자체에서 제공하는 메트릭 정보이다. (프로메테우스가 프로메테우스 자신의 메트릭을 확인하는 것이다.)
* spring-actuator : 우리가 연동한 애플리케이션의 메트릭 정보이다.
* State 가 UP 으로 되어 있으면 정상이고, DOWN 으로 되어 있으면 연동이 안된 것이다

### 프로메테우스 - 기본 기능
이번에는 프로메테우스를 사용하는데 필요한 간단한 기능들을 알아보자.
검색창에 http_server_requests_seconds_count 를 입력하고 실행해보자
![스크린샷](/src/main/resources/img/screenshot5.png)
* 태그, 레이블: error , exception , instance , job , method , outcome , status , uri 는 각각의 메트릭 정보를 구분해서 사용하기 위한 태그이다. 
  마이크로미터에서는 이것을 태그(Tag)라 하고, 프로메테우스에서는 레이블(Label)이라 한다. 여기서는 둘을 구분하지 않고 사용하겠다.
* 숫자: 끝에 마지막에 보면 132 , 4 와 같은 숫자가 보인다. 이 숫자가 바로 해당 메트릭의 값이다.

기본 기능
* Table Evaluation time 을 수정해서 과거 시간 조회 가능
* Graph 메트릭을 그래프로 조회 가능

### 필터
레이블을 기준으로 필터를 사용할 수 있다. 필터는 중괄호( {} ) 문법을 사용한다.
레이블 일치 연산자
* = 제공된 문자열과 정확히 동일한 레이블 선택
* != 제공된 문자열과 같지 않은 레이블 선택
* =~ 제공된 문자열과 정규식 일치하는 레이블 선택
* !~ 제공된 문자열과 정규식 일치하지 않는 레이블 선택
예)
* uri=/log , method=GET 조건으로 필터
  * http_server_requests_seconds_count{uri="/log", method="GET"}
* /actuator/prometheus 는 제외한 조건으로 필터
  * http_server_requests_seconds_count{uri!="/actuator/prometheus"}
* method 가 GET , POST 인 경우를 포함해서 필터
  * http_server_requests_seconds_count{method=~"GET|POST"}
* /actuator 로 시작하는 uri 는 제외한 조건으로 필터
  * http_server_requests_seconds_count{uri!~"/actuator.*"}

### 연산자 쿼리와 함수
다음과 같은 연산자를 지원한다.
* + (덧셈)
* - (빼기)
* * (곱셈)
* / (분할)
* % (모듈로)
* ^ (승수/지수)

#### sum
값의 합계를 구한다.   
예) sum(http_server_requests_seconds_count)

#### sum by
sum by(method, status)(http_server_requests_seconds_count)  
SQL의 group by 기능과 유사하다.
결과
```json
  {method="GET", status="404"} 3
  {method="GET", status="200"} 120 
 ```
#### count
count(http_server_requests_seconds_count)    
메트릭 자체의 수 카운트  
#### topk
topk(3, http_server_requests_seconds_count)  
상위 3개 메트릭 조회  

#### 오프셋 수정자
http_server_requests_seconds_count offset 10m  
offset 10m 과 같이 나타낸다. 현재를 기준으로 특정 과거 시점의 데이터를 반환한다.
  
#### 범위 벡터 선택기
http_server_requests_seconds_count[1m]  
마지막에 [1m] , [60s] 와 같이 표현한다. 지난 1분간의 모든 기록값을 선택한다.  
참고로 범위 벡터 선택기는 차트에 바로 표현할 수 없다. 데이터로는 확인할 수 있다.  
범위 벡터 선택의 결과를 차트에 표현하기 위해서는 약간의 가공이 필요한데, 조금 뒤에 설명하는 상대적인 증가 확인 방법을 참고하자.  

### 프로메테우스 - 게이지와 카운터
메트릭은 크게 보면 게이지와 카운터라는 2가지로 분류할 수 있다.

#### 게이지(Gauge)
임의로 오르내일 수 있는 값  
예) CPU 사용량, 메모리 사용량, 사용중인 커넥션  

#### 카운터(Counter)
단순하게 증가하는 단일 누적 값   
예) HTTP 요청 수, 로그 발생 수  

#### 게이지(Gauge)
게이지는 오르고 내리고 하는 값이다. 게이지는 현재 상태를 그대로 출력하면 된다.
![screenshot](/src/main/resources/img/screenshot6.png)
예를 들어서 대표적인 게이지인 CPU 사용량( system_cpu_usage )을 생각해보자.  
CPU 사용량의 현재 상태를 계속 측정하고 그 값을 그대로 그래프에 출력하면 과거부터 지금까지의 CPU 사용량을 확인할 수 있다.  
게이지는 가장 단순하고 사용하기 쉬운 메트릭이다. 크게 고민하지 않고 있는 그대로를 사용하면 된다.  

#### 카운터(Counter)
카운터는 단순하게 증가하는 단일 누적 값이다. 예를 들어서 고객의 HTTP 요청수를 떠올려 보면 바로 이해가 될 것이다.
HTTP 요청을 그래프로 표현해보자.

계속 증가하는 그래프  
예) http_server_requests_seconds_count{uri="/log"}  
![screenshot](/src/main/resources/img/screenshot7.png)
* 02:42 ~ 02:43: 80건 요청
* 02:43 ~ 02:46: 0건 요청
* 02:46 ~ 02:48: 약 50건 요청

### increase()
increase() 를 사용하면 이런 문제를 해결할 수 있다. 지정한 시간 단위별로 증가를 확인할 수 있다.  
마지막에 [시간] 을 사용해서 범위 벡터를 선택해야 한다.  
예) increase(http_server_requests_seconds_count{uri="/log"}[1m])  

#### 시간 단위 요청 그래프
![screenshot](/src/main/resources/img/screenshot8.png)
* 02:42 ~ 02:43: 80건 요청
* 02:43 ~ 02:46: 0건 요청
* 02:46 ~ 02:48: 약 50건 요청

### rate()
범위 백터에서 초당 평균 증가율을 계산한다.  
increase() 가 숫자를 직접 카운트 한다면, rate() 는 여기에 초당 평균을 나누어서 계산한다.  
rate(data[1m]) 에서 [1m] 이라고 하면 60초가 기준이 되므로 60을 나눈 수이다.  
rate(data[2m]) 에서 [2m] 이라고 하면 120초가 기준이 되므로 120을 나눈 수이다.  

![screenshot](/src/main/resources/img/screenshot9.png)

### irate()
rate 와 유사한데, 범위 벡터에서 초당 순간 증가율을 계산한다. 급격하게 증가한 내용을 확인하기 좋다. 
자세한 계산 공식은 공식 메뉴얼을 참고하자.

![screenshot](/src/main/resources/img/screenshot10.png)

정리
게이지: 값이 계속 변하는 게이지는 현재 값을 그대로 그래프로 표현하면 된다.  
카운터: 값이 단조롭게 증가하는 카운터는 increase() , rate() 등을 사용해서 표현하면 된다. 이렇게 하면 카운터  
에서 특정 시간에 얼마나 고객의 요청이 들어왔는지 확인할 수 있다.  

> 참고
> 더 자세한 내용은 다음 프로메테우스 공식 메뉴얼을 참고하자  
> 기본기능: https://prometheus.io/docs/prometheus/latest/querying/basics/  
> 연산자: https://prometheus.io/docs/prometheus/latest/querying/operators/  
> 함수: https://prometheus.io/docs/prometheus/latest/querying/functions/  

### 그라파나 - 설치
https://grafana.com/grafana/download  

### 실행
http://localhost:3000  
email or username: admin  
Password: admin  
그 다음에 Skip을 선택하면 된다.  

### 그라파나 - 대시보드 만들기
이번시간에는 그라파나를 사용해서 주요 메트릭을 대시보드로 만들어보자  
먼저 다음 3가지를 꼭 수행해두어야 한다.  
* 애플리케이션 실행
* 프로메테우스 실행
* 그라파나 실행

### 대시보드 만들기
#### 대시보드 저장
* 왼쪽 Dashboards 메뉴 선택
* New 버튼 선택 New Dashboard 선택
* 오른쪽 상단의 Save dashboard 저장 버튼(disk 모양) 선택
* Dashboard name: hello dashboard를 입력하고 저장

#### 대시보드 확인
* 왼쪽 Dashboards 메뉴 선택
* 앞서 만든 hello dashboard 선택

### 패널 만들기
#### 대시보드에 패널 만들기
대시보드가 큰 틀이라면 패널은 그 안에 모듈처럼 들어가는 실제 그래프를 보여주는 컴포넌트이다.
* 오른쪽 상단의 Add panel 버튼(차트 모양) 선택
* Add a new panel 메뉴 선택
* 패널의 정보를 입력할 수 있는 화면이 나타난다.
* 아래에 보면 Run queries 버튼 오른쪽에 Builder , Code 라는 버튼이 보이는데, Code 를 선택하자.
* Enter a PromQL query... 이라는 부분에 메트릭을 입력하면 된다.


정리
지금까지 CPU 사용량, 디스크 사용량 메트릭을 대시보드에 추가했다.     
이제 앞서 학습한 다음과 같은 메트릭을 하나하나 추가하면 된다.      
* JVM 메트릭
* 시스템 메트릭
* 애플리케이션 시작 메트릭
* 스프링 MVC
* 톰캣 메트릭
* 데이터 소스 메트릭
* 로그 메트릭
* 기타 메트릭

### 그라파나 - 공유 대시보드 활용
다음 사이트에 접속하자  
https://grafana.com/grafana/dashboards  
이미 누군가 만들어둔 수 많은 대시보드가 공개되어 있다. 우리는 스프링 부트와 마이크로미터를 사용해서 만든 대시보드를 가져다가 사용해보자  
검색창에 spring이라고 검색해보면 다양한 대시보드를 확인할 수 있다.  

### 스프링 부트 시스템 모니터 대시보드 불러오기
https://grafana.com/grafana/dashboards/11378-justai-system-monitor/  
사이트에 접속한 다음에 Copy Id to clipboard 를 선택하자. 또는 ID: 11378 이라고 되어 있는 부분의 숫자를 저장하자
대시보드 불러오기
그라파나에 접속하자
* 왼쪽 Dashboards 메뉴 선택
* New 버튼 선택 Import 선택
* 불러올 대시보드 숫자( 11378 )를 입력하고 Load 버튼 선택
* Prometheus 데이터소스를 선택하고 Import 버튼 선택

### 마이크로미터 대시보드 불러오기
다음 대시보드도 유용한 많은 정보를 제공한다. 이 대시보드도 추가해서 사용하자.  
https://grafana.com/grafana/dashboards/4701-jvm-micrometer/

### 그라파나 - 메트릭을 통한 문제 확인
애플리케이션에 문제가 발생했을 때 그라파나를 통해서 어떻게 모니터링 하는지 확인해보자.
실제 우리가 작성한 애플리케이션에 직접 문제를 발생시킨 다음에 그라파나를 통해서 문제를 어떻게 모니터링 할 수 있
는지 확인해보자.
실무에서 주로 많이 발생하는 다음 4가지 대표적인 예시를 확인해보자.
* CPU 사용량 초과
* JVM 메모리 사용량 초과
* 커넥션 풀 고갈
* 에러 로그 급증

TrafficController.java 참고
실행
http://localhost:8080/cpu
http://localhost:8080/jvm
http://localhost:8080/jdbc (10번 이상 실행하자)
http://localhost:8080/error-log (여러번 실행하자)