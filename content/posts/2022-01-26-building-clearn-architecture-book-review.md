---
date: '2022-01-26 23:39:41 +09:00'
group: blog
image: /images/posts/book/get_your_hands_dirty_on_clean_architecture_kor.jpg
tags:
- "book"
- "architecture"
- "만들면서 배우는 클린 아키텍처"
title: "[북리뷰] 만들면서 배우는 클린 아키텍처"
url: /2022/01/26/book-review-get-your-hands-dirty-on-clean-architecture
type: post
---

스프링 애플리케이션을 작성하면서 항상 아키텍처에 적합한 패키지 구조가 어떤 형태인지에 대한 의문이 있었다. 그러던중 우연찮게 이 책을 읽고나서 많은 부분에서 인사이트를 얻을 수 있었다. 
특히 DDD 에서 말하는 도메인 객체를 설계할 때의 경계에 대한 고민을, 책에서 소개하는 "Hexagonal Architecture" 를 통해서 이해할 수 있었다.

<!--more-->

## 책 소개

이 책은 DDD 와 클린 아키텍처에서 언급되는 "Hexagonal Architecture" 에 대한 설명을 담고 있다. 기존의 전통적인 "Layered Architecture"를 살펴보고
이 아키텍처의 단점과 이를 개선하는 방법에 대해서 설명하고 있으며, 각각의 아키텍처 선택지에 대한 장단점을 이야기해주는 책이다.
가격은 1만 8천원으로, 내가 읽은 시점에는 yes24 평점이 없었지만, 상당히 만족하면서 읽은 책이다.

{{< imageFull src="/images/posts/book/get_your_hands_dirty_on_clean_architecture_kor.jpg" title="만들면서 배우는 클린 아키텍처" border="false" >}}

## 책을 읽은 이유

스프링 애플리케이션을 작성할 때 가장 궁금한 것중 하나는 **패키지 구조를 어떻게 구성하는 것이 좋을까**에 대한 의문이었다. 애플리케이션의 전체적인 설계의도가 잘 드러나는 구조가 적합하다고 생각했지만,
작성되는 클래스는 항상 비슷비슷한 구조를 띄고 있었다. 그리고 대부분은 시작부터 내가 참여하지 않고 레거시 프로젝트를 맡아서 유지보수 하기 마련이라 기존에 구성된 패키지 구조를 그대로 답습하게 되었다.
이 말은 즉 기존의 아키텍처에서 크게 벗어난 형태로 개선하기가 어려웠다는 뜻이기도 하면서, 어떤 아키텍처가 현재에 적합한지 알기가 어려웠다는 뜻이기도 하다. 
그러다가 2021년 새로운 팀에서 업무를 시작하면서 새로운 애플리케이션을 시작할 수 있는 기회가 생겼다. (이른바 first commit 을 내손으로!) 
이왕 시작하는 거 아키텍처에 신경쓰면서 잘 정돈된 패키지 구조를 가지고 싶었는데 막상 2022년 새해를 맞이하면서 10개월 가까이 작성한 코드를 돌아보니 기존의 레거시 프로젝트의 구조와 별반 다를게 없어보였다. 
그래서 좀 더 나은 구조는 어떤 것일까 궁금하여 책을 찾게되었고 마침 "만들면서 배우는 클린 아키텍처" 라는 책이 눈에 띄어 냉큼 구매해 보았다.

- 가장 적절한 패키지 구조는 무엇일까라는 호기심에서 찾게된 책
- 클린 아키텍처라는 말에 현혹됨
- 일단 가격이 싸고 얇다!

## 소감

일단 책이 150페이지도 되지 않아서 읽기에 부담이 없어서 좋았다. 얇은 분량 덕분에 가볍게 마음먹고 시작할 수 있었고 이 책과 같이 번역서의 경우 항상 번역 퀄리티 문제가 있었는데 이 책은 용어의 낯선 느낌이 덜해서 읽는데 방해되는 일이 적었다. 군데 군데 일부 용어들은 현장에서 자주 사용되는 용어로 병기되어 있어서 좋았다. 무엇보다도 전통적인 애플리케이션의 구조를 설명하면서 단계적으로 문제점과 개선점을 소개하고 있는데 바로 그 전통적인 애플리케이션 구조가 내가 작성하던 코드와 유사해서 문제점과 개선점이라는 부분에서 공감이 많이 되었다. 평소에 어떻게 하면 좀 더 나은 구조를 만들 수 있을까 고민했던 부분들에 대한 해답을 찾은 느낌이랄까? 
물론 책에서 설명하는 "Hexagonal Architecture"이 모든 상황에서의 정답은 아니기에 항상 적합하다고 볼수는 없지만, 그동안 피상적으로만 이해하고 있던 DDD, 클린 아키텍처에서 이야기 하는 내용들을 좀 더 잘 이해할 수 있겠다는 생각이 들었다.  

