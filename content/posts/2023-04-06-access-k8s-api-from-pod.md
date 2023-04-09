---
date: '2023-04-06 21:03:22 +09:00'
group: blog
image: /images/posts/kubernetes/Kubernetes-logo.png
tags: ["k8s", "kubernetes", "k8s api", "service account & role"]
title: "쿠버네티스 클러스터의 POD에서 클러스터 API 사용하기"
url: /2023/04/09/access-k8s-api-from-pod
type: post
summary: "쿠버네티스 클러스터 환경에서 애플리케이션을 구성하다보면, 경우에 따라 POD 에서 클러스터 API에 접근하여 클러스터에서 제공하는 다양한 기능들을 활용해야할 때가 있다. 이런 경우 어떤 준비가 필요한지 살펴보았다."
---

# K8S 클러스터 POD에서 K8S API 사용하기

## 배경
 쿠버네티스(줄여서 k8s라고 부르는) 클러스터에서 애플리케이션을 운영하다보면, 경우에 따라 K8S 클러스터 자체의 API를 사용하고 싶을 때가 있다. 예를 들어 '클러스터의 동작 일부를 직접 모니터링 한다던가', '동적으로 POD 또는 서비스의 라이프사이클을 관리한다던가' 하는 경우가 그렇다.
나의 경우에는 단순히 POD의 목록을 확인하려는 필요가 있어서 API 사용방법을 찾아보았는데, 생각보다 고려할 부분이 많아서 글로 정리해보았다.

## 1. K8S 클러스터 살펴보기

### K8S 클러스터의 노드 구성
 기본적으로 K8S 클러스터는 다음의 2가지 노드로 구분될 수 있다.  
- Master Node
- Worker Node 
 
필요에 따라서는 더 세분화할 수도 있지만(Ingress Node 라던가, Node 그룹핑으로 나눌 수도 있다), 최소한 위와 같이 2가지로 구분되며, Master Node 가 클러스터 전체를 관리하는 역할을 수행하며 클러스터 API를 제공하고, Worker Node 는 실제 우리가 원하는 Pod 가 실행되는 역할을 수행한다. 

### K8S API란?
 쿠버네티스 클러스터를 학습하게 되면 배우게 되는 CLI 명령어가 바로 `kubectl` 이다. 알고보면 이 `kubectl` 명령어는 K8S 클러스터의 마스터 노드의 REST API를 사용하여 각종 작업을 처리하는데
이 때 사용하는 API가 바로 **K8S API** 이다.

## 2. K8S API를 사용하기 위한 인증방법 이해하기 (로컬에서 호출해보기)

`kubectl` 도 결국 동작원리를 살펴보면 K8S Master Node API를 호출하는 것을 알게되었지만, 아무런 준비없이 K8S API를 호출한다고 해서 클러스터에게 다양한 작업을 처리하도록 지시할 수는 없다. 
먼저 기본적인 개념이해를 위해서 로컬에서 다음과 같이 하나씩 확인해보자. (미리 준비되어 있는 k8s 클러스터가 있고, 로컬에서 kubectl 을 사용할 수 있는 환경이라고 가정한다.)

1. 클러스터의 정보 획득
    `kubectl` 을 사용중이라면 다음과 같이 명령어를 입력하여 클러스터의 정보를 확인할 수 있다.
    ```bash
    $ kubectl cluster-info
    Kubernetes control plane is running at https://192.168.55.300:8443
    # 클러스터의 Control plane 이 Master Node 로 연결되기 위한 주소이다.
    ```
