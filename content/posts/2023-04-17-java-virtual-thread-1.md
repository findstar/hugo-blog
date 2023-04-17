---
date: '2023-04-17 19:30:00 +09:00'
group: blog
image: /images/posts/java/java-logo.gif
tags: ["java", "jdk 21", "virtual thread", "throughput"]
title: "Virtual Thread란 무엇일까? (1)"
url: /2023/04/17/java-virtual-threads-1
type: post
summary: "2023년 9월 릴리즈 예정인 JDK 21 (LTS) 버전에는 주목할만한 기능이 추가된다. 바로 Virtual Thread다. 이 Virtual Thread 이 탄생한 배경을 살펴보고, 어떤 목적을 가지고 있는지 살펴보았다."
목적 : "Virtual Thread를 소개하고 기본적인 사용방법을 정리하여 공유한다."
대상독자 : "Java 프로그래밍을 사용해보고, Thread에 대해서 기본적인 개념을 이해하고 있는 사람"
---

# Virtual Thread

2023년 9월에 릴리즈될 예정인 **JDK 21** (2023.09 월 릴리즈 예정) 는 Java 8 이후 세번째 LTS 버전이다(11, 17, 21). 이 버전에서는 많은 사람들이 기다리고 있는 `Virtual Thread` 라는 기능이 추가될 에정이다.
이 `Virtual Thread` 가 어떤 의미가 있기 때문에 많은 사람들이 기다리고 있는지 알아보고 그 의미를 정리해보았다. 이 글은 `Virtual Thread`의 첫 번째 글이고 다음 글에서 계속 이어진다.

## Virtual Thread 란? 

`Virtual Thread` 란 기존의 전통적인 Java Thread 에 더하여 새롭게 추가되는 경량 Thread 이다. `Project Loom`의 결과물로 추가된 기능으로 OS Thread를 그대로 사용하지 않고
JVM 자체적으로 내부 스케줄링을 통해서 사용할 수 있는 경량의 스레드를 제공한다. 하나의 Java 프로세스가 수십만~ 수백만개의 스레드를 동시에 실행할 수 있게끔 설계되었다.

> Project Loom 이란?
> `Virtual Thread`를 비롯하여 경량의 스레드를 JDK에 추가하기 위한 여러가지 기능들을 개발하는 프로젝트로 **Loom**이란 Thread 의 사전적 정의가 '**실**' 이라는데 착안하여
실을 엮어 '**직물을 만든다는 뜻**'이다.  
> Loom 프로젝트의 결과로 탄생한 Virtual Thread도 처음에는 Fiber-섬유 라고하는 별도의 기능으로 개발되었으나, 최종적으로는 Thread와의 호환될 수 있는 Virtual Thread로 발전했다.

먼저 이 `Virtual Thread` 가 왜 필요하게 되었는지 그 배경과 `Virtual Thread` 가 해결하고자 하는 문제에 대해서 알아보자.

## 배경
1. **자바의 Thread 는 OS의 warpping**
   - 자바의 전통적인 Thread 는 OS Thread의 wrapping 으로 이를 Platform Thread 라고 정의한다.
   - 따라서 Java 애플리케이션에서 Thread를 사용하는 코드는 실제적으로는 OS Thread를 이용하는 방식으로 동작했다.
   - OS 커널에서 사용할 수 있는 스레드는 갯수가 제한적이고 생성과 유지 비용이 비싸다. 
   - 이 때문에 기존에 애플리케이션들은 비싼 자원인 Platform Thread를 효율적으로 사용하기 위해서 **Thread Pool** 통해서 사용해왔다. 

2. **처리량의 한계**
   - Spring Boot 의 기본적인 사용자 요청 처리 방식은 전통적인 처리 모델인 thread per request (요청당 스레드)이다. 이는 하나의 request(요청)을 처리하기 위해서 하나의 thread를 소비한다.
   - 애플리케이션에서 처리량을 늘리려면 thread를 늘려야 하지만 전통적인 thread는 무한정 늘릴 수 없다. (OS 스레드를 무한정 늘릴 수 없기 때문) 
   - 따라서 애플리케이션의 처리량(throughput)은 Thread, 그리고 Thread Pool 에서 감당할 수 있는 범위를 넘어서 늘어날 수 없다.

3. **Blocking 코드로 인한 리소스 낭비**
   - Thread per Request 모델에서는 요청을 처리하는 스레드에서 IO 작업 처리를 위한 Blocking 이 일어난다. 
   - 이 때문에 Thread 는 IO 작업이 마칠 때까지 다른 요청을 처리하지 못하고 기다려야 한다.(Blocking 동안 대기)
   - 애플리케이션에 유입되는 요청이 많지 않거나 또는 스케일 아웃으로 충분히 커버할 수 있는 정도라면 문제가 없지만
   - 아주 많은 요청을 처리해야하는 상황이라면 Blocking 방식으로 인해 발생하는 낭비를 줄여야 할 필요가 있다.
   - 이 때문에 Blocking 이 아니라 Non-blocknig 방식의 reactive 패러다임이 발전하였다.

