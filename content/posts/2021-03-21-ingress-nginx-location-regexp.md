---
date: '2021-03-21 17:43:01 +09:00'
group: blog
image: /images/posts/kubernetes/Kubernetes-logo.png
tags:
- "kubernetes"
- "nginx regexp"
title: "nginx ingress 에서 정규표현식 경로 지정하는 방법"
url: /2021/03/21/kubernetes-nginx-ingress-controller-regexp-path
type: post
---

k8s 에서 nginx ingress controller 의 경로를 지정할 때 정규표현식을 
사용해서 등록하는 방법을 알아보았다.


<!--more-->

# nginx ingress controller 에서 regexp location 지정하는 방법

## 배경

k8s 에서 nginx ingress controller 의 path 을 지정할 때에 기본적으로는 string match 방식으로 path를 구분한다. 
대부분의 경우에는 별 문제가 없지만 특정 요구사항에 따라서 따라서 path 를 식별하는데 regexp 를 적용하고 싶은 경우가 있다. 
ingress controller 로 동작하는 nginx 의 configuration을
바로 수정하기는 어려우니 ingress controller 에서 지원하는 방식으로 해법을 찾아보았다. 

## 방법

간단히 말해서 다음과 같이 `nginx.ingress.kubernetes.io/use-regex` annotation 을 적용하면 된다.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
```

annotation 을 지정한 다음에 다음과 같이 path 지정을 했다면

```yaml
spec:
  rules:
  - host: test.com
    http:
      paths:
      - path: /foo/.*
        backend:
          serviceName: test
          servicePort: 80
```

nginx ingress controller 에 적용되는 nginx config 는 다음과 같은 형태가 된다. 

```
location ~* "^/foo/.*" {
  ...
}
```

### 주의사항

주의 할점은 regexp 가 적용된 path 의 경우에 매칭되는 패턴이 여러개라면 첫 번째 일치하는 패턴이 적용된다는 점이다.
패턴을 등록할 때 주의하자. 

### 참고
- https://kubernetes.github.io/ingress-nginx/user-guide/ingress-path-matching/