2. 클러스터의 API 조회해보기
   클러스터의 주소를 확인하였으면, `HTTPS` 를 사용한다는 점을 알 수 있다. HTTPS 통신을 위해서는 인증서가 필요한데 지금 당장은 이를 확인할 수 없으니 무시하고 CURL로 API를 요청해보자 .
   ```bash
   $ curl https://192.168.55.300:8443 # 그냥 호출해본다.
    curl: (60) SSL certificate problem: unable to get local issuer certificate
    More details here: https://curl.se/docs/sslcerts.html
    
    curl failed to verify the legitimacy of the server and therefore could not
    establish a secure connection to it. To learn more about this situation and
    how to fix it, please visit the web page mentioned above.
   
   $ curl https://192.168.55.300:8443 -k # insecure 옵션을 추가해서 https 인증서 오류 무시하여 요청
   {
    "kind": "Status",
    "apiVersion": "v1",
    "metadata": {
    
    },
    "status": "Failure",
    "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
    "reason": "Forbidden",
    "details": {
    
    },
    "code": 403
   }
   # 또는 단순하게 'Unauthorized' 라고 응답될 수도 있다. 
   ```

3. 토큰을 포함하여 요청해보기
   `~/.kube/config` 파일을 확인해보면 클러스터에 연결하기 위한 token 을 확인할 수 있다. 이 토큰을 복사하여 다음과 같이 직접 curl 을 요청해보면 사용가능한 클러스터 API 목록을 확인할 수 있다.
   ```bash
   curl -H "Authorization: Bearer JWT_형태의_TOKEN_내용을_이곳에_넣는다" https://192.168.55.300:8443 -k
   {
    "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    .....
    ]
   }
   ```
   
4. kubectl proxy 사용해보기
   매번 이렇게 힘들게 API를 요청하려면 불편하다. 로컬에서 HTTP 연결을 쉽게 하기 위해서 kubectl 의 proxy 기능을 활용해보자. 
   ```bash
   $ kubectl proxy
   Starting to serve on 127.0.0.1:8001
   
   # 새로운 터미널에서 확인해보자.
   $ curl http://localhost:8001
   {
      "paths": [
      "/api",
      "/api/v1",
      "/apis",
      "/apis/",
      .....
      ]
    }
   ```
   복잡하게 토큰을 전달하지 않아도 클러스터 API 호출이 가능하다. 반환된 API 목록을 살펴보면서 내가 필요한 API를 선택할 수 있다.

### 로컬 호출 정리
- K8S API 호출은 기본적으로 HTTP API 이고 이 호출을 위해서는 HTTPS 인증서 확인, JWT 토큰을 활용한 인증처리가 필요하다.
- 로컬에서는 `kubectl proxy` 를 통해서 쉽게 확인할 수 있다.
  
## 3. POD 에서 K8S API 호출하기 (첫번째)

### CURL로 API 호출해보기 

로컬에서 K8S API 호출을 위한 기본적인 개념을 이해했으니, 이제 POD 안에서 호출하는 방법을 살펴보자.

1. pod 는 기본적으로 `kubectl` 과 같은 명령어를 가지고 있지 않다. 따라서 pod 에서 다음의 3가지를 알아내야 한다.
    - API 서버의 위치
    - API 서버의 HTTPS 인증서 대응
    - API 서버에 요청하기 위한 인증토큰

2. 테스트용 POD 구동하기
    동작 확인을 위해서 간단한 curl POD를 띄워보자.
    ```bash
    kubectl run curl -it --rm --image curlimages/curl -- sh
    ```

3. POD 에서 API 주소 찾기 
    Pod 에서는 k8s 관련 정보가 환경변수로 제공된다. 다음과 같이 확인해보자.
    ```bash
    $ env | grep KUBERNETES
    KUBERNETES_SERVICE_PORT=443
    KUBERNETES_SERVICE_HOST=10.0.0.1
    KUBERNETES_SERVICE_PORT_HTTPS=443
    ...그 밖의 다른 환경변수들...
    ```
    정보를 확인하였으면 다음과 같이 호출해보자
    ```bash
    $ curl https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT -k
    {
      "kind": "Status",
      "apiVersion": "v1",
      "metadata": {
      },
      "status": "Failure",
      "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
      "reason": "Forbidden",
      "details": {
      },
      "code": 403
    }
    ```

