---
date: '2018-05-02'
group: blog
image: /images/posts/spring-boot-490x257.png
tags:
- study
- spring boot
title: Spring Boot 매뉴얼 뽀개기 1
url: /2018/05/02/start-spring-boot-1
type: post
---


관리하던 Spring 프로젝트를 버전업과 함께 Gradle 및 Boot 기반으로 전환하고자 하는 이슈가 있어 회사 동료분과 함께 Spring Boot 를 차근차근 학습해보기로 했다.
Spring Boot 를 좀 더 잘 이해해야겠다는 마음에 시작했는데, 야심차게 *"Spring Boot 매뉴얼 뽀개기!"* 라고 스터디 제목을 정했다. (과연..)

<!--more-->

스터디를 어떤 방식으로 진행할까 고민하다가 [백기선님의 유튜브](https://www.youtube.com/watch?v=CnmTCMRTbxo&t=890s)를 참고해서
영상과 함께 매뉴얼을 훑어 보자는 아이디어가 나왔고, 나쁘지 않겠다 싶어서 그렇게 하기로 했다. 일정은 한달 안에 필요한 기능들을 확인하는 걸로 정했다.
어차피 스터디 멤버는 완전 초심자가 아니기 때문에, 결국 대상은 : 자바 스프링을 사용해본 경험이 있으며, 프레임워크 및 개발 경험이 좀 있는 사람이 되었다.
[Getting Started](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started)를 기준으로
유튜브 영상은 알아서 챙겨보면 될 것 같고, 유튜브상에서는 잘 다루지 않는 `Gradle` 만 조금 더 서로 내용을 보강하기로 했다.
(이렇게 말하지만, 사실 난 받아먹는 쪽.. 주도는 동료분께서..)

오늘은 그 첫번째 기록이다.

첫번째 날이므로, 기본적인 매뉴얼 페이지를 훑어보고, 목차를 확인하고, 한달간 진행할 분량을 대략 가늠해 보았다.
먼저 `Getting Started` 페이지의 목차인데, 목차를 보고 느낀건, 매뉴얼에 필요한건 왠만큼 다 있기 때문에,
매뉴얼만 잘 보면 대략적인 구동방식과 기능을 파악하는데 문제가 없겠다는 생각이었다.
그렇지만 다들 매뉴얼을 잘 확인 안하는게 문제다.(RTFM)

```
I. Spring Boot Documentation
II. Getting Started
III. Using Spring Boot
IV. Spring Boot features
V. Spring Boot Actuator: Production-ready features
VI. Deploying Spring Boot Applications
VII. Spring Boot CLI
VIII. Build tool plugins
IX. ‘How-to’ guides
X. Appendices
```

### `I. Spring Boot Documentation` 는 개괄적인 소개이다.


#### 1. 이 문서에 대해서 설명

[HTML](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/html), [PDF](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/pdf/spring-boot-reference.pdf), [EPUB](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/epub/spring-boot-reference.epub) 으로 확인가능하다는 안내

#### 2. 도움이 필요할 때 다음을 참고하세요.
- [spring.io](https://spring.io)
- [stackoverflow](https://stackoverflow.com/tags/spring-boot)
- [github issue](https://github.com/spring-projects/spring-boot/issues)

#### 3. 첫번째로 할일
- [소개](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-introducing-spring-boot), [시스템 필요사항](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-system-requirements), [설치하기](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-installing-spring-boot)를 확인하고
- [튜토리얼1](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-first-application), [튜토리얼2](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-first-application-code)를 진행해보자.
- [예제1](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-first-application-run), [예제2](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-first-application-executable-jar)

#### 4. Spring Boot 와 동작하는 것들
- Build System: [메이븐](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-maven), [그래들](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-gradle), [Ant](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-ant), [Starter](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter) - 요건 Boot에서 소개하는 것.
- Best practices : [코드 구조](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-structuring-your-code), [@Configuration 어노테이션](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-configuration-classes), [@EnableAutoConfiguration 어노테이션](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-auto-configuration), [빈과 의존성 주입](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-spring-beans-and-dependency-injection)
- 코드 실행방법 : [IDE](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-running-from-an-ide), [패키지](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-running-as-a-packaged-application), [메이븐](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-running-with-the-maven-plugin), [그래들](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-running-with-the-gradle-plugin)
- 패키징 : [실서버용 JAR 패키징](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-packaging-for-production)
- [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#cli)

#### 5. Spring Boot 의 기능들
- 핵심 기능 : Spring Application, External Configuration, Profiles, Logging
- 웹 어플리케이션 : MVC, Embedded Container
- SQL , NO-SQL
- Messaging : JMS, RabbitMQ, Kafka 등..
- Testing
- Extending

#### 6. 실서비스에서 활용하기
- Endpoing 관리
- HTTP / JMX
- Monitoring : Metrics, Audting, Tracing, Process

#### 7. 기타 토픽
- Boot Application 배포 : 클라우드.
- 빌드 툴 플러그인 : 메이븐, 그래들


### `II. Getting Started` 을 참고해서 프로젝트를 실행해 본다.

#### 8. Spring Boot 소개하기

Spring Boot 는 손쉽게 단독으로 실행가능(standalone)한 어플리케이션을 만들 수 있도록 도와줍니다. 또한 실행가능한 `jar` 형태로 패키징될 수 있고, 이를 도와주기 위한 CLI도 제공합니다.
- 스프링 개발자를 위한 빠르고, 손쉬운 접근이 가능 경험을 제공하는 것을 목표로 합니다.
- 별다른 설정없이도 구동이 가능한 어플리케이션을 만들 수 있습니다.
- 작은 규모에서 부터 큰 프로젝트에 이르기 까지 수용할 수 있습니다.
- XML 설정을 위한 코드 생성이나 필요사항을 가지지 않습니다.

#### 9. 시스템 요구사항

Spring Boot 2.0.1 버전은 Java 8 또는 9 그리고 스프링프레임워크 5.0.5 이상을 필요로 하고. 빌드툴은 메이븐 3.2+, 그래들4를 지원한다.

#### 10. 서블릿 : 다음의 embedded 서블릿을 제공한다.

서블릿 3.1 이상을 지원하는 컨테이너들을 지원한다
- Tomcat 8.5
- Jetty 9.4
- Undertow 1.4

#### 11. 설치하기
  * 설치는 여러가지 방법이 있는데 일단 먼저 mavan 기반으로 시작해 보도록 한다.
  * 먼저 내가 사용하는 IntelliJ 에서 `new project` 를 선택하고 maven 프로젝트를 선택하자
  * 그 다음 groupid ex) `com.example`, artifactid ex) `demo` 를 지정하고
  * `Next` 를 눌러 project name 과 프로젝트가 저장될 위치를 지정한다.
  * 이제 pom.xml 파일에 parent, dependencies, build 를 추가해서 다음처럼 구성하자.
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>com.example</groupId>
        <artifactId>demo</artifactId>
        <version>1.0-SNAPSHOT</version>

        <!-- Inherit defaults from Spring Boot -->
        <parent>

            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.0.1.RELEASE</version>
        </parent>
        <!-- Add typical dependencies for a web application -->
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
        <!-- Package as an executable jar -->
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>

    </project>
    ```
  * pom.xml 파일 에서 `spring-boot-starter-parent` 을 입력할 때 버전을 지정하면, 하위 ex) `spring-boot-starter-web` 에서도 동일한 버전을 따른다.
  * IDE 의 View - Tool windows - Maven project view를 열어서 maven reimport 를 실행하자. maven 의존 패키지들을 다운받는다.
  * `src/main/java/com.example/` 위치에  `Application.java` 파일을 새롭게 작성하자

    ```
    package com.example;

    import org.springframework.boot.*;
    import org.springframework.boot.autoconfigure.*;
    import org.springframework.web.bind.annotation.*;

    @RestController
    @EnableAutoConfiguration
    public class Application {

        @RequestMapping("/")
        String home() {
            return "Hello World!";
        }


        public static void main(String[] args) throws Exception {
            SpringApplication.run(Application.class, args);
        }
    }
    ```
  * maven project view 에서 run maven build 를 실행해서 어플리케이션을 실행시켜 보자 `http://localhsot:8080` 에서 Hello world 를 확인할 수 있다.
  * 어노테이션을 조금 살펴보면, `@RequestMapping` 어노테이션은 라우팅 정보를 제공한다.
  * `@RestController` 어노테이션은 콜러에서 직접 결과 문자열을 돌려주도록 스프링에게 지시한다. 이 둘은 spring MVC 어노테이션으로 [MVC 섹션](https://docs.spring.io/spring/docs/5.0.5.RELEASE/spring-framework-reference/web.html#mvc)에서 더 자세히 보도록 한다.
  * `@EnableAutoConfiguration` 어노테이션은 추가한 jar를 기반으로 어떻게 스프링을 설정할 것인지 스프링 부트가 추측하도록 지시한다. `spring-boot-stater-web`은 톰캣, 스프링 MVC를 추가하기 때문에 자동-설정은 웹 어플리케이션을 개발한다고 예상하고, 그에 따라 스프링을 set up 한다.
  * 이제 이 어플리케이션을 패키징 해보자. maven project view 에서 Lifecycle > package 를 실행하면 실행가능한 jar 파일이 target 디렉토리 및에 생성된다.
  * console 에서 java -jar target/demo-1.0-SNAPSHOT.jar 라고 입력해보자. maven project 에서 실행한 결과와 동일한 어플리케이션이 구동된다.

일단 첫날은 요정도의 내용을 진행해보았고, 이후에 차근차근 다른 내용을 살펴볼 예정이다. 첫날에는 maven 으로 시작했지만 앞으로의 목표는 `gradle`을 추가적으로 적용해보도록 하겠다.

To be continue..

### 참고

 - [Spring Boot Getting Started](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started)