4. **reactive 스타일의 단점**
   - 처리량을 높이기 위한 방법으로 비동기 방식의 reactive 프로그래밍이 발전해왔다.
   - 한정된 자원인 Platform Thread가 Blocking 되면서 대기하는 데 소요된 스레드 자원을 non blocking 방식으로 변경하면서 다른 요청을 처리하는데 사용할 수 있게 되었다.
   - 대표적으로 reactive 스타일의 webflux 가 이렇게 동작한다. 
   - 다만 이런 reactive 스타일의 코드는 작성하고 이해하는 비용을 높게 만들었다.
   - 그리고 기존의 자바 프로그래밍의 패러다임은 스레드를 기반으로 하기 때문에 thread per request 에 맞춰져 있던 라이브러리들 모두 reactive 스타일로 새롭게 작성되어야 하는 문제가 있다.

5. **자바 플랫폼의 디자인**
   - 자바 플랫폼은 전통적으로 Thread를 중심으로 구성되어 있었다. 
   - Thread 호출 스택은 thread local을 사용하여 데이터와 컨텍스트를 연결하도록 설계되어 있다.
   - 이 외에도 Exception, Debugger, Profile(JFR)이 모두 스레드를 기반으로 하고 있다. 
   - Reactive 스타일로 코드를 작성하면 이 Thread를 넘나들면서 사용자의 요청이 처리되는데 이 때문에 하나의 요청을 처리하는 여러개의 Thread의 상황이 발생할 수 있고 결국 degugging 이 어려워 졌다. 

## 목적

Project Loom 의 결과로 탄생한 Virtual Thread는 다음과 같은 목적을 가지고 있는데, 기존의 Reactive Programming 과 비교해서 생각해보자.

### 해결하고자 하는 문제

1. Java 개발자가 하드웨어의 성능을 잘 활용하는 높은 처리량(쓰루풋)의 서버를 작성하는 것
    - Virtual Thread는 Blocking 이 발생하면 내부적으로 스케줄링을 활용하여 Platform Thread가 그냥 대기하게 두지 않고 다른 Virtual Thread가 작업할 수 있도록 한다.
    - 따라서 reactive programming 의 non-blcking 과 동일하게 Platform Thread의 리소스를 낭비하지 않는다.

2. 동시에 자바 플랫폼의 디자인과 조화를 이루는 코드를 생성할 수 있도록 하는 것
    - 기존 Reactive Programming 의 장점에도 불구하고 전통적인 자바 언어의 구조는 Thread를 기반으로 하였기 때문에 Webflux등을 사용할 때 디버깅이 어려웠다.
    - 하지만 Virtual Thread는 Thread 구조를 그대로 사용하기 때문에 디버깅, 프로파일링등 기존의 도구도 그대로 사용할 수 있다.

### Reactive Programming 과의 비교

* reactive programming 이 달성하고자 리소스를 효율적으로 사용하여 높은 처리량(throughput)을 달성하고자 하는 목적은 동일하다.
* Virtual Thread를 사용하면 Non blocking 에 대한 처리를 JVM 레벨에서 담당해준다.
* 따라서 Spring Web MVC 스타일로 코드를 작성하더라도 내부에서 Virtual Thread가 기존의 Platform Thread를 직접 사용하는 방식보다 효율적으로 스케줄링하여 처리량을 높일 수 있다.


* 결론적으로 Virtual Thread 는 기존 Thread 방식의 이점을 누리면서도 Reactive Programming 의 장점을 취할 수 있다.

{{< imageFull src="/images/posts/java/virtual-thread/virtual-thread.png" title="Virtual Thread와 기존 방식의 비교" border="false" >}}


## 구조

그럼 어떻게 Virtual Thread 가 이런 목표를 달성할 수 있는지 그 구조를 살펴보자.

### Platform Thread와 Virtual Thread 의 구조 차이

앞서 Platform Thread는 OS Thread의 Warpping 이라고 설명했다. 애플리케이션 코드가 Platform Thread를 사용하면 실제로는
OS Thread를 사용하는 것이다. 이 때 Thread는 비용이 비싸기 때문에 Thread Pool 을 사용하여 접근하는 방식으로 사용해왔다.

{{< imageFull src="/images/posts/java/virtual-thread/traditional-thread.png" title="전통적인 Thread 사용방법" border="false" >}}

이에 반해 Virtual Thread는 OS Thread의 Warpping 이 아니기 때문에 애플리케이션 코드는 Virtual Thread Pool 없이 사용하고
JVM 자체적으로 Virtual Thread를 OS Thread와 연결하는 스케줄링한다. 이 작업을 mount / unmount 라고 하고 기존에 Platform Thread라고 하던 부분을
Carrier Thread라고 한다. (Virtual Thread를 실제 OS Thread로 연결해준다는 의미)

