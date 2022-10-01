---
date: '2022-09-17 16:22:01 +09:00'
group: blog
image: /images/posts/spring/Spring_Framework_Logo_2018.png
tags: ["spring", "spring event", "event driven"]
title: "스프링 이벤트 기능을 사용할 때의 고려할 점"
url: /2022/09/17/points-to-consider-when-using-the-Spring-Events-feature
type: post
summary: "처음에는 간단하게 작성한 도메인 로직이 시간이 지나면서 여러가지 추가 로직이 늘어나 복잡해지는 경험을 해본적이 있다. 이런 코드를 리팩토링하면서 스프링의 `Event` 기능을 사용할 수 있는데 스프링 이벤트를 사용할 때 어떤 점을 고려해야하는지 정리해보았다."
---

# 배경

애플리케이션의 코드를 작성하다보면, 처음에는 간단하게 시작한 도메인 로직이 시간이 지나면서 여러가지 추가 로직이 늘어나 복잡해지는 경험을 하게된다.  
예를 들어서 velog, medium 과 같은 블로그 애플리케이션을 만든다고 가정하고 다음의 코드를 살펴보자. 

```kotlin
// 초기 코드
@Service
open class PostService {
    
    // ...

    @Transactional
    open fun savePost(post: Post) {
        // post 저장
        postRepository.save(post)
    }
}

// 시간이 지나고 나서

// 복잡해진 코드
@Service
open class PostService {
    // ...

    @Transactional
    open fun savePost(post: Post) {
        // 추천 시스템을 위한 콘텐츠 풀에 전송
        recommendPoolSender.send(post)
        
        // 통계 시스템의 카운팅
        statisticsCounter.count(post)
        
        // 보다 빠른 검색을 위한 ElasticSearch 검색엔진 연동
        elasticSearch.indexing(post)
        
        // post 저장
        postRepository.save(post)
    }
}
```

위의 코드는 상당한 비약을 섞어 놓은 코드이지만 예시로는 충분하다. 이런 로직들은 핵심 로직과 부가적인 코드가 묶여 있어서 쉽게 분리하기 어려울 때도 있다. 
위의 예제를 정리해보면 다음과 같은 비지니스 로직을 표현하고 있다. 

* 블로그의 포스트가 저장될 때
  - 추천 시스템을 위한 콘텐츠 풀 전송
  - 서비스 전체의 통계를 위한 카운팅 작업 수행
  - 서비스 전체의 글 검색을 위한 ElasticSearch 인덱싱
  
여기에서 핵심은 블로그의 포스트를 저장하는 로직이고 나머지는 이와 연관된 코드라고 정의할 수 있다. 이를 리팩토링하는데 스프링에서 제공하는 `Event` 기능을 사용할 수 있다.

# Spring Event

## 개요

Spring Event란 스프링 프레임워크를 사용할 때 내부에서 데이터를 전달하는 방법 중 하나이다. 이를 사용하면 각각의 코드의 관심사를 분리할 수 있다. 
스프링 이벤트 기능은 이벤트를 발생시키고(publish) 이벤트를 수신하는(subscribe)하는 로직을 분리해서 작성할 수 있다. 다음의 로직을 살펴보자.

```kotlin
@Service
open class PostService {
    // ...
    @Autowired
    private lateinit var applicationEventPublisher: ApplicationEventPublisher

    @Transactional
    open fun savePost(post: Post) {
        // post 저장
        postRepository.save(post)
        applicationEventPublisher.publishEvent(PostCreatedEvent(post))
    }
}

//이벤트 리스너. 일반 이벤트 리스너를 사용하여 핵심 로직과 부가로직을 분리시켰다.
@Component
class PostCreateEventListener {
    @Autowired
    private lateinit var recommendPoolSender: RecommendPoolSender

    @EventListener
    fun sendContentPool(event: PostCreatedEvent) {
        recommendPoolSender.send(event.post)
    }
    
    // ...
}
```

핵심은 스프링이 Event를 발생시키고 이를 처리하는 로직(Listener)들 사이에 데이터(Event)를 전달해주는 역할을 해줌으로써 개발자가 각각 분리된
코드를 작성할 수 있다는 것이다. 이를 통해서 초기의 핵심 로직인 블로그 포스트를 저장하는 로직은 간결하게 유지하고 이 이벤트가 발생했을 때 추가적으로 
처리해야하는 부가적인 코드들은 Listener 를 통해서 처리할 수 있기 때문에 하나의 Service 클래스(위 예시에서는 PostService) 안에 코드가 
계속적으로 증가하는 문제를 해결할 수 있다.

* 요약
  - PostService 는 Post 처리에 집중
  - 부가적인 코드들은 Event Listener 를 통해서 호출
  - 이벤트의 발생과 전달은 Spring 이 해결해줌

이런 구조를 흔히들 pub / sub 구조라고 이야기한다.

## 트랜잭션과 이벤트 처리