4. 클러스터 내부의 DNS 활용하여 API 질의하기
    Pod 내부에서 매번 환변경수를 사용하기는 불편하다 이 때는 클러스터에서 제공하는 내부 DNS 를 사용할 수 있다.
    ```bash
    $ curl https://kubernetes -k
    ... 응답 확인...
    ```

5. https 인증서 처리(서버 인원 확인)
    매번 `-k` 옵션을 붙이자니 찜찜하다. 이를 해결해보자. 먼저 k8s 클러스터에서 제공하는 secret 를 확인하자.
    ```bash
    $ ls /var/run/secrets/kubernetes.io/serviceaccount/ 
    ca.crt namespace token
    ```
    바로 여기에 있는 `ca.crt` 가 서버 인증서이다. 인증서를 사용해서 접근해보자.
    ```bash
    $ curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes 
    ... 응답 확인...
    ```
    매번 `--cacert` 옵션을 주려니 귀찮다. CURL 환경변수로 등록해보자.
    ```bash
    $ export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    ```
   
6. 여기까지 실행하면 curl 클라이언트가 API 서버를 신뢰할 수 있게 되었다. 하지만 아직 token 을 전달하지 않았기 때문에 API 응답이 권한이 없다고 나온다. 토큰도 전달해보자.
    ```bash
    $ TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) 
    $ curl -H "Authorization: Bearer $TOKEN" https://kubernetes/ 
    # 응답 확인...
    {
      "kind": "Status",
      "apiVersion": "v1",
      "metadata": {
      },
      "status": "Failure",
      "message": "forbidden: User \"system:serviceaccount:default:default\" cannot get path \"/\"",
      "reason": "Forbidden",
      "details": {
        "kind": "pods"
      },
      "code": 403
    }
    ```
   
7. 서비스 어카운트 권한이 필요함
   위와 같이 ca.crt, token 을 모두 전달하였을 때 로컬에서 호출했을 때와 동일한 결과를 기대하였지만, 실제로는 그렇지 않다. 그 이유는 POD에 제공된 서비스 어카운트 `system:serviceaccount:default:default` 에 **권한이 부여되어 있지 않기 때문**이다.
   이를 위해서는 서비스 어카운트와 role 을 부여해야한다.

### pod 에서 curl로 API 호출하기 정리
- pod 에서 curl 로 API를 호출하려면 내부 dns 를 사용하여 `https://kubernetes` 로 호출하면된다.
- 서버 신원 확인을 위한 ca.crt, API 인증을 위한 token 정보는 `/var/run/secrets/kubernetes.io/serviceaccount/` 디렉토리에 있다.
- 하지만 pod 에 연결된 서비스 어카운트에 제대로된 role 이 부여되어 있지 않다면 (권한이 없다면) API를 정상적으로 호출할 수 없고 403 응답을 받는다.

## 4. 서비스 어카운트와 Role 이해하기

### RBAC - Role Based Access Control 

 초기 버전의 k8s 클러스터는 `RBAC`가 비활성화 되어 있어서 모든 POD에서 K8S API를 호출할 수 있었다. 그러나 최근의 K8s 클러스터에서는 `RBAC`가 기본적으로 활성화 되어 있고, 
 이 때문에 POD 안에서 K8S API를 호출하려면 해당 POD에게 부여된 서비스 어카운트가 API를 호출할 수 있는 `Role` 을 부여받고 있어야 한다. `Role` 이라는 것은 말 그대로 어떤 역할을 부여 해준다는 것이며
 `RBAC`는 **부여한 역할에 따라서 접근을 제어하는 기능**을 말한다.

### 서비스 어카운트와 기본 서비스 어카운트
 서비스 어카운트는 말 그대로 Account 즉, K8S API 서버에 접근하기 위한 계정을 말한다. Pod 가 구동될 때에는 자동으로 Pod 에게 서비스 어카운트가 부여된다. 
 부여되는 서비스 어카운트는 다음과 같이 확인할 수 있다.

