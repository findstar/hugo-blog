---
date: '2021-04-14 20:55:12 +09:00'
group: blog
image: /images/posts/istio/istio-and-envoy.png
tags:
- "istio"
- "traffic management"
- "kubernetes"
title: "istio의 트래픽 관리 기능 살펴보기"
url: /2021/04/14/istio-traffic-management
type: post
summary: "istio 의 트래픽 관리 기능에 대해서 살펴보았다." 
---
# istio의 트래픽 관리 기능 살펴보기

## 배경

istio 스터디의 두번째 내용으로 istio 의 트래픽 관리 기능을 살펴보았다. 
매뉴얼을 참고로 하여 `gateway`, `virtual service`, `destination rule` 을 살펴보았다.

## 구동 환경

- istio 1.9.1
- k8s on docker desktop [istio on docker desktop 참고](/2021/03/25/install-istio-on-docker-desktop) 
  
### 트래픽 관리 기능

istio 는 envoy 를 사용하여 sidecar 가 적용된 pod 의 트래픽을 관리할 수 있다. 순수하게 k8s 의 기능을 사용하면
`ingress`, `deployment` 만 사용하게 되었을 텐데 `istio` 를 사용하면 `gateway`, `virtualService`, `destinationRule` 을 다루게 된다. 

#### Gateway

먼저 `Gateway` istio 서비스 메쉬로 유입되는 관문을 나타낸다. 게이트웨이는 노출되는 포트, 프로토콜, 호스트, TLS 정보를 담고 있다.
다음은 이전 포스트에서 예제로 실행한 bookinfo-gateway.yml 파일에 들어 있는 내용이다.
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*"
```

위의 yml 을 해석해보자면 먼저 `selector` 에 나타나 있는 `istio: ingressgateway` 는 로드밸런서로 동작하는 `istio-ingressgateway` 에서 실행되라는 의미가 된다.
operator 를 통해서 설치한 ingressgateway 컴포넌트를 통해 실행된 pods 의 라벨을 확인해보자
```shell
$ kubectl get pods/istio-ingressgateway-67d647b4-b6k84 --show-labels -n istio-system # (pods 이름은 그때 그때 각기 다를 수 있다.)
istio-ingressgateway-67d647b4-b6k84   1/1   Running   0     31m   app=istio-ingressgateway,chart=gateways,heritage=Tiller,install.operator.istio.io/owning-resource=unknown,istio.io/rev=default,istio=ingressgateway,operator.istio.io/component=IngressGateways,pod-template-hash=67d647b4,release=istio,service.istio.io/canonical-name=istio-ingressgateway,service.istio.io/canonical-revision=latest,sidecar.istio.io/inject=false
```

결과에서 보면 `istio=ingressgateway` 라는 라벨을 확인할 수 있다. ingressgateway 에서 실행되는 논리적인 게이트웨이 나타낸것이 바로 `Gateway` 컴포넌트이다.
게이트웨이는 `VirtualService` 와 함께 사용되어 트래픽을 제어하게 된다.

#### VirtualService

`VirtualService` 는 트래픽 라우팅과 관련된 설정이다. `gateway` 와 결합하여 트래픽을 제어한다. 다음은 bookinfo 예제에서 사용한 yml 이다.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - "*"
  gateways:
    - bookinfo-gateway
  http:
    - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
        - uri:
            exact: /login
        - uri:
            exact: /logout
        - uri:
            prefix: /api/v1/products
      route:
        - destination:
            host: productpage
            port:
              number: 9080
```

spec 을 살펴보면 연결할 게이트웨이를 `gateways` 에 `bookinfo-gateway`로 표시해주었다. `http` 설정은 http 프로토콜만을 설명하는 것은 아니고
`HTTP`, `HTTP2`, `GRPC` 를 포함한다. 그 다음으로 매칭 룰을 지정하는데 (`match`) `exact` 는 정확히 매칭되어야 하는 경우 `prefix` 는 uri 의 앞부분만 매칭되면 되는경우로 해석된다.
`route` 에서는 연결할 대상을 지정하는데 `host` 가 `productpage` 로 되어 있는 것은 앞선 예제에서 등록한 `Service` 컴포넌트로 연결하라는 의미가 된다. (k8s 내부에서 서비스명으로 바로 접속할 수 있기 때문)

`Gateway` 와 `VirtualService` 는 실제로 별도의 pods 를 실행하는 것은 아니고 *ingressgateway 의 envoy 설정을 변경한다.(중요)* ingressgateway 의 라우트 설정은 다음의 명령어로 확인할 수 있다. 
자세히 보면 prefix 로 지정한 uri 는 match 뒤에 `*` 표시가 붙어 있는 것을 확인할 수 있다.

```shell
$ istioctl pc routes istio-ingressgateway-67d647b4-b6k84.istio-system
NOTE: This output only contains routes loaded via RDS.
NAME        DOMAINS     MATCH                  VIRTUAL SERVICE
http.80     *           /productpage           bookinfo.default
http.80     *           /static*               bookinfo.default
http.80     *           /login                 bookinfo.default
http.80     *           /logout                bookinfo.default
http.80     *           /api/v1/products*      bookinfo.default
            *           /healthz/ready*
            *           /stats/prometheus*
```

`VirtualService` 에는 라우팅을 위한 좀 더 디테일한 설정도 가능하다

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews.prod.svc.cluster.local
  http:
  - headers:
      request:
        set:
          test: "true"
    route:
    - destination:
        host: reviews.prod.svc.cluster.local
        subset: v2
      weight: 25
    - destination:
        host: reviews.prod.svc.cluster.local
        subset: v1
      headers:
        response:
          remove:
          - foo
      weight: 75
