---
date: 2021-08-06T20:08:09+09:00
group: blog
image: /images/posts/docker/docker.png
tags: ["dockerize", "spring boot", "maven"]
title: "Spring 프로젝트 Maven을 사용할 때 도커라이즈 캐싱방법"
url: /2021/08/06/dockerize-maven-project
type: post
summary: "spring 프로젝트에서 maven 을 사용할 때 도커라이즈에서 레이어를 캐싱하여 빌드 속도를 향상시키는 방법을 살펴보았다. "
---

# 개요

Spring boot 프로젝트 개발할 때 의존 패키지를 다운받는 과정이 소모적으로 느껴져 도커의 빌드 레이어를 사용하여 
의존성을 다운받는 과정을 생략할 수 없을까 고민하게 되었다. 의존성 매니저 도구로 Maven 을 사용하고 있었기 때문에
몇번의 시행착오를 거쳐서 현재 적용하고 있는 방법을 정리해보았다.

# 아이디어 

Spring Boot 프로젝트를 `Dockerfile` 을 사용하여 도커라이즈 하면 매번 maven 의존성을 다운로드 하는 과정을 거친다.
이 과정을 Jar 패키징과 분리하여 처리하여 의존성을 매번 다운로드 받지 않아도 되기 때문에 좀더 빠른 도커 빌드가 가능해질꺼라고 생각되었다.

## `mvn go-offline` 을 사용하는 방법

작업하는 Spring 에서 Maven 을 사용하고 있었기 때문에 Maven 의 `go-offline` 기능을 사용해보려고 시도하였다. 
`Dockerfile` 은 다음과 같이 구성하였다. 

```dockerfile
FROM maven:3.6.3-jdk-11 as builder

# working directory 지정
WORKDIR /app

# maven dependency 를 복제해서 캐싱하도록 시도
COPY pom.xml .

# maven dependency 를 다운로드 받는다.
RUN mvn dependency:go-offline

# 소스코드 복사
COPY src/ /app/src/

# 메이븐 패키징
# RUN mvn -o package (이 명령어는 오프라인의 디펜던시를 활용하여 패키징 하는 명령어지만, 도커 빌드가 실패하여 사용이 불가능했다.)
RUN mvn package

# 멀티 스테이지 빌드
FROM adoptopenjdk/openjdk11:jre-11.0.10_9-alpine

WORKDIR /app

COPY --from=builder /app/target/*-SNAPSHOT.jar /app

ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

EXPOSE 8080

ENTRYPOINT [                                                \
    "java",                                                 \
    "-jar",                                                 \
    "-Djava.security.egd=file:/dev/./urandom",              \
    "/app/my-app.jar"                  \
]
```

그런데 나의 도커 빌드 환경은 로컬이나 별도의 CI 환경이 아니라 사내에서 운영하는 private docker registry 에서 제공하는 기능을 사용하고 있어서
kaniko 를 사용하고 있었다. 여기에서는 내가 원하는 대로 `go-offline` 이후에 의존성이 캐싱되지 않았다. 그리고 어떤 원인에서인지는 정확히 알 수 없지만
`pom.xml` 에 정의해둔 maven plugin + kotlin 설정으로 인해서 `go-offline` 을 통해서도 완전히 의존성이 다운로드 되지 않고 `mvn package` 에서
매번 추가 의존 패키지를 다운로드 받는 것을 확인하였다. 원래는 `go-offline` 이후 `mvn -o package` 를 사용하여 offline 에 다운로드 받은 의존성 패키지만을 사용하여 패키징 하도록 시도하였으나 이경우에는 아예 도커 빌드가 실패하였다.   

## `mvn vefiry` 를 사용하는 방법 

두 번째로는 `mvn verify` 명령어를 사용하는 방법이다. 

```dockerfile
FROM maven:3.6.3-jdk-11 as builder

# working directory 지정
WORKDIR /app

# maven file copy
COPY pom.xml .

# Download maven dependency
RUN mvn verify --fail-never

# 소스코드 복사
COPY src/ /app/src/

# 메이븐 패키징
RUN mvn package

# 멀티 스테이지 빌드
FROM adoptopenjdk/openjdk11:jre-11.0.10_9-alpine

WORKDIR /app

COPY --from=builder /app/target/*-SNAPSHOT.jar /app

ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

EXPOSE 8080

ENTRYPOINT [                                                \
    "java",                                                 \
    "-jar",                                                 \
    "-Djava.security.egd=file:/dev/./urandom",              \
    "/app/my-app.jar"                  \
]
```

이 경우에는 의도한 대로 조금 더 빠른 도커 빌드가 가능한 결과를 확인할 수 있었다. 

### 빌드 시간 비교

1. mvn verify 없이 통째로 빌드하는 경우 : `271.0s` 
2. mvn verify 도입후 디펜던시 변경시 : `338.6s` 
3. mvn verify 도입후 캐시 레이어 기반에서 소스만 변경시 `99.7s` 


### 기타

* 만약 jenkins 환경이 별도로 구성되어 있고 도커 빌드 타이밍에 의존성 패키지를 다운로드 받는 캐시를 위한 볼륨 마운트가 가능하다면 https://daddyprogrammer.org/post/12542/springboot-docker-integration/ 에서와 같이 볼륨 마운트를 시도해 볼수도 있다.

* 또는 Java 한정이긴 하지만 `jib` 이나 `buildpack` 을 시도해 볼 수도 있을 것 같다. 

### 요약 

Maven 기반의 Spring boot 프로젝트를 도커라이즈 할 때 `mvn verify` 를 사용하면 의존 패키지를 도커 레이어에 캐싱 할 수 있어 
전체 도커 빌드 타임이 줄어드는 효과를 얻을 수 있다. 

### 참고 
- https://ohjongsung.io/2019/10/20/spring-boot-%EB%8F%84%EC%BB%A4-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%B5%9C%EC%A0%81%ED%99%94
- https://velog.io/@ojwman/docker-maven
- https://daddyprogrammer.org/post/12542/springboot-docker-integration/
