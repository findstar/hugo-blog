---
date: '2018-06-24'
group: blog
tags:
- nginx
- "rate litmiting"
- limit_req_zone
title: nginx reverse proxy로 동일 IP 중복 요청 제한
url: /2018/06/24/nginx-rate-limiting
type: post
---

Nginx 로 reverse proxy를 구성할 때 과도한 요청에 대한 제한을 두기 위해서 `limit_req` 모듈을 적용해보았다.

<!--more-->

[nginx](http://nginx.org/)에서 지원하는 [limit_req](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html) 모듈을 사용하면,
동일한 IP 에서 과도한 요청이 유입될 때, 이를 제한할 수 있다.

## Limit req 설정

```
limit_req_zone $binary_remote_addr zone=depend_rate_limit:10m rate=10r/s;

# binary_remote_addr : 클라이언트의 IP를 기준으로 제한하겠다는 의미.
# zone name : depend_rate_limit
# share memory assign : 10M
# rate : 10 request / second

```

### 하나씩 살펴보자면,
  - limit_req_zone 이라는 것은 요청-request를 확인하고 이를 제한하기 위해서 특정한 영역(zone)을 선언한다는 의미이다.
  - $binary_remote_addr 은 nginx 에서 기본적으로 제공하는 내장 변수로, 클라이언트의 IP를 기준으로 제한을 하겠다는 의미이다.
  - zone=depend_dos 라는 것은 zone 의 이름을 설정하는 것으로, zone의 이름은 임의로 변경이 가능하다
  - 30m 이라는 것은 zone에서 활용가능한 `share memory size` 로. 10M 정도면 충분하다.
  - rate 는 요청-request 의 비율로, 여기에서는 초당 10개 이상의 요청-request 이 유입되면 제한을 하겠다는 의미이다. (r/s, r/m 가능)

### 이제 다음과 같이 적용하면 된다.

```
http {
    limit_req_zone $binary_remote_addr zone=depend_rate_limit:10m rate=10r/s;

    ...

    server {

        ...

        location / {

            ...

            limit_req zone=depend_rate_limit burst=5;
        }


```

`burst` 를 적용한 것은 rate(여기서는 10r/s) 이상의 request-요청에 대해서 5개 까지는 queue에 보관하도록 하고, 그 이상은 에러를 반환하게 한다는 의미이다.

### 에러 반환

기본적으로 정의한 rate 를 넘어서는 request-요청에 대해서는 `503` 에러를 반환하는데. 이를 변경할 수 있다.
429 status code는 `Too Many Requests`를 의미한다. [참고](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)

```

limit_req               zone=depend_rate_limit burst=5 nodelay;
limit_req_status        429;

```

### log level 설정

제한에 걸린 request-요청을 기록할 때 log level 을 설정할 수 있다.

```

limit_req               zone=depend_rate_limit burst=5 nodelay;
limit_req_status        429;
limit_req_log_level     error;

```

## 제한에서 제외할 IP 설정

실제로 limit_req 모듈을 적용하기 전에, 내부 네트워크 대역이나, 특정 IP에 대해서는 제한을 두지 않을 수 있다.

```

geo $apply_limit {
    default         $binary_remote_addr;
    10.10.0.0/16    '';                   # 내부 네트워크 대역 10.10.*.* 은 access limit 사용안함
    211.33.188.246  '';                   # 외부의 특정 IP 211.33.188.246 는 access limit 사용안함
}

...
...

limit_req_zone $apply_limit zone=depend_rate_limit:10m rate=10r/s;

...

```

`geo` 모듈을 사용해서 client ip 를 확인해서 `$apply_limit` 이라는 변수를 새롭게 할당했다.
**10.10.*.*** 대역 이거나, (내부 네트워크 대역인 경우), 외부의 특정 **211.33.188.246** 인 경우에는 빈값이 지정된다.

이렇게 하고 나서 `limit_req_zone` 에 정의한 `$apply_limit` 변수를 사용하면, 예외로한 IP 에 대해서는 접속제한이 동작하지 않는다.



### 참고
  - http://nginx.org/en/docs/http/ngx_http_limit_req_module.html
  - https://sarc.io/index.php/nginx/99-2014-03-18-14-30-00
  - https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429
