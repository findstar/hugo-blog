---
date: 2021-08-22T21:22:30+09:00
group: blog
image: /images/posts/kubernetes/nginx-ingress-controller-logo.png
tags: ["nginx ingress", "forwarded option"]
title: "Nginx ingress 에서 X-forwarded header 사용법"
url: /2021/08/22/nginx-ingress-controller-use-forwarded-for-option
type: post
summary: "K8s 환경에서 Nginx ingress controller 를 사용하면서 앞단의 트래픽에서 전달하는 X-forwarded 값을 넘겨받는 방법에 대해서 살펴보았다."
---

# 개요

Nginx ingress controller 를 사용하는 K8S 환경에서 네트워크 구성에 따라서 X-forwarded 값을 전달받아야 하는 경우가 있다. 이 때 적용할 수 있는 Nginx ingress controller 의 `use-forwarded-for` 옵션에 대해서 살펴보자.  

## DSR LB 만 존재하는 경우

먼저 K8S 환경에서 Nginx ingress controller 를 사용한다고 가정하겠다. 일단 k8s 클러스터의 앞단에 LB 가 있고 이 LB 는 **DSR** 모드로 작동해서 외부의 트래픽을 바로 Ingress 로 전달한다고 생각하면 다음과 같은 형태로 트래픽이 유입된다.

{{< imageFull src="/images/posts/kubernetes/nginx-ingress/k8s-network-case1.png" title="k8s network case1" border="true" >}}

여기서는 Nginx ingress controller 가 바로 reverse proxy 역할을 하기 때문에 Service 로 전달되는 트래픽에는 Nginx ingress 에서 전달해주는 `X-forwarded-*` 값을 사용하게 된다. 이 경우에는 nginx ingress controller 의 `use-forwarded-for` 옵션이 false 로 지정한다. 만약 클라이언트가 보내는 `X-forwarded-*` 값이 있더라도 이를 무시하고 덮어쓴다.)  

## 앞단에 별도의 Reverse proxy 가 존재하는 경우

경우에 따라서는 L7 proxy를 k8s 클러스터 앞단에 위치 시킬수도 있는데 이 경우에는 다음과 같은 형태의 트래픽 유입이 일어난다. (나의 경우에는 Haproxy 를 두었다.)

{{< imageFull src="/images/posts/kubernetes/nginx-ingress/k8s-network-case2.png" title="k8s network case2" border="true" >}}

이 경우에는 실제로 클라이언트의 트래픽이 서비스까지 전달되려면 reverse proxy 를 2번 거치게된다. (첫 번째는 L7 Proxy, 두 번째는 Nginx ingress) 
따라서 nginx ingress 입장에서는 L7 proxy 에서 전달되는 `X-forwarded-*` 값을 신뢰하고 연결되는 서비스로 넘겨줘야한다. 
이 경우에는 nginx ingress controller 의 `use-forwarded-for` 옵션을 true 로 지정한다. 


## 적용 방법 

nginx ingress controller 에 연결되는 config map 에 값을 설정하면 된다. 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  name: ingress-forwarded
  namespace: ingress-nginx
data:
  ## ... other values 생략.. ##
  use-forwarded-headers: "true"
```

## 요약 

K8s 와 연결되는 네트워크 구성과 그 앞단의 프록시 유무에 따라서 `use-forwarded-headers` 값을 적절하게 사용할 수 있다.

## 참고자료 
- https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#use-forwarded-headers