{{< imageFull src="/images/posts/java/virtual-thread/virtual-thread-structure.png" title="Virtual Thread 사용방법" border="false" >}}

구조적으로 보자면 OS Thread를 사용하기 전에 하나의 레이어가 더 있는 것 처럼 보인다. (Virtual Thread Scheduling) 
하지만 이 자체적인 Scheduling 을 통해서 큰 차이가 발생한다. **기존의 Thread 는 Blocking 이 발생하면 그냥 기다려야** 했는데, 
**Virtual Thread는 Blocking 이 발생하면 내부의 Scheduling 을 통해서 실제 작업을 처리하는 Carrier Thread는 다른 Virtual Thread의 작업을 처리하면 된다.**
따라서 Non-Blocking 이 누리는 장점을 동일하게 누릴 수 있다. 이를 도식화 하면 다음과 같다.

{{< imageFull src="/images/posts/java/virtual-thread/virtual-thread-mount-and-unmount.png" title="Virtual Thread Scheduling" border="false" >}}

다만 위와 같은 구조는 Virtual Thread가 수십~수백만까지 늘어날 수 있기 때문에 전통적인 Thread와 동일한 메모리 비용, 컨텍스트 비용이 발생하면 감당하기 어렵다.
따라서 내부적으로는 전통적인 Thread와 Virtual Thread는 자원 사용량의 차이가 있다.

### 사용하는 자원의 차이
|              | Platform Thread         | Virtual Thread |
|--------------|-------------------------|----------------|
| Metadata size | 약 2kb(OS별로 차이있음)        | 200~300 B      |
| Memory       | 미리 할당된 Stack 사용         | 필요시 마다 Heap 사용 |
| 컨텍스트 스위칭 비용  | 1~10us (커널영역에서 발생하는 작업) | ns (or 1us 미만) |


## 사용법

`Virtual Thread`의 배경과, 목적, 구조에 대해서 알아보았으니 실제 `Virtual Thread` 를 사용하는 코드를 실행시켜보자. 

### 준비과정