```bash
$ kubectl get sa # serviceaccoutn 의 약어
NAME      SECRETS   AGE
default   1         7d
```

서비스 어카운트는 네임스페이스 단위로 관리되며 새로운 k8s 리소스가 생성될 때 서비스 어카운트가 없다면 자동으로 `default` 서비스 어카운트가 생성된다. 앞서 Pod 안에서 curl 로 API 호출했을 때 403 이 반환되는데
이 때 서비스 어카운트를 다음과 같이 표현한다는 것을 알 수 있다.

```bash
system:serviceaccount:<namespace>:<service account name>
system:serviceaccount:default:default
```

### 새로운 서비스 어카운트 생성하기
`system:serviceaccount:default:default` 서비스 어카운트는 `default` 네임스페이스의 모든 pod 가 공통으로 사용하는 서비스 어카운트다. API 호출이 필요한 POD를 위한 새로운 서비스 어카운트를 생성해보자.

```bash
$ kubectl create serviceaccount my-serviceaccount
serviceaccount/my-serviceaccount created

$ kubectl describe serviceaccount my-serviceaccount
Name:                my-serviceaccount
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   my-serviceaccount-token-5wch8
Tokens:              my-serviceaccount-token-5wch8
Events:              <none>
```

### Role 을 생성하여 권한 부여하기

서비스 어카운트만으로는 앞서 기본으로 제공된 `default:default` 서비스 어카운트와 다를게 없다. 권한을 부여하기 위한 Role 을 생성하고 이 Role 을 서비스 어카운트에 연결해보자. 이 과정을 RoleBinding 이라고 한다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: role-for-k8s-api-call
  namespace: default
rules:
- apiGroups:
  - "" # 리소스에 그룹을 지정할 수 있으나, 여기에서는 사용하지 않으므로 "" 으로 입력한다.
  resources:
  - pods # 어떤 리소스를 확인할 것인지 지정할 수 있다 복수형으로 입력해야한다. ex) services, configmaps, secrets ..., 여기에서는 pod 를 대상으로 한다.
  verbs: # 개별 Pod를 이름으로 가져오고, 목록을 확인할 수 있는
  - get
  - list
```

( 내용이 길어서 yaml 로 작성했지만 다음과 같이 명령어로도 가능하다.)

`kubectl create role role-for-k8s-api-call --verb=get --verb=list --resource=pods -n default`


작성한 yaml 을 적용하고, 생성한 role 을 서비스 어카운트에 연결한다.

```bash
$ kubectl apply -f role-for-k8s-api-call.yaml
role.rbac.authorization.k8s.io/role-for-k8s-api-call created

$ kubectl create rolebinding api-call --role=role-for-k8s-api-call --serviceaccount=default:my-serviceaccount -n default
rolebinding "api-call" created
```

### 서비스 어카운트와 Role 정리
- 역할에 따라 API 에 접근할 수 있는 체계를 RBAC 라고 한다.
- K8S API에 접근하기 위한 계정을 서비스 어카운트라고 한다.
- 서비스 어카운트는 지정되지 않으면 `default` 가 부여된다.
- 별도의 서비스 어카운트를 만들고 Role 을 생성하여 연결하는 것을 롤바인딩이라고 한다.

## 5. 서비스 어카운트 지정하여 Pod 실행하기

자 이제 필요한 서비스 어카운트도 생성하였고, Role 도 생성하여 롤바인딩도 마쳤다. 생성한 서비스 어카운트를 Pod 에 적용해보자.
- 서비스 어카운트는 Pod 가 생성될 때 지정되며 도중에 변경할 수 없다. 
- 그리고 Pod 는 하나의 서비스 어카운트만 가질 수 있다.
- Pod 에 서비스 어카운트를 지정하는건 명령어로는 불가능하다. yaml 파일을 작성하자.

### 서비스 어카운트가 적용된 Pod 생성 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-with-serviceaccount
spec:
  serviceAccountName: my-serviceaccount
  containers:
    - image: curlimages/curl
      command: # pod 가 바로 종료되지 않도록 하는 명령어
        - sleep
        - "999999"
      name: curl-with-serviceaccount 
```

