---
date: '2023-07-02 19:29:00 +09:00'
lastmod: '2023-11-24 20:18:00 +09:00'
group: blog
image: /images/posts/java/virtual-thread/project-loom-logo.png
tags: ["java", "jdk 21", "virtual thread", "throughput", "benchmark"]
title: "Virtual Thread란 무엇일까? (2)"
url: /2023/07/02/java-virtual-threads-2
type: post
summary: "이전 글에 이어서 Virtual Thread에 대해서 알아보았다. 성능 테스트를 수행해보고, 사용시 주의사항, 그리고 Virtual Thread를 사용하는데 제약사항에 대해서 살펴보았다."
목적 : "Virtual Thread를 소개하고 기본적인 사용방법을 정리하여 공유한다."
대상독자 : "Java 프로그래밍을 사용해보고, Thread에 대해서 기본적인 개념을 이해하고 있는 사람"
---

# Virtual Thread (2)

[이전글](/2023/04/17/java-virtual-threads-1) 에서 `가상스레드`에 대한 **배경**과, **목적**, **간단한 사용법**에 대해서 알아보았다. 
이번에는 자주 사용하는 Spring Boot 애플리케이션에서 `가상스레드`를 사용하는 방법과 기존 **스레드 풀** 방식에 비해서 실제로 처리량이 늘어나는지 확인해보았다. 
마지막으로 `가상 스레드`를 사용할 때 주의할 점도 정리해보았다. 이 글은 `가상 스레드`와 관련된 두 번째 글이다.

* 시리즈
    - [Virtual Thread란 무엇일까? (1)](/2023/04/17/java-virtual-threads-1)
    - [Virtual Thread란 무엇일까? (2)](/2023/07/02/java-virtual-threads-2)


## 스프링 부트에서 사용하기

먼저 **Spring Boot**에서 `가상 스레드`를 적용하는 방법을 살펴보기 전에 몇가지 알아두어야 할 것들이 있다.

**주의사항** 
1. JDK 21 은 2023.09.19 정식 릴리즈되었다. 
2. Gradle 버전은 8.4 버전 이상에서 JDK 21을 지원한다. 
3. Spring Boot 3.2 가 2023.11.23 정식 릴리즈되어 JDK 21을 지원한다.

**`가상 스레드`를 제대로 확인하려면 버전을 꼭 확인하기 바란다.**

### 적용방법 

