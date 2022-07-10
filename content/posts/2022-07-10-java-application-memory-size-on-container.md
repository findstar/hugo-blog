---
date: '2022-07-10 13:23:03 +09:00'
group: blog
image: /images/posts/java/java-logo.gif
tags: ["java", "heap memory"]
title: "컨테이너 환경에서의 java 애플리케이션의 리소스와 메모리 설정"
url: /2022/07/10/java-application-memory-size-on-container
type: post
summary: "K8S에서 Java 애플리케이션을 구동할 때에는 리소스 설정과 메모리 설정에 주의하지 않으면 성능에 이슈가 발생할 수 있어, 관련 설정을 정리해보았다."
---

# 개요

k8s 와 같은 컨테이너 환경에서 jvm 기반 애플리케이션을 배포할 때 메모리 설정에 주의하지 않으면 애플리케이션의 성능에 이슈가 발생할 수 있다. 
그래서 적절한 메모리 설정 방법에 대해서 정리해보았다. 

# 배경

k8s 환경에서 pod를 배포할 때 종종 5XX 에러가 발생하는 경우가 있는데, 원인이 여러가지가 될 수 있지만 (graceful deploy 적용이 되어 있지 않다면 [이전 글](/2022/05/27/k8s-graceful-deploy-pod)을 참고해주기를 바란다.)
메모리 설정을 잘못하면 이런 에러가 발생할 수 있어서 관련 설정을 주의해야 한다. 특히 pod 가 간헐적으로 restart 되는 경우가 지속된다면 메모리 설정을 제대로 되어 있는지 확인해볼 필요가 있다.

## 기본적인 pod 의 메모리 설정

k8s deployment yml 파일 설정이 다음과 같다면 최대 가용 메모리는 4G 로 설정된다. 
```yaml
containers:
- name: my-app
  image: our-compay-private-repo/our-org/my-app:latest
  imagePullPolicy: Always
  resources:
    limits:
      cpu: 4000m
      memory: 4Gi
    requests:
      cpu: 4000m
      memory: 4Gi
```

# 주의사항

## 1. limit 과 request를 동일하게 설정하자.

먼저 확인할 것은 resource 설정에서 limit 과 request 를 동일하게 설정하는 것이다. 처음 resource 설정할 때 실수하기 쉬운 부분 중 하나인데 이 두 값이 같지 않다면 (ex, request 2G, limit 4G 와 같이 설정한다면) 
의도하지 않게 pod가 restart 되는 경우가 발생할 수 있다.(에러 메세지는 OOM eviction) 따라서 두 값을 동일하게 설정하기를 권장한다. 

### k8s 의 자원할당 QOS 구성은 총 3가지 클래스가 있다.

- Guaranteed : request = limit 
- Burstable : request < limit 
- BestEffort : request, limit 설정 없음

위에서 언급한 request 와 limit을 같게 설정하라는 의미는 QOS 클래스 중에서 guaranteed를 사용하라는 의미인데, 그 이유는 k8s의 메모리 보장방식 때문이다. 

클러스터에서 Pod를 생성할 때 충분한 메모리가 없으면 우선순위가 낮은 pod를 죽여서 메모리를 보장해주는데, 여기서는 OOM Killer에 대해서 알아야 한다. 
OOM Killer는 실행 중인 모든 프로세스를 살펴보며 각 프로세스의 메모리 사용량에 따라 OOM 점수를 산출한다. OS에서 메모리가 더 필요하면 점수가 가장 높은 프로세스를 종료시킨다.
Pod도 프로세스이기 때문에 여기에 해당하는데 QOS 클래스에 따라서 OOM Killer가 참고하는 oom_score_adj 값이 다르다. 

