---
date: '2018-08-13 23:11:17 +09:00'
group: blog
image: /images/posts/haproxy/haproxy-logo.png
tags:
- haproxy
- "dos attack"
- "rate limit"
- "haproxy 1.8"
title: Haproxy를 활용해서 dos 공격 방어하기
url: /2018/08/13/block-dos-attacks-using-haproxy-rate-limit
type: post
---

앞서 [Haproxy 로 로드밸런서를 구성하는 방법](/2018/07/27/install-haproxy)을 살펴보았다. Haproxy를 로드밸런서로 운영하면서 DOS 공격이 유입되어 서비스 운영에 장애가 발생하였다.
이번에는 Haproxy 의 기능을 이용해서 단순한 패턴의 DOS 공격을 방어하는 방법을 살펴보았다.

<!--more-->

### DOS 란?

먼저 DOS 공격의 정의를 살펴보았다.(DDOS 가 아니다, DOS다)
[위키피디아](https://ko.wikipedia.org/wiki/%EC%84%9C%EB%B9%84%EC%8A%A4_%EA%B1%B0%EB%B6%80_%EA%B3%B5%EA%B2%A9) 에 따르면
DOS 란 "Denial of Service attack". 서비스 거부 공격으로 악의적 공격으로 해당 시스템의 자원을 부족하게 하여 원래 의도된 용도로 사용하지 못하게 하는 공격을 의미한다.
여기에서는 Haproxy를 사용하여 가장 단순한 패턴인 `단일 IP에서 유입되는 과도한 request` 를 방어하는 방법을 살펴보았다.

### 구성 요건

* Haproxy 의 `rate limit` 기능을 사용하여 단일 IP 에서 유입되는 과도한 request 를 막는다.

* 내부 IP 대역은 제외한다. - vpc 내의 서버들이나, 내부 연동 서버들은 제한 대상에서 제외한다.

* 특정 ULR 들은 제외한다. - static file 등을 위한 whitelist 로 관리되는 url 목록을 관리한다.


### Haproxy 설정하기

1. 우선 제외할 IP 대역과, URL 리스트 파일을 정의하자.

    ```
    # ACL "vpc-network" 선언
    acl vpc-network src 10.10.1.0/16


    # ACL "allow_url" 선언
    # whitelist url 목록 파일 지정
    acl accept_url url_beg -i -f /etc/haproxy/accept_url.list

    ```

2. accept_url.list 파일의 내용은 다음과 같다.

    ```
    # Allowd url

    /public
    /assets
    /resource
    /feed
    ```

3. 이제 client ip 를 저장할 공간(sticky session)을 정의하자.

    ```
    # stickiness (Sticky Sessions)을 사용하여 10m 메모리에 10초동안 요청되는 client ip 를 저장한다. (1분 유지)
    # 이를 통한 ACL 제어는 다음의 컨트롤이 가능하다
    # - conn_rate(10s) : 10초 동안 TCP 커넥션의 rate
    # - conn_cur : 동시 접속 TCP 커넥션
    # - http_req_rate(10s) : 10초 동안의 HTTP 커넥션 rate
    # - http_err_rate(20s) : 20초 동안의 HTTP 커넥션 에러(비정상적인 커넥션 요청)
    # 여기서는 http_req_rate를 사용했다.
    stick-table type ip size 500m expire 1m store http_req_rate(10s)
    ```

4. 정의한 stick table의 카운트를 올리려면 다음과 같이 지정하자

    ```
    # 카운트 table 유지 vpc network 접속이 아니고 AND 허용되는 url 가 아닐 때 ( true AND true 조건 형태)
    http-request track-sc1 src table www if !vpc-network !accept_url
    ```

5. ACL 'dos-attack' 정의

    ```
    # 위에서 정의한 stick-table 에 따라서 10초 동안 요청되는 HTTP connectoin rate 가 100이 넘으면 DOS 공격으로 판단.
    acl dos-attack src_http_req_rate(www) ge 100
    ```

6. DOS 공격으로 판단될 때 사용할 backend 연결

    ```
    use_backend blocked_page if dos-attack !accept_url
    ```

7. blocked_page backend 정의

    ```
    backend blocked_page

        mode http
        # 벡엔드에서 연결될 서버가 없으므로 503 에러가 발생할 것이다. 이 때 보여줄 페이지는 아래와 같다.
        errorfile 503 /etc/haproxy/errors/blocked.http
    ```

8. block 되었을 때 보여줄 에러 페이지

    status code 429는 too many connection 오류를 의미한다.

    ```
    HTTP/1.1 429 Too Many Requests
    Content-Type: text/html
    Retry-After: 600

    <!doctype html>
    <html lang="ko">
    <head>
    <title>Can not connect page</title>
    <body>
    <p>Nothing</p>
    </body>
    </html>
    ```


### 완성된 Haproxy conf

위의 과정을 거치면 대략 아래와 같은 내용의 파일을 구성할 수 있다.

```
frontend www


    ...

    # ACL "vpc-network" 선언
    acl vpc-network src 10.10.1.0/16

    # ACL "allow_url" 선언
    # whitelist url 목록 파일 지정
    acl accept_url url_beg -i -f /etc/haproxy/accept_url.list


    # stickiness (Sticky Sessions)을 사용하여 10m 메모리에 10초동안 요청되는 client ip 를 저장한다. (1분 유지)
    # 이를 통한 ACL 제어는 다음의 컨트롤이 가능하다
    # - conn_rate(10s) : 10초 동안 TCP 커넥션의 rate
    # - conn_cur : 동시 접속 TCP 커넥션
    # - http_req_rate(10s) : 10초 동안의 HTTP 커넥션 rate
    # - http_err_rate(20s) : 20초 동안의 HTTP 커넥션 에러(비정상적인 커넥션 요청)
    # 여기서는 http_req_rate를 사용했다.
    stick-table type ip size 500m expire 1m store http_req_rate(10s)

    # 위에서 정의한 stick-table 에 따라서 10초 동안 요청되는 HTTP connectoin rate 가 100이 넘으면 DOS 공격으로 판단.
    acl dos-attack src_http_req_rate(www) ge 100


    # 카운트 table 유지 vpc network 접속이 아니고 AND 허용되는 url 가 아닐 때 ( true AND true 조건 형태)
    http-request track-sc1 src table www if !vpc-network !accept_url

    use_backend blocked_page if dos-attack !accept_url

    default_backend web-svr

backend blocked_page

    mode http
    # 벡엔드에서 연결될 서버가 없으므로 503 에러가 발생할 것이다. 이 때 보여줄 페이지는 아래와 같다.
    errorfile 503 /etc/haproxy/errors/blocked.http


backend web-svr

    ......

```

### 테스트

apache bench 를 사용해서 테스트 해보았다.

```
$ ab -r -c 10 -n 2000 -l http://my.service.host/
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking my.service.host (be patient)
Completed 200 requests
Completed 400 requests
Completed 600 requests
Completed 800 requests
Completed 1000 requests
Completed 1200 requests
Completed 1400 requests
Completed 1600 requests
Completed 1800 requests
Completed 2000 requests
Finished 2000 requests


Server Software:        Apache
Server Hostname:        my.service.host
Server Port:            80

Document Path:          /
Document Length:        Variable

Concurrency Level:      10
Time taken for tests:   2.991 seconds
Complete requests:      2000
Failed requests:        0
Non-2xx responses:      1901
Total transferred:      24512750 bytes
HTML transferred:       24332595 bytes
Requests per second:    668.73 [#/sec] (mean)
Time per request:       14.954 [ms] (mean)
Time per request:       1.495 [ms] (mean, across all concurrent requests)
Transfer rate:          8004.14 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        2    5   4.2      5      71
Processing:     6   10   7.3      8      73
Waiting:        5    9   5.5      7      73
Total:          9   15   8.4     13      80

Percentage of the requests served within a certain time (ms)
  50%     13
  66%     13
  75%     14
  80%     14
  90%     16
  95%     40
  98%     45
  99%     48
 100%     80 (longest request)
```

`Non-2xx responses` 라고 표시된 부분이 haproxy 에서 block 되어 429 response 응답을 수신한 부분이고, 로드 밸런서로 지정된 web-srv 서버들에도
access log 가 남지 않았다.


### rsyslog 로깅 변경

Haproxy 는 rsyslog 로 로그를 남기는데 여기서 dos attack 인 녀석들만 로그를 기록해 보자. `/etc/rsyslog.d/haproxy-dos.conf` 파일을 구성한다.
사전에 haproxy-info.log 가 존재하고, 로그에 dos 용 backend 인 'blocked_page' 라는 "backend name" 을 로깅하도록 커스텀 포맷이 지정되어 있다고 가정한다.

```
# Provides UDP syslog reception
$ModLoad imfile
$template Haproxy, "%msg%\n"
$InputFilePollInterval 1

# UDP 로 메세지를 수신하지 않고, haproxy-info 파일을 watch 하는 형태로 입력받음
# 기존에 이미 haproxy-info.log 가 있다고 가정.

$InputFileName /var/log/haproxy/haproxy-info.log
$InputFileTag haproxy-dos:
$InputFileStateFile haproxy-dos
$InputFileFacility local0
$InputRunFileMonitor

# blocked_page 라는 단어가 포함된 메세지를 haproxy-dos.log 에 다시 기록함
:syslogtag, isequal, "haproxy-dos:" {
  :msg, contains, "blocked_page" {
    local0.* /var/log/haproxy/haproxy-dos.log;Haproxy
  }
  stop
}
```

### 적용 후기

* 기존에 [nginx rate limit](/2018/06/24/nginx-rate-limiting)으로도 동일한 구성을 해보았는데, Haproxy 는 좀 더 유연하고, 더 디테일하게 제어가 가능했다.
여기서는 가장 기본적인 "단일 IP에서의 과도한 유입"만 막을 수 있었지만, 더 연구해서 다른 패턴도 막을 수 있도록 알아봐야겠다.(.. 언제?)

### 참고
  - https://blog.codecentric.de/en/2014/12/haproxy-http-header-rate-limiting/
  - https://gist.github.com/jeremyj/e964a951634f1997daea
  - https://www.loadbalancer.org/blog/simple-denial-of-service-dos-attack-mitigation-using-haproxy-2/
  - https://www.haproxy.com/blog/use-a-load-balancer-as-a-first-row-of-defense-against-ddos/
