---
date: '2022-05-13 20:17:32 +09:00'
group: blog
image: /images/posts/docker/docker.png
tags: ["docker", "build cache"]
title: "Gradle을 사용할 때 도커 빌드를 빠르게 하는 방법"
url: /2022/05/13/gradle-docker-cache
type: post
summary: "gradle 을 사용할 때 docker 캐시 레이어를 사용해서 도커 빌드 속도를 빠르게 하는 방법을 알아보았다."
---

# 개요

Spring boot 프로젝트 개발할 때 빌드 과정에서 의존 패키지를 다운받는 작업은 시간이 오래 걸리기 때문에 
도커 빌드 과정에서 이를 매번 반복하면 전체적인 도커 빌드 시간, 나아가 배포 시간이 길어지는 문제가 있다. 

Gradle을 사용할 때 도커 캐시 레이어를 사용해서 이 도커 빌드 타임을 줄이는 방법을 정리해보았다. 
만약 Maven을 사용한다면 [이전 포스트](/2021/08/06/dockerize-maven-project)를 참고하길 바란다.

# 기본적인 도커 빌드 과정

Spring Boot 프로젝트를 배포하기 위한 `Dockerfile` 이 다음과 같을 때 빌드 과정을 살펴보자. 

```dockerfile
FROM gradle:7.4-jdk-alpine
WORKDIR /app
COPY ./ ./
RUN gradle clean build --no-daemon
CMD java -jar build/libs/*.jar
```

1. 기반 이미지에서 gradle 을 사용하여 의존 패키지를 다운받는다. 
2. 애플리케이션 구동을 위한 jar 파일이 빌드된다. 

# 문제점

도커 명령어 `RUN gradle clean build --no-daemon` 을 실행할 때마다 **매번 의존 패키지를 다운받기 때문에 시간이 오래걸린다.**
나의 경우에는 위의 `Dockerfile`을 사용해서 간단한 샘플 애플리케이션 코드를 도커 빌드하는데 `184초`가 걸렸다. 

# 해결방법

## 핵심 아이디어

한번 gradle 빌드를 하면 이 다음에 빌드할 때는 의존 패키지는 캐싱해두었다가 다시 사용되면 도커 빌드 타임이 줄어든다. 따라서 의존 패키지의 캐싱방법을 알아보았다.

## 대안

### 1. 별도의 캐시 디렉토리를 지정하는 방법