그런데 위의 코드는 한 가지 고민해보아야 할 문제가 있다. 바로 트랜잭션 안에서 Post 가 저장된다는 사실이다. 따라서 부가적인 코드(추천 연동, 검색 연동, 통계 연동..)는
트랜잭션이 성공적으로 수행된 이후에 실행되어야만 하는데, 위의 코드로는 트랜잭션의 성공적인 수행을 보장할 수 없다는 문제가 발생한다. 

이를 위해서 스프링에서는 `EventListener` 대신 `TransactionalEventListener` 제공한다. 말 그대로 트랜잭션 안에서 이벤트를 발생시킬 때 트랜잭션 처리와 결합하여
이벤트를 수신하는 로직을 처리할 수 있다. 따라서 위의 로직은 다음과 같이 변경할 수 있다.

```kotlin
// 이벤트 리스너. 트렌젝션 이벤트 리스너를 사용했다.
// 스프링이 이를 식별할 수 있도록 @Component 어노테이션을 붙여주어야 한다.
@Component
class PostCreateEventListener {
    @Autowired
    private lateinit var recommendPoolSender: RecommendPoolSender

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    fun sendContentPool(event: PostCreatedEvent) {
        recommendPoolSender.send(event.post)
    }
    
    // ...
}
```

`@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)` 어노테이션을 자세히 보면 `phase` 옵션이 있는데 이를 해석하자면
`AFTER_COMMIT` 즉 트랜잭션이 성공적으로 commit 된 이후에 리스너 로직을 처리하라는 의미가 된다. 

`phase` 값은 다음의 4가지 옵션을 지정할 수 있고, 일반적으로 `AFTER_COMMIT` 이 적용하기 적합한 경우가 많다.

  - AFTER_COMMIT (트랜잭션이 성공했을 때 실행)
  - AFTER_ROLLBACK (트랜잭션 롤백시 실행)
  - AFTER_COMPLETE 트랜잭션 완료시 (AFTER_COMMIT+AFTER_ROLLBACK)
  - BEFORE_COMMIT (트랜잭션 commit 되기전에)

결과적으로 `TransactionalEventListener` 어노테이션을 사용하면 원하는 대로 트랜잭션 처리를 보장하면서 로직을 분리해서 관리할 수 있게 된다.

하!지!만! 이 어노테이션의 `phase`를 `AFTER_COMMIT` 로 사용할 때 주의하지 않으면 문제가 발생할 수 있다. 

### 이벤트 리스너의 트랜잭션 처리 주의사항

`TransactionalEventListener`을 사용하여 이벤트 구조를 도입하여 간결한 코드 구조를 유지할 수 있어서 장점이 있지만, 트랜잭션과 함께 이벤트를 처리할 때 주의사항이 있다.
`phase` 값이 `AFTER_COMMIT` 으로 정의해놓은 경우 리스너 코드 안에서 다시 트랜잭션을 처리하면 해당 트랜잭션은 커밋되지 않는 현상이 발생한다.

```kotlin
// ...
  @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
  fun updateCounterForStatistics(event: PostCreatedEvent) {
      // 통계 시스템의 카운팅 갱신
      // 이 코드는 @Transactional 코드
      // 정상적으로 트랜잭션이 commit 되지 않는다.
      statisticsCounter.count(post)
    }
    
// ...
```

이 현상에 대한 원인은 TransactionSynchronization 의 afterCommit 주석부분에 설명되어 있다.

```
/**
* Invoked after transaction commit. Can perform further operations right
* <i>after</i> the main transaction has <i>successfully</i> committed.
* <p>Can e.g. commit further operations that are supposed to follow on a successful
* commit of the main transaction, like confirmation messages or emails.
* <p><b>NOTE:</b> The transaction will have been committed already, but the
* transactional resources might still be active and accessible. As a consequence,
* any data access code triggered at this point will still "participate" in the
* original transaction, allowing to perform some cleanup (with no commit following
* anymore!), unless it explicitly declares that it needs to run in a separate
* transaction. Hence: <b>Use {@code PROPAGATION_REQUIRES_NEW} for any
* transactional operation that is called from here.</b>
* @throws RuntimeException in case of errors; will be <b>propagated to the caller</b>
* (note: do not throw TransactionException subclasses here!)
**/
```   

요약하자면, 이전의 이벤트를 publish 하는 코드에서 트랜잭션이 이미 커밋 되었기 때문에 `AFTER_COMMIT` 이후에 새로운 트랜잭션을 수행하면 해당 데이터소스 상에서는 트랜잭션을 커밋하지 않는다는 것이다.
따라서 `@Transactional` 어노테이션을 적용한 코드에서 `PROPAGATION_REQUIRES_NEW` 옵션을 지정하지 않는다면 (매번 새로운 트랜잭션을 열어서 로직을 처리하라는 의미) 이벤트 리스너에서 트랜잭션에 의존한 로직을 실행했을 경우 
이 트랜잭션은 커밋되지 않는다.

{{< imageFull src="/images/posts/spring/you_activated_my_trap_card.png" class="medium-width" title="spring TransactionalEventListener" border="false" caption="매뉴얼을 안읽으면 이렇게 되기 쉽다.">}}

이를 해결할 수 있는 추가적인 방법이 필요하다.