- 생각보다 적용방법은 간단하다. Spring Boot 3.2 버전 부터는 `spring.threads.virtual.enabled` 옵션을 `true` 설정해주면 된다. [링크](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes#support-for-virtual-threads)
- 만약 3.2 버전 보다 낮은 버전을 사용중이라면 `가상 스레드` Executor Bean 을 등록해주면 된다. 이 Bean 은 Tomcat이 사용자의 요청(Request)을 처리하기 위해 스레드를 사용할 때
**플랫폼 스레드(OS 스레드)** 대신 `가상 스레드` 를 사용하게 한다.

```yaml
# application.yaml
spring:
  threads:
    virtual:
      enabled: true
```

Sprinb Boot 3.2 보다 낮은 버전 사용중이라면 아래와 같이 직접 Bean 을 등록해주면 된다.

```java
// Web Request 를 처리하는 Tomcat 이 Virtual Thread를 사용하여 유입된 요청을 처리하도록 한다.
@Bean
public TomcatProtocolHandlerCustomizer<?> protocolHandlerVirtualThreadExecutorCustomizer() 
{
  return protocolHandler -> {
    protocolHandler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
  };
}

// Async Task에 Virtual Thread 사용
@Bean(TaskExecutionAutoConfiguration.APPLICATION_TASK_EXECUTOR_BEAN_NAME)
public AsyncTaskExecutor asyncTaskExecutor() {
  return new TaskExecutorAdapter(Executors.newVirtualThreadPerTaskExecutor());
}
```

이렇게만 해주면 기존의 **플랫폼 스레드**를 사용하지 않고 `가상 스레드` 를 사용하게 된다.

그럼 이제 실제로 처리량이 좋아지는지 한번 확인해보자.

## 성능 테스트

### 테스트 환경

- Ubuntu 20
- Java 21 (sdkman)
- Gradle 8.4 build
- VM 인스턴스 머신 4 Core / 8 GiB memory 
- 별도의 mariadb instance (max connection size 151)
- Max heap 2G

```java

    @GetMapping("/")
    public String getThreadName() {
        // 단순히 스레드 이름을 반환 아무런 blocking 코드 없음  
        return Thread.currentThread().toString();
    }

    @GetMapping("/block")
    public String getBlockedResponse() throws InterruptedException {
        // Thread sleep 1초
        // 비지니스 로직 처리에 thread 가 blocking 되는 환경 가정
        Thread.sleep(1000);
        return "OK";
    }

    @GetMapping("/query")
    public String queryAndReturn() {
        // 쿼리 질의가 1초 걸린다고 가정
        return jdbcTemplate.queryForList("select sleep(1);").toString();
    }
    
```


### 시나리오
* 테스트는 3개의 API Endpoint 를 호출하였다. (**simple response**, **block response**, **sleep query**)
* 모든 API 응답이 `200 OK` 확인될 때까지 VU를 높여보았다.
* `200 OK` 가 유지되는 동안 `virtual thread` 와 `platform thread` 의 `throughput` 을 비교해보았다. 

### 결과

* Simple response 호출
| 구분              | throughput | virtual users  |   
|-----------------|:------:|:---------------:|
| Virtual Thread(1회차)  |   **24360.88**     |      3000       |
| Virtual Thread(2회차)  |   **24608.85**     |      3000       |
| Virtual Thread(3회차)  |   **24455.14**     |      3000       |
| Platform Thread(1회차)  |   **36085.42**     |      3000       |
| Platform Thread(2회차)  |   **36396.71**     |      3000       |
| Platform Thread(3회차)  |   **36107.85**     |      3000       |

* Thread.sleep(1000) - Blocking 호출
  | 구분              | throughput | virtual users  |   
  |-----------------|:------:|:---------------:|
  | Virtual Thread(1회차)  |   **2975.38**     |      3000       |
  | Virtual Thread(2회차)  |   **2979.87**     |      3000       |
  | Virtual Thread(3회차)  |   **2978.39**     |      3000       |
  | Platform Thread(1회차)  |   **199.78**     |      3000       |
  | Platform Thread(2회차)  |   **199.58**     |      3000       |
  | Platform Thread(3회차)  |   **199.6**     |      3000       |

* Sleep 이 걸려 있는 쿼리 호출 (Hikari connection pool - max 150)
  | 구분              | throughput | virtual users  |   
  |-----------------|:------:|:---------------:|
  | Virtual Thread(1회차)  |   **SQLTransientConnectionException**     |      3000       |
  | Virtual Thread(2회차)  |   **SQLTransientConnectionException**     |      3000       |
  | Virtual Thread(3회차)  |   **SQLTransientConnectionException**     |      3000       |
  | Platform Thread(1회차)  |   **149.26**     |      3000       |
  | Platform Thread(2회차)  |   **149.53**     |      3000       |
  | Platform Thread(3회차)  |   **149.53**     |      3000       |

### 결론

- Thread Blocking 이 발생하지 않는 경우 Platform Thread 가 더 처리량이 높다. (Virtual Thread Scheduling 을 위한 오버헤드의 영향으로 보인다.)
- Thread Blocking 이 발생하는 경우 Virtual Thread를 사용할 때가 처리량이 더 높다. 
- DB Query 에 대해서는 Virtual Thread를 사용할 때 **SQLTransientConnectionException** 이 발생했는데, Tomcat 이후로 로직이 넘어갔는데 DB Connection 을 얻으려다가 timeout (30s)이 발생하는 것으로 추정된다.
- DB Connection 과 같은 한정된 자원에 접근을 제한하려면 semaphores를 도입하는걸 고려해야할것 같다.
- 실제 produciton 코드는 테스트 환경과 다르고 구동 환경도 다르기 때문에 참고용임을 감안하더라도 `Virtual Thread` 가 `Platform Thread` 에 비해서 처리량이 늘어날 수 있다는 점을 확인했다. 

> `Virtual Thread` 사용시 기존 `Platform Thread` 보다 일정영역에서 처리량이 늘어나는 것을 확인할 수 있다.

## 주의사항

- `가상 스레드` 를 사용하여 높은 처리량을 얻으려면 이를 잘 사용해야한다.
- 막연하게 설정을 활성화 하고 처리량이 높아지기를 기대하면 안된다. 몇가지 주의사항을 살펴보자.

1. 기존 **스레드 풀**을 사용하지 말고, 개별 작업에 `가상 스레드` 를 할당하는 형태로 변경하자.
    ```java
    ourExecutor.submit(task1);
    ourExecutor.submit(task2);
    
    ===>>>>
    
    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    	executor.submit(task1);
    	executor.submit(task2);
    }
    ```

2. **ThreadLocals** 에 값비싼 객체를 캐싱하지 말자
  
   - `가상 스레드` 또한 자바 Thread 이므로 ThreadLocal을 지원한다. 
   - 기존에 **플랫폼 스레드**는 비싸기 때문에, 여러 작업 사이에 공유하는 형태로 개발해왔다 (**Thread Pool**). 
   - 그래서 기존에는 스레드 로컬 내부에 값비싼 객체를 캐시하는 것이 일반적인 패턴으로 활용되었다. 
   - 모든 작업이 해당 스레드의 객체를 공유하도록 유도하였다. 
   - 하지만 `가상 스레드`는 작업당 하나를 활용하는 것이 권장되며, 내부의 객체를 공유하지 않는다. 
   - 따라서 내부에 값비싼 객체를 캐싱하는 것은 도움이 되지 않는다. 
   - 오히려 `가상 스레드`가 예상보다 더 많은 메모리를 사용하게 만드는 주범이 된다.

    ```java
    static final ThreadLocal<SimpleDateFormat> cachedFormatter = 
    	ThreadLocal.withInitial(SimpleDateFormat::new);
    ...
    
    cachedFormatter.get().format(...);
    
    
    =====>>>>>>>
    
    static final DateTimeFormatter formatter = DateTimeFormatter....;
    ...
    formatter.format(...);
    
    ```

3. **synchronized** 키워드 사용시 주의가 필요하다. (Pinning 이슈)

    - **synchronized** 키워드를 사용한 코드 블럭 안에서 blocking IO작업을 수행하는 경우에는 `가상 스레드` 를 unmount 할 수 없어서 Carrier Thread(Platform Thread)까지 Blocking 되는 현상이 발생한다. (이를 pinning 이라고 지칭함) 
    - 이런 경우에는 `가상 스레드` 의 이점을 누릴 수가 없다.
    - **synchronized**가 필요한 경우 자바의 동시성 유틸리티에 있는 **lock** 을 사용하자. 이렇게 되면 pinning의 영향에서 벗어날 수 있다.
    - 이런 제약은 현재 개선작업이 진행중이긴 하지만 JDK21에서는 이를 주의해야한다. (JEP 425 에서 **synchronized** 키워드를 사용해도 쓰레드가 pinning 되지 않도록 개선하고 있다.)
    - pinning 이 발생하는지 탐지하려면 JFR을 사용하거나 `-Djdk.tracePinnedThread` 옵션을 사용하면 pinning을 탐지할 수 있다.


    ```java
    synchronized(lockObj) {
    	frequentIO();
    }
    
    =====>>>>
    
    lock.lock();
    try {
    	frequentIO();
    } finally {
    	lock.unlock();
    }
    ```

## 정리

### 요약
- 지금까지 JDK21 (LTS)에 추가된 `가상 스레드`에 대해서 알아보았다.
- `가상 스레드` 는 리액티브 프로그래밍과 동일한 결과를 좀 더 쉽게, 덜 장황하게 달성한다.
- `가상 스레드` 가 더 좋은 이유는 **기다림**에 대한 방식이 개선되기 때문이다.
- `가상 스레드`는 기존의 플랫폼 스레드(전통적인 스레드)를 대체하려는 것이 아니며 둘다 사용이 가능하다.
- `가상 스레드`를 사용시 **처리량**을 증가시킬 수 있다.
- Spring Boot 3.2 에서 JDK21과 호환 작업이 적용되었다. Sprinb Boot 버전업을 진행하면 기존 코드 그대로 사용하면서 혜택을 누릴 수 있을 것이다.
- Profject Loom 의 결과물은 `가상 스레드`만 있는 것은 아니다. 앞으로 추가 JEP가 더 개발될 예정이다.

### 소감

`가상 스레드`가 아무리 좋아보여도 실제 production 에 적용되기 까지는 시간이 필요할 것이다. Spring Boot 의 버전업과 기존에 다양한 라이브러리들이 호환 작업을 진행하고 또 이런 내용들이 안정화 될 때까지 시간이 필요하기 때문이다.

`가상 스레드`는 [은빛 총알](https://en.wikipedia.org/wiki/No_Silver_Bullet)이 아니다. 막연하게 적용만 하면 처리량이 늘어날 것을 기대하면 안된다. 잘 알고 사용하고 또 한계점에 대해서 인지해야한다. 

약간 아쉬운 부분 중 하나는 `가상 스레드`는 아직 추가 JEP들의 도움을 받아야 기술이 성숙해질 것 같다는 점이고, 다른 하나는 Golang 의 **고루틴** 과 같은 완전한 경량 스레드는 아니라는 점이다.

그렇지만 5년내내 묵묵히 개발을 진행해온 `Project Loom` 개발팀에 박수를 보내며 앞으로 추가로 릴리즈될 JEP를 기대해본다. 👏👏👏👏👏👏

(+코틀린에서의 지원, 코루틴과 궁합은 또 어떻게 될지..?)

* 시리즈
    - [Virtual Thread란 무엇일까? (1)](/2023/04/17/java-virtual-threads-1)
    - [Virtual Thread란 무엇일까? (2)](/2023/07/02/java-virtual-threads-2)

## 참고자료
 - https://www.youtube.com/watch?v=YQ6EpIk7KgY
 - https://www.youtube.com/watch?v=n8uGsc4y6W4
 - https://medium.com/@zakgof/a-simple-benchmark-for-jdk-project-looms-virtual-threads-4f43ef8aeb1
 - https://medium.com/naukri-engineering/virtual-thread-performance-gain-for-microservices-760a08f0b8f3
 - https://perfectacle.github.io/2022/12/29/look-over-java-virtual-threads/
 - https://blog.devgenius.io/spring-boot-3-with-java-19-virtual-threads-ca6a03bc511d
 - https://howtodoinjava.com/java/multi-threading/virtual-threads/
 - https://www.linkedin.com/pulse/virtual-threads-java-any-benefit-all-use-cases-arvind-kumar/
 - https://www.youtube.com/watch?v=zluKcazgkV4&ab_channel=KotlinbyJetBrains