만약 jenkins 와 같이 별도의 빌드 환경을 구축해서 사용한다면 캐시 디렉토리를 지정할 수 있다. gradle 6.1 버전 부터는 캐시 디렉토리를 지정할 수 있게 되었다. 
[6.1 의존 패키지 캐싱 디렉토리 지정 매뉴얼](https://docs.gradle.org/6.1/userguide/dependency_resolution.html#sub:cache_copy) 따라서 빌드 할 때 이 캐시 디렉토리를 지정하면 이전에 다운로드된 의존 패키지는 다시 다운로드 받지 않는다.

`$GRADLE_HOME` 변수를 지정할 수 있는데, 문제는 프로젝트마다 각기 다른 디렉토리를 지정해주어야 한다는 불편함이 있어서 다른 방법을 찾아보았다.

### 2. gradle 의존성을 다운로드 받고 이를 도커 캐시에 저장하는 방법

gradle 에서는 의존성만 다운받아서 캐싱하는 명령어는 없다. 하지만 약간의 꼼수를 사용하면 가능하다. 다음과 같이 `Dockerfile`을 변경했다. 


```dockerfile
FROM gradle:7.4-jdk-alpine
WORKDIR /app
ADD build.gradle.kts /app/
RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true

COPY . /app
RUN gradle clean build --no-daemon
CMD java -jar build/libs/*.jar
```

핵심은 `RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true` 명령어다.
앞서 `build.gradle.kts` 파일만 복사했기 때문에 제대로된 빌드가 수행될 수 없다. 하지만 도커 빌드 이미지 내부에 의존 패키지는 다운로드가 된다.
(참고로 그래들 의존 패키지가 저장되는 디렉토리는 `${HOME}/.gradle/` 이다.)
그리고 에러가 발생하지만 `/dev/null` 로 메세지를 보내버렸기 때문에 아무런 응답이 없고 `|| true` 라는 파이프를 연결해서 강제로 성공처리해버렸다.
그 다음에 정상적으로 애플리케이션 소스 파일을 복사하고 다시 빌드하면 빌드 이미지 내부에 저장된 의존 패키지 덕분에 빌드가 빠르게 수행된다. 
그리고 이 과정은 각각의 도커 빌드 명령어로 나뉘어졌기 때문에 다음번에 소스코드만 변경하게 된 경우에는 
의존 패키지를 다시 다운받지 않게 되어 최종적으로 도커 빌드 시간이 줄어든다. 

**최초 도커 파일 적용시 빌드 로그**
```shell
[+] Building 189.3s (11/11) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                                                                                                                                0.0s
 => => transferring dockerfile: 262B                                                                                                                                                                                                                                                0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                                                                                                                                                     0.0s
 => [internal] load metadata for docker.io/library/gradle:7.4-jdk-alpine                                                                                                                                                                                                            1.1s
 => [internal] load build context                                                                                                                                                                                                                                                   0.0s
 => => transferring context: 36.87kB                                                                                                                                                                                                                                                0.0s
 => [1/6] FROM docker.io/library/gradle:7.4-jdk-alpine@sha256:93f4bd52c3372e54e4262c5ec9f0e060747b598b9effd1153def7e89833009ba                                                                                                                                                      0.0s
 => CACHED [2/6] WORKDIR /app                                                                                                                                                                                                                                                       0.0s
 => CACHED [3/6] ADD build.gradle.kts /app/                                                                                                                                                                                                                                         0.0s
 => [4/6] RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true                                                                                                                                                                                                 121.7s
 => [5/6] COPY . /app                                                                                                                                                                                                                                                               0.2s
 => [6/6] RUN gradle clean build --no-daemon                                                                                                                                                                                                                                       63.2s
 => exporting to image                                                                                                                                                                                                                                                              3.0s
 => => exporting layers                                                                                                                                                                                                                                                             2.9s
 => => writing image sha256:ad2b0af2a18dbf64ca93d427868de1562e3a1c3ff8d2c5968e89ba3bc5bc443c                                                                                                                                                                                        0.0s
```

**이후 소스 코드의 일부만 변경된 경우**
```shell
[+] Building 65.6s (12/12) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                                                                                                                                0.0s
 => => transferring dockerfile: 37B                                                                                                                                                                                                                                                 0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                                                                                                                                                     0.0s
 => [internal] load metadata for docker.io/library/gradle:7.4-jdk-alpine                                                                                                                                                                                                            2.4s
 => [auth] library/gradle:pull token for registry-1.docker.io                                                                                                                                                                                                                       0.0s
 => [internal] load build context                                                                                                                                                                                                                                                   0.1s
 => => transferring context: 41.38kB                                                                                                                                                                                                                                                0.1s
 => [1/6] FROM docker.io/library/gradle:7.4-jdk-alpine@sha256:93f4bd52c3372e54e4262c5ec9f0e060747b598b9effd1153def7e89833009ba                                                                                                                                                      0.0s
 => CACHED [2/6] WORKDIR /app                                                                                                                                                                                                                                                       0.0s
 => CACHED [3/6] ADD build.gradle.kts /app/                                                                                                                                                                                                                                         0.0s
 => CACHED [4/6] RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true                                                                                                                                                                                            0.0s
 => [5/6] COPY . /app                                                                                                                                                                                                                                                               0.2s
 => [6/6] RUN gradle clean build --no-daemon                                                                                                                                                                                                                                       62.3s
 => exporting to image                                                                                                                                                                                                                                                              0.5s
 => => exporting layers                                                                                                                                                                                                                                                             0.4s
 => => writing image sha256:7cb34a440b507949715da716d9f4f41ce5ed0b8ada396b9b2f5630e27b91bc11                                                                                                                                                                                        0.0s
```

자세히 보면 `CACHED [4/6] RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true`와 같이 의존패키지를 다운받는 과정이 `CACHED` 라고 표시된다.
도커 레이어 캐시를 활용한 것이다. 

## 중간 결과

1. 도커 빌드 시간이 확 줄어들었다. `189s` -> `65s`
2. 의존 패키지의 변경이 있을 때만 (보다 정확히는 `build.gradle.kts` 파일이 변경 되었을 때만) 새롭게 의존성 패키지를 다운받는다.
3. gradle 의존 패키지는 도커 레이어 캐시에 저장된다.

## 추가 작업

이렇게 하면 도커 빌드는 줄었지만, 최종 도커 이미지가 늘어난다. `multi-stage` 빌드로 바꾸고 약간 더 정리해보자. 

```dockerfile
FROM gradle:7.4-jdk17-alpine as builder
WORKDIR /build

# 그래들 파일이 변경되었을 때만 새롭게 의존패키지 다운로드 받게함.
COPY build.gradle.kts settings.gradle.kts /build/
RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true

# 빌더 이미지에서 애플리케이션 빌드
COPY . /build
RUN gradle build -x test --parallel

# APP
FROM openjdk:17.0-slim
WORKDIR /app

# 빌더 이미지에서 jar 파일만 복사
COPY --from=builder /build/build/libs/my-app-0.0.1-SNAPSHOT.jar .

EXPOSE 8080

# root 대신 nobody 권한으로 실행
USER nobody
ENTRYPOINT [                                                \
    "java",                                                 \
    "-jar",                                                 \
    "-Djava.security.egd=file:/dev/./urandom",              \
    "-Dsun.net.inetaddr.ttl=0",                             \
    "my-app-0.0.1-SNAPSHOT.jar"              \
]
```

이렇게 하면 gradle 의 모든 의존 패키지가 다운로드된 도커 이미지의 최종 사이즈가 다음과 같이 `1.11G` -> `456MB` 로 줄어든다.

```shell
REPOSITORY               TAG       IMAGE ID       CREATED             SIZE
<none>                   <none>    7cb34a440b50   8 minutes ago       1.11GB
<none>                   <none>    cc07b80891c1   About a minute ago   456MB
```

따라서 최종적으로 빌드 시간도 줄어들고, 사이즈도 줄여서 배포시간에도 긍정적인 효과를 가져올 수 있다. 

# 참고자료

- [Gradle 6.1 릴리즈 노트](https://docs.gradle.org/6.1/release-notes.html)
- https://localcoder.org/docker-cache-gradle-dependencies
- https://zwbetz.com/reuse-the-gradle-dependency-cache-with-docker/