### 핵심적인 몇몇 내용들
 1. `UserService` 대신 `RegisterUserService` 를 작성하라
  : 이 부분은 상당히 공감이 많이 되었던 내용이었다. 특히나 고민없이 클래스를 만들었던 과거에 수많은 `Service`, `Manager`, `Handler` 를 양상해본 경험이 있어서 더욱더 깊은 반성이 되었던 내용이다.
    평소 객체지향의 제일 쉬운 접근은 클래스를 잘게 쪼개 보는 방법이라고 생각하는데, 이렇게 서비스 객체를 `UseCase` 에 특화해서 만드는게 좋다고 생각했다.
 2. `SRP` - 컴포넌트를 변경하는 이유는 오직 하나뿐이어야 한다.
  : "하나의 클래스는 하나의 책임을 가져야 한다"로 알고 있었는데, 생각해보니 책의 말대로 변경의 이유가 하나여야 한다고 이해하는 것이 더 적합하다고 생각이 들었다. 둘은 미묘하게 다른데 역할 또는 책임이라는 것을
    명시적으로 설명하기 어려운데 "변경 이유" 라고 정의하니 훨씬 더 잘 이해하기 쉽다고 느껴졌다.
 3. Hexagonal Architecture 의 패키지 구조 
  : 패키지 구조를 명시적으로 보여줌으로써, 각각의 레이어의 의존성 역전, CQRS, Domain-Jpa Mapping 구조까지 잘 보여주는 예제를 확인할 수 있었다 
   ```
account
├── adapter
│ ├── in
│ │ └── web
│ │     └── SendMoneyController.java
│ └── out
│     └── persistence
│         ├── AccountJpaEntity.java
│         ├── AccountMapper.java
│         ├── AccountPersistenceAdapter.java
│         ├── ActivityJpaEntity.java
│         ├── ActivityRepository.java
│         └── SpringDataAccountRepository.java
├── application
│ ├── port
│ │ ├── in
│ │ │ ├── GetAccountBalanceQuery.java
│ │ │ ├── SendMoneyCommand.java
│ │ │ └── SendMoneyUseCase.java
│ │ └── out
│ │     ├── AccountLock.java
│ │     ├── LoadAccountPort.java
│ │     └── UpdateAccountStatePort.java
│ └── service
│     ├── GetAccountBalanceService.java
│     ├── MoneyTransferProperties.java
│     ├── NoOpAccountLock.java
│     ├── SendMoneyService.java
│     └── ThresholdExceededException.java
└── domain
    ├── Account.java
    ├── Activity.java
    ├── ActivityWindow.java
    └── Money.java
   ```
 4. 매핑전략 설명 : 각각의 매핑전략을 설명하고 장단점을 설명한 부분이 좋았다. 
 5. 의식적으로 지름길 선택하기 : 책에서 설명한 대로 모든 의존성 역전구조와, 경계별로 나뉘어진 모델 객체를 사용하려면 작성할 클래스가 아주 많아지고 보일러플레이트 코드가 많아질 수 밖에 없다. 그래서 저자는 이부분을 적절하게 구분하여 의식적으로 경계를 느슨하게 할 수 있는 전략을 설명해주고 있는데, 이런 경계에 대한 인식없이 그냥 작성하는 것과 알고서 의식적으로 선택하는 것은 차이가 있다고 생각한다. 
  예를 들면 JPA Entity 를 RegisterUserService 의 응답 객체로 활용할 수 있는데 이 경우 어떤 부분이 문제가 되는지 설명해주고 그럼에도 이렇게 사용할 수 있다는 점을 설명해주었는데 이런 내용들이 공감이 많이 되었다.

## 아키텍처는 절대적이고 전역적이어야할까? 

책에서는 줄곧 "Hexagonal Architecture"를 기준으로 설명하고 있지만, 마지막까지 이야기 하는 내용을 요약하자면 다음 메세지가 아닐까 한다. 
> 아키텍터는 팀의 의사결정에 따라 결정된다. 이 때 결정되는 아키텍처는 한번 정하면 불변하는 것이 아니고, 지속적으로 개선(또는 선택)될 수 있다. 
> 그리고 모든 부분에 일관된 하나의 아키텍처(스타일, 코딩 컨벤션이든 무엇이든)를 적용하지 않아도 된다

결국 은빛 탄환은 없다.(No Silver Bullet)

## 덧붙이며
 이전에 읽은 POEAA(엔터프라이즈 애플리케이션 아키텍처 패턴)에서 도메인과 영속성 매핑에 관해서 강조했는데 코드가 너무 낡아서 이해하기가 어려웠었다. JPA 와 Domain 기반 코드를 작성하면서 이 책에서 예시로든
 PersistenceAdapter 가 매핑구조를 이해하는데 도움이 되었다.

## 한줄 요약

* "레이어의 경계를 고민하고 있다면 적절한 지침서가 되는 책"

### 참고
* yes24 링크 : https://www.yes24.com/Product/Goods/105138479
* 21년 6월의 책 - Get Your Hands Dirty On Clean Architecture : https://velog.io/@gyunghoe/21%EB%85%84-6%EC%9B%94%EC%9D%98-%EC%B1%85-Get-Your-Hands-Dirty-On-Clean-Architecture
* Get Your Hands Dirty on Clean Architecture 를 읽고 : https://namkyujin.com/post/20210401-get-your-hands-on-clean-arch/
* [Line engineering] 지속 가능한 소프트웨어 설계 패턴: 포트와 어댑터 아키텍처 적용하기 : https://engineering.linecorp.com/ko/blog/port-and-adapter-architecture/ 
