---
date: '2023-03-25 15:49:05 +09:00'
group: blog
image: /images/posts/blockhound/BlockHound.png
tags: ["webflux", "blocking code", "blockhound"]
title: "BlockHound - Webflux를 사용할 때 Blocking 코드가 사용되고 있는지 검출하는 방법"
url: /2023/03/25/block-hound-for-webflux
type: post
summary: "Spring Webflux 기반의 애플리케이션을 작성할 때에는 모든 코드가 reactive 해야 최상의 성능(처리량)이 나온다. 그런데 일부 구간에서 blocking 코드가 존재하는지 일일이 눈으로 확인하기는 매우 어렵다. 그래서 이를 자동으로 검출해주는 `BlockHound` 를 사용하여 blocking 코드를 검출해보았다."
---

# 배경
Spring Webflux 기반의 애플리케이션을 작성할 때에는 모든 코드가 reactive 해야 최상의 성능(처리량)이 나온다.
이 말은 코드 사이에 blocking 코드가 존재한다면 원하는 대로 충분한 성능이 발휘되지 않는다는 뜻이다. 
그러나 작성한 코드에 blocking 코드가 존재하는지 확인하기 위해서 일일이 코드를 살펴볼 수는 없다. 그래서 별도의 도구가 필요하다.

# BlockHound
[BlockHound](https://github.com/reactor/BlockHound)는 webflux 에서 사용하는 reactor 팀에서 개발한 도구로 애플리케이션에서 blocking 코드가 작성되었는지 여부를 검출해주는 도구이다.
직접 작성한 코드 뿐만 아니라, 서드 파티 라이브러리에서 사용한 블로킹 코드도 전부 검출한다.

## 사용법

1. 의존성 추가
    먼저 의존성을 추가해주어야 한다.
    ```kotlin
    // BlockHound 의존성을 추가한다.
    // 기본적으로 이 도구는 test 수행시 사용되나, 나의 경우에는 blcking 코드 검출 작업을 위해서 implementation 의존성으로 추가했다.
    // 버전은 github 에서 알려주는 최신버전을 사용했다.
    dependencies {
      ...
      implementation("io.projectreactor.tools:blockhound:1.0.7.RELEASE")
      ...
    }
    ```
2. 애플리케이션에 적용
    ```kotlin
   // Application Main 함수에서 BlockHound 를 활성화 한다.
    fun main(args: Array<String>) {
        BlockHound.install()
        runApplication<MyApplication>(*args)
    }
    ```

## blocking 코드 검출

애플리케이션을 구동하고, blocking 코드가 실행되도록 하자. 나의 경우에는 http 요청을 처리하는 spring security filter 로직에서 blocking 코드가 존재했다.

```kotlin
reactor.blockhound.BlockingOperationError: Blocking call! java.io.RandomAccessFile#readBytes
    at java.base/java.io.RandomAccessFile.readBytes(RandomAccessFile.java)
    Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException:
Error has been observed at the following site(s):
    *__checkpoint ⇢ Handler my.example.app.controller.SampleApiController#blockingMethod(Continuation) [DispatcherHandler]
    *__checkpoint ⇢ org.springframework.security.web.server.authentication.AuthenticationWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.authorization.AuthorizationWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.authorization.ExceptionTranslationWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.savedrequest.ServerRequestCacheWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.context.SecurityContextServerWebExchangeWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.authentication.AuthenticationWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.authentication.AuthenticationWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.context.ReactorContextWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.header.HttpHeaderWriterWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.config.web.server.ServerHttpSecurity$ServerWebExchangeReactorContextWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ org.springframework.security.web.server.WebFilterChainProxy [DefaultWebFilterChain]
    *__checkpoint ⇢ HTTP GET "/api/v1/blocking-code-test" [ExceptionHandlingWebHandler]
```

작성한 코드에서 blocking 코드가 검출되면 이를 nonblock 으로 대체하는 작업을 진행한다. 만약 라이브러리에서 blocking 코드가 검출된다면
reactive 스타일을 지원하는지 확인하고 대체할 수 있는지 확인하였다. 

## 동작방식

BlockHound 가 활성화되면 JVM 활성화 되면 바이트 코드레벨을 조작하여 메서드 호출에 다음의 코드를 추가한다.
```java
// java.net.Socket
public void connect(SocketAddress endpoint, int timeout) {
reactor.blockhound.BlockHoundRuntime.checkBlocking(
"java.net.Socket",
"connect",
/*method modifiers*/
);
```

## 커스터마이징

도메인 로직 또는 의도적으로 추가한 인프라 코드가 아니라, 라이브러리들이 blocking 코드로 검출되어 에러를 유발시키는 경우가 있다.
이런 경우에는 직접적으로 관심있는 코드가 아니기 때문에 에러가 너무 많이 발생하므로 이를 의도적으로 허용하도록 설정할 수 있다.

대표적으로 Jackson 의 ObjectMapper 가 (`jackson-module-kotlin` 의존성) blocking 코드를 사용한다고 BlockHound 에러가 발생하는데,
이는 외부 IO를 사용하는 것이 아니기 때문에 허용가능한 수준으로 본다는 [github 이슈](https://github.com/FasterXML/jackson-module-kotlin/issues/315)도 있었다. 

## BlockHound 에서 allow 코드 등록 
```kotlin
// BlockHound 를 그냥 install 하지 않고 다음과 같이 허용할 대상을 지정해줄 수 있다.
BlockHound.install(
    BlockHoundIntegration { builder: BlockHound.Builder ->
        builder
        .allowBlockingCallsInside(ObjectMapper::class.qualifiedName, "readValue")
        .allowBlockingCallsInside(ObjectMapper::class.qualifiedName, "canSerialize")
    }
)

```

## 소감

Reactive 스타일의 코드가 아직 익숙하지 않은 상황에서 blocking 코드를 사용했는지 파악하기 어려운데, `BlockHound` 를 사용해서
내가 작성한 코드가 blocking 을 유발하는지 확인할 수 있어서 도움이 되었다. 하지만 여전히 reactive 스타일은 적응이 잘 되지 않는다. 🥲 

## 참고자료
- [NHN FORWARD 2020] 내가 만든 WebFlux가 느렸던 이유 - https://forward.nhn.com/2020/session/26
- BlockHound의 동작 방식 - https://blog.frankel.ch/blockhound-how-it-works/
- WebFlux의 개념 / Spring MVC와 간단비교 - https://devuna.tistory.com/108