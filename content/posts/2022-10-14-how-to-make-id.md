---
date: '2022-10-14 23:26:22 +09:00'
group: blog
image: /images/posts/resource/id-generation.jpg
tags: ["database", "ulid", "uuid"]
title: "리소스의 고유한 식별자는 어떤 형식을 사용해야할까?"
url: /2022/10/14/resource-id-generation
type: post
summary: "서비스를 설계할 때 데이터베이스의 저장되는 리소스의 Key를 어떤 방식으로 생성해야하는지에 대한 고민을 정리해보았다. 찾아보면서 알게된 것이지만 생각보다 고려할 점이 많았다."
---

# 고유한 식별자를 가지는 리소스 

일반적인 웹 애플리케이션을 설계하다 보면 많은 Entity, 도메인 모델, 데이터베이스 테이블을 정의하게 된다. 이 때 특정 리소스가 고유한 값을 가져야 되는 요구사항은 
너무나도 자주 마주친다. 예를 들어 사용자 정보를 저장하기 위한 ID 라던지 (사용자가 입력하는 UserID와 구분한다.), 작성한 글의 번호 라던지, URL을 통해서 접근하는 대부분의 리소스의 구분자가 모두 
고유한 값을 가지고 있어야만 하는 경우라고 할 수 있다. 

이런 리소스를 구분하기 위해서는 고유한 식별자 (Identifier)를 가져야 하는에 이 식별자를 생성하는 방식은 여러가지가 있다. 

## Auto Incremental ID - Sequential

1. Auto Incremental ID란? 
   - 데이터베이스가 자동으로 생성해주는 number ID 이다. 
   - `mysql` 기준으로하면
     - int : -2147483648 ~ 2147483647 ( 4 바이트 )
     - bigint -9223372036854775808 ~ 9223372036854775807 (8바이트) 의 값을 가진다. 

2. 장점
   - 순차적인 값을 가지고 데이터베이스가 알아서 고유한 id 를 생성해주기 때문에 크게 고민하지 않고 사용할 수 있다.
   - 생성 순서대로 정렬하기 용이하다.

3. 단점
   * **외부 노출의 문제** : URL을 통해서 외부에 노출 되었을 때 서비스의 주요 지표가 드러나게 되는 상황이 발생할 수 있다. 
     예를 들어 사용자의 프로필을 조회하는 URL이 `https://myservice.app/users/12345`와 같이 표현된다면 경쟁사 입장에서는 전체 회원수가 몇명인지 금방 알아낼 수 있다.
     - 새로운 user가 등록될 때마다 id값도 1씩 오르기 때문인데 이를 회피하기 위해서는 auto incremental id 를 노출하지 않도록 신경써야하거나 다른 id 형식을 사용해야한다.
     - 불필요한 내부 정보 추정이 가능해지는 문제 이외에도 보안 취약점을 위한 스캔이나, 웹 공격 대상으로 지정되는 경우 타겟 리소스의 주소가 추정이 가능하기 쉬운 문제도 있다.
   * **JPA 쓰기 지연 활용 불가** : 만약 Spring JPA같은 기술을 사용한다면 실제 Identification Key 가 Insert 쿼리가 실행된 이후에 결정되버린다는 특징에 영향을 받는다.
     - 영속성관리 타이밍상 mysql 의 auto increment id 를 사용하면 IDENTITY 식별자 생성 전략(mysql의 auto-increment 등)은 어쩔 수 없이,
       쓰기 지연이 동작하지 않고 영속화 할 때 insert 쿼리가 먼저 데이터베이스로 나간다 (즉 JPA의 `쓰기 지연` 효과를 누릴 수 없다)

## UUID - Universally Unique Identifier

1. UUID란?
   - 문자형 ID
   - RFC4122에 정의되어 있는 ID 생성 방식이다.(https://tools.ietf.org/html/rfc4122)
   - 여러가지 버전이 있지만 그 중에서 UUID4를 많이 사용함 ex) `83fda883-86d9-4913-9729-91f20973fa52`
   - 128bit(16바이트)를 사용한다.