| Pod QOS    | oom score adj    |
|------------|------------------|
| Guaranteed | -998             |
| Burstable  | min(max(2, 1000  |
| BestEffort | 1000             |

따라서 Pod QOS 클래스를 Guaranteed 로 설정해놓으면 OOM Killer로 인해서 갑자기 Pod가 죽는 상황은 발생하지 않는다. (갑자기 죽어버린 Pod가 있다면 의심해보자.)
개런티드 QOS 에 대해서 공식 매뉴얼을 참고하길 바란다. [링크](https://kubernetes.io/ko/docs/tasks/configure-pod-container/quality-service-pod/)

## 2. Heap Memory 설정

위와 같이 설정해놓았더라도 pod 가 메모리를 효율적으로 사용하기 어렵다. 높은 확률로 메모리의 낭비가 발생하게 되기 때문에 다음으로 Heap 설정을 살펴보아야 한다.
JVM 애플리케이션에서 OOM Exception이 발생한다면 다음과 같은 부분이 잘 설정되었는지 고민해보아야 한다. 

### JVM의 Default MaxHeap 사이즈

JVM은 기본적으로 Max Heap 메모리의 사이즈를 물리적인 메모리의 25%를 할당한다. (JVM 10이상부터 컨테이너의 메모리의 25%를 할당한다. 이전 버전에는 버그로 인해서 컨테이너 메모리를 제대로 인식하지 못하는 오류가 있었다.) 
따라서 Memory Limit 을 4G로 설정해놓은 상태에서 별도의 설정을 하지 않았다면 Max Heap 사이즈는 1G가 된다. Heap 영역 이외에도 기본적으로 애플리케이션이 구동되기 위한 메모리가 필요하긴 하지만 이를 감안하더라도 
`4G - Heap memory 1G - (non-heap + system) = 남는 메모리` 가 되는 것이다. non-heap + system이 1.5G라고 가정한다면 Pod에서 1.5G는 Java 애플리케이션에서 활용하지 못하고 낭비된다. 
이렇게 되면 트래픽이 늘어나는 상황에서 메모리가 부족해지면 애플리케이션 메모리는 Pod 메모리 Limit에 도달하지 못했음에도 Heap 메모리가 부족하여 성능이 안 좋게 되는 현상이 발생할 수 있다. 따라서 MaxHeap 에 대한 설정을 추가해주어야 한다. 

* Pod 메모리 구성
  - JVM (Heap)
  - JVM (Non-heap)
  - System
  - IDLE

애플리케이션마다 메모리 사용량이 다를 수 있기 때문에 기본적인 System 메모리의 사용량을 확인하고 Idel을 여유 있게 설정한 다음 Max Heap Size를 늘려서 설정하는 것이 좋다.
(최적의 사이즈는 모니터링을 통해서 조절하는 것이 좋다.) 앞선 경우와 같이 Pod Limit이 4G 라면 System + Non Heap + Idel을 포함하여 2G로 설정하고 Max Heap을 2G로 설정한다면
기본 설정보다 Heap Size가 두 배가 되는 것이다. 

```
java -Xms2048m -Xmx2048m -jar myapp.jar
```

### InitialRAMPercentage, MaxRAMPercentage 옵션 사용 

그런데 이렇게 매번 Pod의 Limit 에 따라서 JVM 애플리케이션 구동 옵션을 변경시키는 것은 번거롭다. 그래서 `Xms`, `Xmx` 옵션대신 `InitialRAMPercentage`, `MaxRAMPercentage` 옵션을 사용하는 것이 더 좋다. 
`MaxRAMPercentage` 옵션은 말 그대로 지정된 메모리 사이즈 대신 퍼센트를 입력할 수 있다. 따라서 결과적으로는 다음과 같게 설정할 수 있다. 

```
java -XX:InitialRAMPercentage=50.0 -XX:MaxRAMPercentage=50.0 -jar myapp.jar
```

단, 낮은 jdk 버전에서는 `MaxRAMPercentage` 옵션이 정상동작하지 않는 버그가 있다. 다음은 openjdk 11 에서의 `MaxRAMPercentage` 옵션이 제대로 할당되지 않는 모습을 확인할 수 있다.
openjdk 13 에서 수정되었다고 하는데 나의 경우에는 openjdk 17 에서 정상동작 하는 것을 확인할 수 있었다. 

```
$ docker run -m 4GB openjdk:11 java -Xms2G -Xmx2G -XshowSettings:vm -version
VM settings:
    Max. Heap Size: 2.00G
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "11.0.15" 2022-04-19
OpenJDK Runtime Environment 18.9 (build 11.0.15+10)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.15+10, mixed mode, sharing)

$ docker run -m 4GB openjdk:11 java -XX:InitialRAMPercentage=50.0 -XX:MaxRAMPercentage=50.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 994.00M
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "11.0.15" 2022-04-19
OpenJDK Runtime Environment 18.9 (build 11.0.15+10)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.15+10, mixed mode, sharing)

$ docker run -m 4GB openjdk:13 java -XX:InitialRAMPercentage=50.0 -XX:MaxRAMPercentage=50.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 994.00M
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "13.0.2" 2020-01-14
OpenJDK Runtime Environment (build 13.0.2+8)
OpenJDK 64-Bit Server VM (build 13.0.2+8, mixed mode, sharing)

$ docker run -m 4GB openjdk:17 java -XX:InitialRAMPercentage=50.0 -XX:MaxRAMPercentage=50.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 2.00G
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "17.0.2" 2022-01-18
OpenJDK Runtime Environment (build 17.0.2+8-86)
OpenJDK 64-Bit Server VM (build 17.0.2+8-86, mixed mode, sharing)
``` 

## 정리 

1. k8s 와 같은 컨테이너 환경에서 jvm 기반 애플리케이션을 배포할 때 request / limit 리소스 설정을 동일하게 하여 Guaranteed QOS 클래스를 설정하자. 
그렇지 않으면 OOM Killer 에 의해서 의도하지 않게 Pod가 restart 되는 경우가 있다. 
2. Heap Size에 대한 설정을 고려하자. 그렇지 않으면 리소스가 할당되어 있음에도 충분히 활용하지 못해서 성능저하가 발생할 수 있다. 
3. 컨테이너 환경에서는 메모리 할당이 가변적이기 때문에 `MaxRAMPercentage` 옵션을 사용하는 것이 좋다. 단 jdk 버전이 낮으면 버그로 동작하지 않는다.

**리소스와 메모리 설정을 잘 지정하여 의도하지 않게 Pod 가 restart 되거나, 메모리가 남아 있음에도 활용하지 못하고 성능이 저하되는 경우를 겪지 않도록 주의하자.**