```bash
$ kubectl apply -f curl-with-serviceaccount.yaml
pod/curl-with-serviceaccount created
```

### 다시 Pod 내부에서 API 호출하기

다시 pod 안에서 K8S API를 호출해보자.

```bash
$ kubectl exec -it curl-with-serviceaccount -- sh

/ $ # pod 안으로 들어옴
/ $ export TOKEN=$(car /var/run/secrets/kubernetes.io/serviceaccount/token) # TOKEN 을 변수에 등록
/ $ export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt # ca.crt 를 curl 이 신뢰할 수 있도록 curl 변수에 등록
/ $ curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods # pod 목록을 조회하는 k8s 클러스터 API 호출
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/default/pods",
    "resourceVersion": "184545912"
  },
  "items": [
    {
      "metadata": {
        "name": "bb-5c747f77c8-mw5nw",
        "generateName": "bb-5c747f77c8-",
        "namespace": "default",
        ....
        .... # 전체 pod 목록이 출력된다.
        }
    }  
  ]
}
```

이제 원하는대로 Pod 안에서 K8S API를 호출하고 그 결과를 확인할 수 있다.

## 6. API Client 사용하기

위와 같이 curl 을 사용하여 Pod 안에서 K8S API를 호출할 수 있지만, 매번 수동으로 호출할 수는 없다. K8S 공식 사이트에서 안내하고 있는 언어별 Client 라이브러리를 사용해보자.
- [Go 라이브러리](https://github.com/kubernetes/client-go/) 
- [Python 라이브러리](https://github.com/kubernetes-client/python/)
- [Java 라이브러리](https://github.com/kubernetes-client/java/)

나의 경우에는 Spring Boot 애플리케이션에서 사용하기 위해서 Java 라이브러리를 선택했고, 코드는 Kotlin 으로 작성하였다. "default" 네임스페이스의 전체 pod 목록을 확인하는 샘플코드는 다음과 같다. 

```kotlin
val client: ApiClient = Config.defaultClient()
Configuration.setDefaultApiClient(client)

val api = CoreV1Api()
val list = api.listNamespacedPod("default", null, null, null, null, null, null, null, null, null, false)
for (item in list.items) {
    println(item.metadata!!.name)
}
```

## 정리요약

K8S 클러스터 안에서 Pod 가 K8S API를 호출하기 위해서는 `서비스 어카운트`, `Role`, `RoleBinding` 이 필요하다. 
pod 내부에서 curl 과 같이 직접 raw 하게 호출하려면 token, ca.crt 값을 신경써야 한다. 
하지만 K8S 공식 사이트에서 안내하고 있는 API Client 라이브러리르 사용하면 손쉽게 API를 호출하고 그 결과를 확인할 수 있다.

{{< imageFull src="/images/posts/kubernetes/serviceaccount/diagram.png" title="Pod 가 호출할 때 연결구조" border="false" >}}

k8s API를 호출하기 위한 작업들이 복잡하게 나열된것 같지만, 알고나면 크게 어렵지는 않다. 다만 k8s 에 대해서 잘 이해하고 있지 않다면, 개념을 잡는데 시간이 걸린다.
이렇게 또 서비스 어카운트와 Role, RoleBnding 에 대해서 좀 더 이해하게 되었다.


## 참고자료
- 쿠버네티스 API 설명 : https://kubernetes.io/ko/docs/concepts/overview/kubernetes-api/
- access cluster api : https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/
- kubernetes api reference : https://kubernetes.io/docs/reference/kubernetes-api/
- API Client 설명 : https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/#using-official-client-libraries 
- 클라이언트 라이브러리 소개 페이지 : https://kubernetes.io/docs/reference/using-api/client-libraries/ 


https://kubernetes.io/ko/docs/tasks/run-application/access-api-from-pod/