```

대표적으로 header를 수정할 수도 있고, destination 에 weight 값을 지정할 수도 있다. (가중치 값은 합쳐서 100이 되어야 한다.) 
위의 yml 에서는 `test:true` 라는 헤더 값을 추가하고 v1 subset 에 대해서만 응답에서 foo 라는 헤더값을 제거한다. 

#### DestinationRule

`VirtualService` 가 어느 서비스로 트래픽을 라우팅할지 결정했다면, `DestinationRule` 은 어디로 라우팅할지 결정된 이후에 트래픽에 적용되는 정책을 정의하는 기능이다. (트래픽이 어디로 갈지 결정된 이후에 트래픽을 어떻게 보낼지 정의한 것이라고 이해하면 된다.) 이렇게 이야기 하면 감이 잘 오지 않는데 간단한 예시로 pods 의 라벨에 따라서 부하를 분산 시킬 때 사용하는 것이 바로 `DestinationRule` 이다.   

다음은 [bookinfo 예제](https://istio.io/latest/docs/examples/bookinfo/)에서 사용하는 `DestinationRule` yml 이다.

```yml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ratings
spec:
  host: ratings
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v2-mysql
    labels:
      version: v2-mysql
  - name: v2-mysql-vm
    labels:
      version: v2-mysql-vm
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: details
spec:
  host: details
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
---
```

자세히 보면 `subsets` 라고 보이는데 pods 의 label 을 그룹핑한 단위라고 보면 된다. pods 를 하나의 subset 으로 묶은 다음에 트래픽을 제어하는데 별도의 정책이 지정되지 않았으므로 라운드 로빈이 지정된다.
설정 가능한 옵션은 [문서](https://istio.io/latest/docs/reference/config/networking/destination-rule/#LoadBalancerSettings-SimpleLB)를 참고하자.
다음과 같이 기본 정책을 지정하고 특정 subset 에 대해서만 다른 정책을 취할 수도 있다.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN # 커넥션이 더 적은 쪽으로 연결한다. 
  subsets:
  - name: testversion
    labels:
      version: v3
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN # 라운드 로빈으로 연결한다(기본값)
```

다음은 user 라는 쿠키값을 해싱한 결과값을 기반으로 연결을 설정한다.

```yaml
 apiVersion: networking.istio.io/v1alpha3
 kind: DestinationRule
 metadata:
   name: bookinfo-ratings
 spec:
   host: ratings.prod.svc.cluster.local
   trafficPolicy:
     loadBalancer:
       consistentHash:
         httpCookie:
           name: user
           ttl: 0s
```


#### 기타 옵션 

다음은 istio 공식 사이트에서 제공한는 `Gateway` 예제이다.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
  namespace: some-config-namespace
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      httpsRedirect: true # HTTP 로 접근시 301 리다이렉트 반환
  - port:
      number: 443
      name: https-443
      protocol: HTTPS
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      mode: SIMPLE # 이 포트에 대해서 HTTPS 활성화
      serverCertificate: /etc/certs/servercert.pem
      privateKey: /etc/certs/privatekey.pem
  - port:
      number: 9443
      name: https-9443
      protocol: HTTPS
    hosts:
    - "bookinfo-namespace/*.bookinfo.com"
    tls:
      mode: SIMPLE # 이 포트에 대해서 HTTPS 활성화
      credentialName: bookinfo-secret # k8s secret 에 등록된 이름
  - port:
      number: 9080
      name: http-wildcard
      protocol: HTTP
    hosts:
    - "*"
  - port:
      number: 2379 # 내부 서비스를 2379포트 번호를 통해서 외부에 노출한다.
      name: mongo
      protocol: MONGO # 프로토콜은 HTTP, HTTPS, GRPC, HTTP2, MONGO, TCP, TLS 중 하나여야 한다.
    hosts:
    - "*"
```

### 트래픽의 실제 연결 흐름

새로운 용어가 계속 나오기 때문에 실제 트래픽이 어떻게 제어되는지 헷갈려서 정리해보았다. 

* 로드밸런서로 부터 트래픽이 유입된다. (`$ kubectl get svc -n istio-system` 결과값을 생각해보자.)
* 로드밸런서 타입의 서비스(ingressgateway controller)는 실제로는 `istio-ingressgateway` 라는 pods 를 실행시킨다.
  - k8s 의 ingress controller 가 실제로 nginx pods 를 띄우는 것과 동일하다. 다만 다른점은 envoy 를 띄운다는점
* IngressGateway 의 pods 는 Gateway 와 VirtualService 에 의해서 configuration 이 결정된다. 
  - `$ istioctl pc routes istio-ingressgateway-67d647b4-b6k84.istio-system` 와 같이 등록된 라우팅 설정을 확인할 수 있다.
  - Gateway 는 포트, 프로토콜, TLS 등 인증을 설정한다.
  - VirtualService 는 어떤 서비스로 연결될 것인지 라우팅을 설정한다.
* 이것만으로도 트래픽이 연결되지만 트래픽 연결 정책을 추가하고 싶은 경우 DestinationRule 을 지정할 수 있다. 
  - 여기서는 pods 의 라벨을 하나로 그룹핑하여 Subset 이라고 한다.

{{< imageFull src="/images/posts/istio/traffic/istio-request-routing.png" title="istio traffic routing by envoy ingress-gateway" border="false" >}}

### 참고

- https://istio.io/latest/docs/reference/config/networking/gateway/
  https://istio.io/latest/docs/reference/config/networking/virtual-service/
  https://istio.io/latest/docs/reference/config/networking/destination-rule/
  https://istio.io/latest/docs/ops/diagnostic-tools/proxy-cmd/
- https://istio.io/latest/docs/examples/bookinfo/
- https://blog.jayway.com/2018/10/22/understanding-istio-ingress-gateway-in-kubernetes/

