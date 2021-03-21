---
date: '2019-09-08 23:42:45 +09:00'
group: blog
image: /images/posts/haproxy/haproxy-logo.png
tags:
- haproxy
- "haproxy acl logging"
- "haproxy custom variable logging"
title: Haproxy에서 acl을 log에 기록하기
url: /2019/09/08/haproxy-acl-logging
type: post
draft: false
summary: "Reverse Proxy 로 활용중인 haproxy 를 운영하면서 한가지 불편한 점이 있었는데,
  설정을 통해서 정의한 custom variable(ACL) 을 log에 기록하는 방법을 알기가 어려웠다는 점이었다.
  cfg 파일에 acl 로깅을 활성화 하는 방법에 대해서 알아보았다. "
---

## Haproxy 의 ACL 설정

reverse proxy 로 활용하면서 header 의 값을 판단하거나, source 의 IP 대역을 확인하거나, 
또는 기타 특정 backend 연결을 위해서 acl을 정의한다.

```
    # ACL "vpc-network" 선언
    acl vpc-network src 10.10.1.0/16

    # ACL "allow_url" 선언
    # whitelist url 목록 파일 지정
    acl accept_url url_beg -i -f /etc/haproxy/accept_url.list

    # ACL "found-header" 선언
    acl found-header req.hdr_val(my_custom_header) -m found
```

### 조건에 따른 분기 처리를 위한 변수 획득

예를들어 header 의 특정 값을 변수로 획득하여 조건에 따라 다른 백엔드 서버로 요청을 처리를 하고 싶다고 가정해보자. 
이 값을 내부 변수로 획득하기 위해서는 몇가지 설정을 통해서 처리할 있다.

apache 에서는 다음과 같이 처리할 수 있다.

```
#apache

    # my_custom_header 라는 값이 존재하는지 확인하고 이 값을 
    # CUSTOM_HEADER 라는 변수에 할당한다.
    SetEnvIf my_custom_header (.+) CUSTOM_HEADER=$1

    # 로그에 CUSTOM_HEADER 라는 변수값을 기록한다.
    LogFormat "%{CUSTOM_HEADER}e %{Host}i %D [%{%FT%T%z}t] \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

nginx 에서는 다음과 같이 처리할 수 있다.

```
#nginx 

    # http_my_custom_header 라는 값을 찾아 custom_header 라는 변수에 할당한다
    # 값이 없다면 공백으로 지정
    # 값지 존재한다면 해당값을 변수에 저장한다.
    map $http_my_custom_header $custom_header {
        "-"     "";
        default $http_my_custom_header;
    }

    # custom_header 라는 변수값을 로그에 기록한다.
    log_format main         '$custom_header $forward_ip $http_host $request_time [$time_iso8601] ';
```


haproxy document 를 확인하면 애석하게도 acl 을 곧바로 로그에 남기는 방법이 존재하지 않는다.
apache 나 nginx 보다는 조금 불편한 방법을 거쳐야 한다. 다음과 같이 해보자.

```
#haproxy

    # request header 에 들어 있는 my_custom_header 라는 변수값을 찾아 이를
    # req.my_custom_variable 이라는 값으로 지정한다.
    http-request set-var(req.my_custom_variable) req.hdr(my_custom_header)
    
    # 특정 acl 에 따라서 string 값으로 지정할 수 있다.
    http-request set-var(req.my_custom_variable) str("other_value") if my_acl
    
    # 길이 제한 10으로 이 값을 capture 하자.
    http-request capture var(req.my_custom_variable) len 10
    
    # 위에서 캡처한 값을 로그에 기록한다.
    # 캡쳐한 값이 여러개라면 index 번호는 0, 1, 2.. 순으로 늘어난다.
    log-format "%[capture.req.hdr(0)]\ %Tt [%trl]\ \"%HM\ %HU\ %HV\"\ %ST\ %B\"
```

이렇게 하면 원하는 값을 acl 에 따라서 지정할 수 있고, 이를 로그에서 확인할 수 있다. 
나의 경우에는 내부 내트워크 에서 접속인지 아닌지 로그에서 바로 확인하는 용도로 사용한다.
테스트는 haproxy 1.9 버전에서 정상적으로 동작하는 것을 확인했다.

#### 요약

haproxy 에서 acl 값을 log 에 바로 남기는 방법을 알아보았다. apache 처럼 바로 acl 을 변수값으로 확인해서 기록하는 방법이 있으면 편하겠지만.
조금 불편한 방법을 거쳐야 한다. 또한 capture 한 값을 index 값처럼 0, 1, 2.. 로 확인하려니 설정이 지저분해지는 단점이 있다.

### 참고
 * 크롬 브라우저에서 header 에 특정 값을 추가하는 extension : https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj
 * stat haproxy 매뉴얼 : https://cbonte.github.io/haproxy-dconv/1.8/management.html#9
 * socket 으로 haproxy stat 획득하기 : https://makandracards.com/makandra/36727-get-haproxy-stats-informations-via-socat
 * stat 수집 script : https://gist.github.com/bpaquet/7153979
 * stat 설명 : https://gist.github.com/alq666/20a464665a1086de0c9ddf1754a9b7fb