### 문제 해결 방법

위 문제를 해결하는 첫 번째 방법으로는 `AFTER_COMMIT` 이후에 동일한 데이터소스를 사용하지 않는 방법이 있다. 이를 위해서는 이벤트 리스너를 별도의 스레드에서 실행하는 방법이 있다. 바로 `@Async` 어노테이션을 추가하는 방법이다.

```kotlin
// ...
  @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
  @Async
  fun updateCounterForStatistics(event: PostCreatedEvent) {
      // 통계 시스템의 카운팅 갱신
      // 이 코드는 @Transactional 코드
      // AFTER_COMMIT 이라서 이후의 트랜잭션은 커밋되지 않지만, 
      // @Async 어노테이션으로 인해서 별도의 스레드에서 처리하므로 커밋이 정상적으로 실행됨
      statisticsCounter.count(post)
    }
    
// ...
```

이렇게 하면 이벤트 리스너 로직이 별도의 스레드에서 실행되어 트랜잭션이 커밋되기 때문에 의도한 결과를 얻을 수 있다. 단 이렇게 하면 테스트코드를 작성하기는 좀 까다로울 수 있다.

다른 방법으로는 `AFTER_COMMIT` 대신 `BEFORE_COMMIT` 을 사용하는 방법이다. 이렇게 되면 커밋이 되기 전에 리스너 로직이 실행되기 때문에 정상적으로 리스너 로직의 트랜잭션이 커밋될 수 있다. 
하지만 이 경우 리스너 로직에서 예외가 발생하면 이벤트를 발생시키는 핵심 로직의 트랜잭션에 영향을 줄 수 있기 때문에 주의해서 사용해야한다.

## 결론

- 스프링 이벤트기능을 사용하여 복잡한 로직을 별도의 리스너 로직으로 분리해서 핵심 로직은 간결하게 유지할 수 있다. 
- 스프링 이벤트의 핵심은 이벤트의 발생(publish)와 이벤트의 처리(listener)를 연결해주는 역할이며 이런 구조를 pub/sub 구조로 이해할 수 있다.
- 개발자는 이벤트 listener 로직을 필요할 때마다 추가할 수 있으므로 핵심로직은 간결하게, 리스너로 분리된 메서드로 유지할 수 있어서 구조를 이해하기 용이해진다.  
- 트랜잭션과 함께 사용할 때는 `AFTER_COMMIT` 이 일반적으로 사용되지만, 리스너 로직에서 트랜잭션이 필요한 경우 `@Async` 등으로 우회할 수 있다. 다만 테스트가 불편해질 수 있다.

## 더 고민해볼 내용

스프링 이벤트를 사용하여 핵심 로직과 리스너 로직을 분리하고, 이벤트의 전달은 스프링 이벤트가 책임지게 하여 간결한 코드를 유지할 수 있게 되었다. 하지만 추가적으로 더 고민해보아야할 이슈들이 있다. 

### 이벤트 리스너 로직의 예외처리 및 재처리 

이벤트 리스너 로직을 수행하는데 예외가 발생하는 경우 분리된 구조로 인해서 핵심 로직에는 영향을 주지 않을 수 있다. 그래서 리스너 안에서 예외처리에 주의를 기울여야 한다. 그리고 리스너로 처리하는 로직이 
핵심로직과 함께 중요도가 높은 로직이라면 별도의 재시도 처리등을 수행해야 하는 경우가 발생한다. 이런 경우는 이벤트 리스너안에 복잡한 처리를 추가하기 어렵다. 
따라서 이런 경우라면 메세지 큐를 도입하여 이벤트 발생시 kafka 등으로 메세지를 보내고, 이에 대한 처리는 별도의 분리된 마이크로 서비스가 처리하게 하는 것이 나을 수도 있다.

### 메세지 큐 도입시 메세지 발행 실패에 대한 처리

만약 스프링 이벤트를 메세지 큐로의 메세지 발행을 처리하는 용도로 사용한다면, 이후에 연결되는 로직들은 메세지 큐에 의존적이게 되고, 이 메세지 큐는 트랜잭션과 함께 아주 중요한 인프라 자원이된다. 
그리고 트랜잭션과 함께 메세지 큐의 발행이 실패하면 안되게 되는데, 이 때 딜레마가 발생한다. 바로 DB 트랜잭션과 메세지 큐 발행이 동시에 성공해야하는데 둘 중 하나만 성공하는 경우에 대한 처리가 어렵다는 점이다. 
이런 경우라면 `Transactional outbox pattern` 같은 기법을 도입하여 이벤트의 발행 신뢰도를 높일 수도 있다. 다만 이렇게 까지 해야하는가에 대한 고민은
비지니스 요구사항과, 애플리케이션의 복잡성, 중요도등을 고려해서 결정해야한다.


# 참고자료
- Baeldung Srping event : https://www.baeldung.com/spring-events
- https://kwonnam.pe.kr/wiki/springframework/transaction/transactional_event_listener
- Transactional outbox pattern : https://microservices.io/patterns/data/transactional-outbox.html