2. 장점 
   - 위의 Auto incremental ID가 순차적인 특성 때문에 가지는 단점을 극복할 수 있다.
     - 노출되더라도 서비스의 주요 지표가 노출되는 일이 없다.
     - JPA 기술을 사용할 때 소스코드레벨에서 ID를 결정할 수 있기 때문에 쓰기 지연을 활용할 수 있다.

3. 단점
   - 순차 정렬을 사용할 수 없다.
   - 128비트이기 때문에 int id 값보다 보다 4배나 크다. 
     - ID라는 특성상 참조하는 테이블이 많아질 수록 스토리지의 용량이 많이 필요하다. 
     - 이에 따라서 메모리 사용량이 커지고, 성능에 영향을 끼칠 수도 있다.
     - percona 블로그 포스트에 따르면 평균 2.5배, 최악의 경우, UUID가 Auto Increment에 비해 28배 느린 경우도 발생할 수 있다고 한다.
       인덱스 테이블에서는 기본키를 포인터로 해서 보조 인덱스가 저장되므로 5개의 보조 인덱스의 UUID기반 스키마의 경우
       총 6번 저장. 총 1억 개의 테이블에서 216G, 즉 스토리지의 70%를 차지할 수도 있음 (https://percona.com/blog/2019/11/2)

## 어떤 방식을 선택할 것인가에 대한 고민

각각의 장단점이 있기 때문에 어떤 방식을 선택할지는 상황에 맞게 결정해야한다. 다만 생각해볼 것은, 고민하지 않고 그냥 가장 익숙한 방식을 선택하지는 말자는 것이다.
ID가 외부에 노출되기 부담스러운 경우에는 auto incremental id 보다는 UUID 방식이 나을 수 있고, 그 반대라면 auto incremental id 가 더 나을 수도 있다.

## 추가적으로 생각해볼 것들

### 만약 아주 큰 사이즈의 데이터를 담게되는 경우

아주 많은 데이터를 저장해야하는 경우, 그러니까 경험적으로는 데이터의 갯수가 수천만건~1억건 사이에서 부터 데이터를 어떻게 분산해서 저장할지에 대한 고민이 필요해진다.
이럴 때에도 키를 어떤 방식을 사용했느냐에 따라고 고민이 달라질 수 있다. 

* 분산 데이터베이스에서의 키 관리 
  - spanner 와 같은 분산 데이터베이스를 고려한다면 구글의 아티클에서 권장하는 UUID 방식이 적합해보인다.
  - https://cloud.google.com/spanner/docs/schema-design#primary-key-prevent-hotspots

### Redis 에서 key 를 적극적으로 사용하는 경우 

redis와 같은 캐싱 시스템을 활용하는경우 엔티티의 id 값을 저장하게 되는데, 경우에 따라서는 bitmap 구조를 활용할 수도 있다. [리멤버 블로그 - 유저 목록을 Redis Bitmap 구조로 저장하여 메모리 절약하기](https://blog.dramancompany.com/2022/10/%EC%9C%A0%EC%A0%80-%EB%AA%A9%EB%A1%9D%EC%9D%84-redis-bitmap-%EA%B5%AC%EC%A1%B0%EB%A1%9C-%EC%A0%80%EC%9E%A5%ED%95%98%EC%97%AC-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%A0%88%EC%95%BD%ED%95%98%EA%B8%B0/)
그런데 이렇게 bitmap 을 활용하기 위해서는 id 구조가 number id 구조, 즉 auto incremental 인 경우를 가정해야한다. 따라서 UUID 방식이라면 bitmap을 활용한 목록 캐싱은 불가능하다.

## 다른 대안은 없을까?

ID를 어떤 방식으로 선택할 것인가에 대한 문제는 오랫동안 논의되어온 주제인 만큼 여러가지 대안들도 존재한다. 

### HashIDs (https://hashids.org/)

이 선택지의 주요 컨셉은 auto incremental ID를 선택했을 때 외부에 노출되는 것이 문제라면, 외부에 노출될 때 hashing 을 통해서 노출하자라는 것이다. 
예를 들면 ID가 12345 일 때 hashing 을 통해서 '3sy561e' 와 같이 변환해서 노출할 수 있다. 해싱에서는 salt 로 사용되는 값을 정할 수 있는데 
이 값을 관리하여 '3sy561e' 라는 값을 다시 12345 와 같이 변환할 수 있다. 따라서 이 값은 외부에 노출되면 안된다.

### Snowflake  

이 선택지의 주요 컨셉은 ID를 생성하는 것 자체가 독립적인 별도의 서비스를 사용해서 처리한다는 것이다. 트위터에서 선택하는 방식으로 알려져있다.
UUID에 비해서 절반인 64비트이며, 앞부분에 시간값을 입력하여 정렬도 가능하도록 설계되어 있다. discord, instagram에서 사용했다고 알려졌다. 
현재 github 주소는 아카이빙 처리되었다. (https://github.com/twitter/snowflake)

### ULID (https://github.com/ulid/spec) 

* Universally Unique Lexicographically Sortable Identifier
이 선택지의 주요 컨셉은 UUID의 단점을 개선하는데 초점이 맞춰져 있다. UUID 128비트 구조와 호환하면서, 정렬이 가능하며, 특수문자를 포함하지 않아 URL에서 사용해도 안전하다.
다양한 언어별 구현체가 있다. 

### nano id (https://github.com/ai/nanoid)

UUID 보다 가볍고, URL 친화적인 ID를 생성하는 라이브러리이다. JavaScript 가 메인 타겟이지만, 다양한 언어 구현체도 존재한다.
이 선택지는 사용하는 문자의 범위를 지정하고, 사이즈를 가변적으로 선택할 수 있다는 장점이 있다. 

## 결론

고유한 식별을 위한 리소스 ID를 선택할 때, 다양한 방식을 선택할 수 있다는 것을 알고 있으면 되겠다. 꼭 어떤 방식이 무조건적으로 선택될 수도 없고, 
`각각의 상황에 맞게 선택하면 된다`고 생각한다. 그리고 그 상황에는 분산처리, 외부 노출의 허용정도, 캐싱에서의 활용, JPA등에서 사용하는 쓰기 지연의 활용등을 고려하여 선택할 수 있다.
물론 지금 만들고 있는 아주 작은 볼륨을 가진 서비스에는 이렇게 생각하는 것이 **오버엔지니어링**일 수도 있다. 


## 참고자료 :

- PK-를-auto-increment자동-증가-할-경우-생기는-문제점 (https://ssunw.tistory.com/entry/14-PK-%EB%A5%BC-auto-increment%EC%9E%90%EB%8F%99-%EC%A6%9D%EA%B0%80-%ED%95%A0-%EA%B2%BD%EC%9A%B0-%EC%83%9D%EA%B8%B0%EB%8A%94-%EB%AC%B8%EC%A0%9C%EC%A0%90)
- UUID vs Auto Increment (https://velog.io/@qnfmtm666/DevTip-UUID-vs-Auto-Increment)
- UUID 퍼포먼스 이슈 - percona (mysql 8.0 미만) - (링크)
- ycombinator.com ULID 토론  (2018.12 시작) - https://news.ycombinator.com/item?id=18768909
- sequential-uuid-generators (https://www.2ndquadrant.com/en/blog/sequential-uuid-generators/)
- ulid-creator (https://github.com/f4b6a3/ulid-creator)
- google colud spanner 스키마 설계 권장사항 (https://cloud.google.com/spanner/docs/schema-design#primary-key-prevent-hotspots)
- 가상의 상황 트위터 (https://twitter.com/dylayed/status/1496020581610778625 )
- JPA Auto incremet 가 포함된 Insert 쿼리는 언제 나갈까? (https://velog.io/@sangwoo0727/JPA-Auto-Increment%EA%B0%80-%ED%8F%AC%ED%95%A8%EB%90%9C-Insert%EC%BF%BC%EB%A6%AC%EB%8A%94-%EC%96%B8%EC%A0%9C-%EB%82%98%EA%B0%88%EA%B9%8C) 
- sharing & IDs at instagram (https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c)
- hashids (https://hashids.org/)