아직 JDK 21은 정식 릴리즈 되지 않은 EAP 단계이기 때문에 [SDKMan](https://sdkman.io/) 을 사용하여 21 EAP (OpenJDK) 를 설치한다.

```bash
$ sdk install java 21.ea.18-open

Downloading: java 21.ea.18-open

In progress...

################################################################################################ 100.0%

Repackaging Java 21.ea.18-open...

Done repackaging...
Cleaning up residual files...

Installing: java 21.ea.18-open
Done installing!


Setting java 21.ea.18-open as default.

$ java --version
openjdk 21-ea 2023-09-19
OpenJDK Runtime Environment (build 21-ea+18-1480)
OpenJDK 64-Bit Server VM (build 21-ea+18-1480, mixed mode, sharing)
```

다음은 `Virtual Thread` 를 사용하는 코드를 작성해보자. IntelliJ 에서 아직 JDK 21 문법을 지원하지 않아 빌드가 잘 되지 않아 CLI 에서 실행하였다.

```java
public class Main {
    public static void main(String[] args) throws Exception {
        run();
    }

    public static void run() throws Exception {
         
         // Virtual Thread 방법 1
         Thread.startVirtualThread(() -> {
	        System.out.println("Hello Virtual Thread");
         });
         
         // Virtual Thread 방법 2
         Runnable runnable = () -> System.out.println("Hi Virtual Thread");
         Thread virtualThread1 = Thread.ofVirtual().start(runnable);
         
         // Virtual Thread 이름 지정
         Thread.Builder builder = Thread.ofVirtual().name("JVM-Thread");
         Thread virtualThread2 = builder.start(runnable); 
    }
}
```

```bash
$ java --version
openjdk 21-ea 2023-09-19
OpenJDK Runtime Environment (build 21-ea+18-1480)
OpenJDK 64-Bit Server VM (build 21-ea+18-1480, mixed mode, sharing)
$ javac Main.java
$ ls
Main.class Main.java
$ java Main
Hello Virtual Thread
Hi Virtual Thread
Hi Virtual Thread
```

기존의 Thread를 생성하던 문법과 큰 차이가 없이 `Virtual Thread`를 만들 수 있다는 걸 알수 있다. 이번에는 Executors 를 사용하여 10만개의 `Virtual Thread`를 만들고 
2초간 대기하도록 한뒤에 전체 실행시간을 측정해보자. Blocking 이 발생하면 다른 `Virtual Thread`가 실행되기 때문에 제대로 동작한다면 약 2초를 조금 넘기는 시간 안에 완료되어야 한다.


```java
public class Main {
    public static void main(String[] args) throws Exception {
        run();
    }

    public static void run() throws Exception {
        while (true) {
            long start = System.currentTimeMillis();
            try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
                // 10만개의 Virtual Thread 실행 
                for (int i = 0; i < 100_000; i++) {
                    executor.submit(() -> {
                        Thread.sleep(Duration.ofSeconds(2));
                        return null;
                    });
                }

            }
            long end = System.currentTimeMillis();
            System.out.println((end - start) + "ms");
        }
    }
}
```

마찬가지로 cli 에서 실행해보았다.

```bash
$ java --version
openjdk 21-ea 2023-09-19
OpenJDK Runtime Environment (build 21-ea+18-1480)
OpenJDK 64-Bit Server VM (build 21-ea+18-1480, mixed mode, sharing)
$ javac Main.java
$ ls
Main.class Main.java
$ java Main
2550ms
2562ms
2073ms
2089ms
2102ms
2099ms
```

결과를 보면 알 수 있지만 2초에 가까운 시간이 출력된다. `Thread.sleep` 에 의해서 Blocking 되었지만 내부 `Virtual Thread` 스케줄러에 의해서 
다음 `Virtual Thread`가 실행되기 때문에 전체 처리 시간은 2초에 가깝게 나온다. 이 것을 통해서 Blcoking 코드가 있더라도 `Virtual Thread`를 사용하면 처리량이 늘어날 수 있다는 것을 예상할 수 있다. 

## 정리

### 요약

- 높은 처리량을 높이기 위해서 기존에는 Reactive Programming 과 같은 방식을 사용했지만, 
- `Virtual Thread` 를 사용하면 Reactive Programming 이 추구하는 Nonblocking을 통한 효율적인 자원 사용이 가능해진다.
- `Virtual Thread` 가 JVM 내부에서 알아서 스케줄링 해주기 때문에 Virtual Thread Pool 을 사용하지 않는다.
- Reactive Programming 보다 가독성 좋은 코드를 유지할 수 있고, 기존 Thread 와 동일하게 동작하므로 디버깅이 용이하다. 

### 생각해볼 부분

1. `Virtual Thread` 기능이 추가되었다고 해서 기존의 Thread를 사용하지 못하는 것은 아니다.
   - 기존의 Thread도 사용가능하고, 추가된 `Virtual Thread` 도 사용가능하다. 서로 대치되는 것이 아니라 공존하는 것이다.
2. `Virtual Thread` 를 사용하더라도 응답속도가 빨라지지는 않는다. (오히려 약간 느려질 수도). 다만 처리량이 늘어날 수 있다.
3. 실제 Thread를 직접 다루거나 Executors를 사용하는 코드를 많이 작성하지는 않는다. 오히려 기존의 라이브러리들이 `Virtual Thread`를 사용할 수 있도록 개선될 것같다.
4. Reactive Programming 과 같이 높은 처리량을 필요로하는 부분들은 `Virtual Thread`를 사용하는 방식으로 전환될 수도 있을것 같다.


### 소감

`Virtual Thread` 를 알아보면서 JDK 21 버전이 기대되기 시작했다. 더욱이 **JDK 21** 버전이 **LTS** (Long Term Support)로 출시되기 때문에 
새로운 프로젝트들은 JDK 21 을 기반으로 시작한다면 `Virtual Thread`의 이점을 적용할 수 있을 것이다. 그리고 기존 프로젝트들도 높은 처리량(throughput)을 필요로 하는 경우에
JDK 버전업을 고려해볼만 하다고 생각된다. 

자료를 정리하면서 **Project Loom** 에서 `Virtual Thread` 기능을 구현하기까지 5년 넘게 개발했다는 것을 알게되었다. 기존의 Thread 사용성을 해치지 않으면서도
단점만을 개선한 `Virtual Thread` 를 내놓기 까지 얼마나 많은 고민을 했을까 생각해았다. 쉽지 않은 일이었을텐데 결국 이렇게 JDK 21에 포함되는 기능을 내놓은 것이 대단하다고 느껴졌다.


내용이 길어져 다음 글에서 실제 Spring Boot 에 `Virtual Thread` 적용한뒤 실제로 기존 Thread Pool 방식과 비교하여 처리량이 늘어나는지, 그리고 `Virtual Thread`를 사용할 때 주의해야할 점에 대해서 추가 내용을 정리해보겠다. 


### 참고 자료
- https://openjdk.org/jeps/444
- https://spring.io/blog/2022/10/11/embracing-virtual-threads
- https://www.infoq.com/articles/java-virtual-threads/
- https://dev.to/jorgetovar/virtual-threads-in-java-23mf
- https://howtodoinjava.com/java/multi-threading/virtual-threads/
- https://www.infoq.com/news/2023/04/virtual-threads-arrives-jdk21/