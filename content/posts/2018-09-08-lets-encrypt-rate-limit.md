---
date: '2018-09-08 15:26:07 +09:00'
group: blog
image: /images/posts/letsencrypt/letsencrypt.png
tags:
- "let's encrypt"
- "letsencrypt"
title: Let's encrypt 의 인증서를 생성할 때 주의사항
url: /2018/09/08/lets-encrypt-certificates-rate-limit
type: post
---

Let's encrypt를 활용해서 SSL 인증서를 생성할 때에는 몇가지 주의해야할 사항이 있다. 인증서를 무한대로 생성할수 없는 것이 당연하고, 생성에 대한 제약사항을 정리해보았다.

<!--more-->

### Let's encrypt?

[Let's encrypt](https://letsencrypt.org/) 는 https 접속을 지원하기 위한 인증서를 무료로 생성할 수 있는 서비스라고 생각하면 이해하기 쉽다. 개인 블로그의 경우 인증서를 직접 구매하기 어려운데,
let's encrypt를 사용하면 무료로 인증서를 발급 받을 수 있다. [모질라](https://www.mozilla.org/), [아카마이](https://www.akamai.com/), [시스코](https://www.cisco.com/),
[크롬](https://www.google.com/chrome/), [페이스북](https://www.facebook.com/), [automattic](https://automattic.com/) 과 같은 회사들이 스폰서로 있어서 서비스가 지원을 중단할 걱정도 없다고 할 수 있다.
이 블로그의 경우에는 [github page](https://pages.github.com/) 에서 제공해주는 ssl 서비스를 사용하고 있는데 여기에서 사용하는 인증서도 let's encrypt를 사용한 것이다.
국내에서는 [티스토리](https://www.tistory.com)에서 개인도메인에 대한 ssl 인증서를 let's encrypt를 사용해서 지원하고 있다.

### 인증발급의 각 단계

인증서는 발급/갱신/취소 의 세가지 단계가 있다. 발급은 새로운 인증서 파일을 생성하는 것, 갱신은 생성된 인증서의 유효기간이 30일 이하로 남은 경우 이를 새롭게 90이상 사용가능하도록 하는 것,
마지막으로 취소는 생성한 인증서 파일을 유효하지 않게 하는 동작이다.

### 생성된 인증서의 유효기간

let's encrypt를 통해서 생성한 인증서는 기본적으로 3개월(90일)동안 사용할 수 있다. 일반적으로 사설 업체를 통해서 구매한 인증서가 최소 1년이라는 것을 감안하면 기간이 짧다고 할 수 있다.
그렇지만 재갱신 또한 무료로 가능하기 때문에, 제 때에 갱신하는 것을 깜빡하지(?) 않다면, 인증서를 계속 갱신해서 https 접속을 유지할 수 있다.

### 인증서를 발급 할 때 제약사항

인증서를 발급하는 방법은 자료가 많으니, 인증서를 발급할 때 제약사항에 대해서 알아보자. 일단 [공식 사이트](https://letsencrypt.org/docs/rate-limits/) 에서 자세하게 나와 있긴 한데
처음에는 뭐가뭔지 헷갈린다. 이를 정리해 보았다.

1. 인증서는 하나의 `Registered Domain` 기준으로 1주에 50개까지 인증서 발급이 가능하다.

    `Registered Domain` 이라는 것은 일반적으로 우리가 도메인을 신규로 구매할 때 결정되는 도메인이라고 생각하면 된다.
"www.example.com" 에서 `Registered Domain` 도메인은 "example.com" 부분을 말한다.
만약 서브 도메인이 "new.blog.example.co.kr" 이라면 `Registered Domain` 은 "example.co.kr" 이 된다. `Registered Domain` 에 대한 판단은
[Public Suffix](https://publicsuffix.org/) 를 사용해서 판단된다.

2. 하나의 인증서는 최대 100개의 서브 도메인 인증 정보를 포함시킬 수 있다.

    하나의 인증서에는 단 하나의 도메인 인증 정보만 담을 수 있는 것은 아니다. 두개 이상의 도메인 인증 정보를 담을 수 있는데, (이러한 인증서를 멀티 도메인 인증서 라고 한다)
let's encrypt를 사용해서 발급하는 인증서 파일 하나에 담을 수 있는 도메인 인증정보는 파일당 최대 100개의 제한이 있다. 따라서 앞서 첫번째 제한사항과 함께 고려한다면,
일주일에 최대 5000개의 서브 도메인에 대한 인증정보를 생성할 수 있다.

3. 동일한 도메인에 대한 인증정보 그룹(domain set)은 일주일에 최대 5개까지 발급이 가능하다.

    인증서를 생성할 때에는 동일한 도메인 인증정보 그룹(domain set)에 대해서는 일주일에 최대 5개까지 발급이 가능하다.
인증정보 그룹(domain set)이라는 것은 ["www.example.com", "example.com"] 와 같이 한번에 인증서 발급을 위해서 요청하는 도메인들(하나 또는 그이상)의 집합이라고 할 수 있다.
일반적으로는 하나의 도메인에 대해서 인증서를 발급 요청하기 때문에 이 제약사항은 그냥 동일한 도메인에 대해서는 인증서 발급은 일주일에 5번으로 제한된다고 이해하면 된다.
만약 인증정보 그룹(여러개의 도메인에 대한 인증서)이 동일한 인증서 발급 요청을 한다면, 일주일 동안에는 최대 5개까지만 가능하다는 의미다. 단, 인증정보 그룹을 추가한다면 추가적으로 인증서 발급이 가능하다
일주일 사이에 ["www.example.com", "example.com"] 으로 5번 인증서를 생성했더라도, ["www.example.com", "example.com", "blog.example"] 으로 인증서를 추가적으로 생성할 수 있다.

### 인증서를 갱신할 때 제약사항

인증서를 생성할 때만 제약이 있는 것은 아니고, 인증서를 갱신할 때에도 제약사항이 있다.

1. 생성 limit 에 해당되더라도, 갱신은 가능하다.

    만약 일주일에 50개의 인증서를 발급하여, 생성 limit에 해당되더라도, 기존에 생성한 인증서의 갱신(renewal)은 가능하다. 따라서 일단 인증서를 생성했다면, 갱신하는데는 문제가 없다.
(그렇지만, 언제나... 유저 불량이 발생한다.)

2. 갱신을 하기 전에 만료 기간에 해당하는지 확인해야한다.

    생성된 인증서는 90일 동안 유효하고, 30일이 남은 시간 부터 갱신이 가능하다, 만약 그 이전에 동일한 인증정보 그룹(domain set)으로 갱신 요청을 한다면,
이때는 동일한 도메인 인증서 발급 요청으로 취급되어, 동일한 도메인 인증정보 발급의 제약을 받는다. 그러니, 갱신을 하기전에. 만료기간에 해당하는지 확인하고 요청을 하도록 하자.

3. 인증서 갱신을 활용하면 사용가능한 하나의 도메인의 서브 도메인별 인증서는 계속 늘어날 수 있다.

    인증서는 기본적으로 생성에 제약이 있고, 갱신에는 제약이 없기 때문에 이를 잘 활용한다면, 1주에 50개씩 (하나의 도메인씩 늘려간다고 가정할 때) 계속 사용가능한 인증서를 늘려갈 수 있다.
만약 하나의 도메인에 여러개의 서브도메인을 모두 인증서를 발급해야 한다면, 이 정책을 사용하면 시간이 걸릴지언정, 언젠가는 모두 인증서 발급이 가능하다.

### 인증서 취소의 참고사항

1. 인증서 취소(revoke)는 인증서 생성 limit 을 초기화 시키지 않는다

    인증서 생성단계에서 어떠한 제약사항에 해당되어(주로 동일한 도메인 인증서를 계속해서 생성하는 경우) 기존 인증서를 취소(revoke)하면 limit count가 초기화되거나, 감소되지 않는지
궁금할 수도 있다. 공식 사이트에서는 명시적으로 인증서 취소는 limit count를 초기화 시키지 않는다고 알려주고 있다. 그러니, 대부분은 그냥 기다리는게 상책이다.

### 인증서 발급 실패와 관련된 제약사항

1. 계정별, 호스트별, 시간당 5번의 실패까지만 허용된다.

  호스팅 업체, 대규모 도메인 등록 서비스를 진행하는 경우, 또는 개인 이더라도 인증서 발급을 자동화하도록 구현하는 과정에서 여러가지 테스트를 수행하게 된다.
이 때, 잘못하면 fail vadliation 제약에 해당되어서 한 시간 동안 기다려야 되는 불행한(!) 사태가 발생한다. 그러니, 인증서 발급을 요청하기 전에, DNS 설정은 잘 되었는지,
"/.well-known/acme-challenge/" 경로는 잘 허용되었는지(발급을 위한 validation 과정은 방식에 따라 차이가 있을 수 있다) 확인을 잘 하고 발급 요청을 진행해야 한다.

2. "new-reg", "new-authz", "new-cert" API는 초당 최대 20개까지 허용된다.

   일반 사용자는 대부분 고려할 필요가 없는 사항인데, 대규모 인증서 발급이 필요한 경우에는 해당 API는 초당 20개까지 가능하다는 점을 염두해둬야 한다.
(그렇지만 이렇게 까지 많은 API를 사용하려면 얼마나 많은 도메인을 관리해야 하는걸까..)

3. "/directory" "/acme" API는 초당 최대 40개까지 허용된다.

    위와 마찬가지로 해당 API는 초당 최대 40개까지 가능하다

### IP별 생성 제한

인증서를 발급하는데는 IP에 대한 제약사항도 존재한다.(이렇게 놓고보니, 참 많은것 같다.)

1. IP별로 인증서 생성을 위한 계정을 3시간에 10개까지 생성이 가능하다

    let's encrypt를 통해서 인증서를 생성하려면 계정이 필요한데 이 계정은 하나의 IP에서 3시간에 10개까지 생성이 가능하다

2. IP별로 인증서 생성은 3시간에 500개까지 가능하다

    하나의 IP에서 인증서 생성은 3시간에 500개까지 가능하다. 이걸 넘는 경우는 아마 대규모 호스팅 업체의 경우일 것이다.

### 제약에 대한 개인 경험

나의 경우에는 대부분 인증서 발급 자동화를 테스트 하다가 `실패 제한(fail validation)`에 해당되어서 한시간 기다려야 되는 경우를 빈번하게 경험했다.
또한 대규모 인증서 발급을 해본 경험으로는 let's encrypt client를 개발하다가(자동화 과정에서) IP 당 발급 제한(3시간에 500개)에 해당되는 경우가 많았다.
따라서 let's encrypt를 통해서 대규모로 사용하건/개인용으로 사용하건 제약사항을 잘 알고나서 사용해야 한다. 다행히도 요즘엔 직접 let's encrypt를 사용하기 보다는
다양한 툴들을 통해서 이러한 제약에 해당되지 않고도 테스트를 할 수 있게 잘 구성되어 있는 편이다(stage environment를 잘 활용하자).
그리고 제발 같은 도메인으로 인증서 신규 발급 계속하지 말자. (오늘도 여전히 RTFM)


### 참고

- https://letsencrypt.org/docs/rate-limits/
