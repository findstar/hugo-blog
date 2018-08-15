---
date: '2018-08-15 21:46:18 +09:00'
group: blog
image: /images/posts/haproxy/haproxy-logo.png
tags:
- haproxy
- "seamless reload"
- "zero downtime"
- "haproxy 1.8"
title: Haproxy를 다운타임 없이 재시작(reload)하기
url: /2018/08/15/seamless-reload-haproxy
type: post
---

앞서 [Haproxy를 reload fail 해결하기](/2018/08/14/fix-haproxy-reload-fail)을 살펴보았다. 이 과정에서 Haproxy가 실제로 reload 될 때 seamless 하게 작동할 수 있는가에 대한 확인을 해보았다.

<!--more-->

### Seamless reload?

Haproxy 를 구성한 뒤에 서비스를 재시작(restart) 하는 것이 아니라, 다시 로드(reload) 하는 순간이 있다. 단순히 설정파일의 일부 값을 변경하고 이를 다시 읽어 들이거나,
연결한 인증서 파일을 갱신/추가/삭제 하고 다시 로드하기 위함 뿐만 아니라, 최근의 도커 기반의 오케스트레이션 시스템을 사용한다면, 서비스는 더 빈번하게 끊김없이 다시 로딩되어야할 필요가 있는 것이다.
이를 `seamless reload` 라고 하고 (다르게는 `gracefully restart` 라고도 하는 것 같다) Haproxy 에서 적용한 방법이다.

### 리서치

* 이 주제는 논의가 제법 이전부터 이루어 져왔었고 [메일링 리스트 참조](https://www.mail-archive.com/haproxy@formilux.org/), 작년(2017.05)에
[공식사이트의 블로그](https://www.haproxy.com/blog/truly-seamless-reloads-with-haproxy-no-more-hacks/)에서 소개되면서 정리가 되었다.


### Haproxy 의 재기동 절차

공식 블로그에 따르면 Haproxy 가 다시 시작하기 위해서는 다음의 과정을 거친다고 한다.

1. 새로운 프로세스가 필요한 전체 포트(port)를 수신할 수 있는지(listen) 확인한다.
2. 만약 그렇지 않다면 이전 프로세스(old process)에게 임시적으로(temporarily) 포트 수신은 해제하라는 신호를 보낸다.
3. 새로운 프로세스는 계속해서 다시 확인한다.
4. 포트 수신이 가능한지 확인하는 과정이 계속된다면 일정 횟수 이후에는 새로운 프로세스가 구동하기를 포기하고(give up) 이전 프로세스(old process)에게 유입되는 커넥션을 계속 담당하라고 신호를 보낸다.
5. 만약 새로운 프로세스가 필요한 전체 포트를 수신할 수 있게 되었다면 이전 프로세스(old process)에게 현재의 커넥션에 대한 처리가 종료되고 나서 프로세스를 종료하라고 신호를 보낸다.

여기서 문제가 되는 부분은 2번과 3번 사이인데, 실제로 포트가 해제된 후에(release) 신규 프로세스가 수신대기(listen)하기 까지 아주 약간의 시간이 필요하며 이 과정에서
유입되는 커넥션에 대한 처리가 불가능해진다는 점이다.

블로그에서는 이 문제를 수정하기 위해서 어떠한 변경사항이 있었는지 [리눅스 커널 패치](https://lkml.org/lkml/2008/2/28/360) 이야기 까지 하면서 설명해 놓았다.
하나하나 읽어 보는 것도 재미나지만, 간략하게 정리하면 다음과 같다.

### Haproxy 의 Seamless reload 를 위한 기술적 내용

1. 이전 프로세스에서 file descriptor 를 기반으로한 socket 을 통해서 현재 관리되고 있는 커넥션을 새로운 프로세스로 전달한다.
2. 이과정에서 file socket(unix socket) 에서 확인되는 커넥션은 끊기지 않는다.
3. 프로세스는 master-worker 로 동작하면서 이 작업을 수행한다.

요약하자면 unix socket을 사용해서 커넥션 상태를 유지하고, 이를 이전 프로세스와 새로운 프로세스간에 전달함으로써, 커넥션의 유실을 막는다는 것이다.

참고로 haproxy 의 동작모드는 다음과 같고 Systemd 를 사용해서 reload 를 하려면 `-Ws` 모드를 사용해야 한다. (또한 빌드할 때 `USE_SYSTEMD` 가 활성화 되어 있어야 한다)

```
  -D : start as a daemon. The process detaches from the current terminal after
    forking, and errors are not reported anymore in the terminal. It is
    equivalent to the "daemon" keyword in the "global" section of the
    configuration. It is recommended to always force it in any init script so
    that a faulty configuration doesn't prevent the system from booting.

  -W : master-worker mode. It is equivalent to the "master-worker" keyword in
    the "global" section of the configuration. This mode will launch a "master"
    which will monitor the "workers". Using this mode, you can reload HAProxy
    directly by sending a SIGUSR2 signal to the master.  The master-worker mode
    is compatible either with the foreground or daemon mode.  It is
    recommended to use this mode with multiprocess and systemd.

  -Ws : master-worker mode with support of `notify` type of systemd service.
    This option is only available when HAProxy was built with `USE_SYSTEMD`
    build option enabled.
```

### 설정 파일의 변경

이 기능은 Haproxy 1.8 버전에서 정식으로 적용되었고, 이를 사용하기 위해서는 `expose-fd listeners` 설정을 추가해주어야 한다.
이 옵션의 의미하는 것은 file descriptor 를 통해서 connection listener 하는 것을 노출(expose)하겠다라는 뜻이다.

```
# seamless reload 를 위한 status socket 운영
stats socket /var/lock/subsys/haproxy mode 777 level admin expose-fd listeners
```

이제 `sudo systemd reload haproxy` 를 실행하면 정상적으로 reload 가 수행된다.

### 테스트

그렇다면 이렇게 `seamless reload` 를 구성했으니, 정말 커넥션 손실이 없는가 결과를 확인해 보아야 한다. ab(apache bench) 툴을 사용해서 확인해보았다.

1. 먼저 정상적으로 서비스가 동작하고 있는 상태에서 아무런 액션을 취하지 않았을 때

    ```
    $ ab -r -c 100 -n 100000 -l http://haproxy-lb.host/
    ab: illegal option -- l
    This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking haproxy-lb.host (be patient)
    Completed 10000 requests
    Completed 20000 requests
    Completed 30000 requests
    Completed 40000 requests
    Completed 50000 requests
    Completed 60000 requests
    Completed 70000 requests
    Completed 80000 requests
    Completed 90000 requests
    Completed 100000 requests
    Finished 100000 requests


    Server Software:        Apache
    Server Hostname:        haproxy-lb.host
    Server Port:            80

    Document Path:          /
    Document Length:        32987 bytes

    Concurrency Level:      100
    Time taken for tests:   35.472 seconds
    Complete requests:      100000
    Failed requests:        0
    Write errors:           0
    Total transferred:      3340500000 bytes
    HTML transferred:       3298700000 bytes
    Requests per second:    2819.15 [#/sec] (mean)
    Time per request:       35.472 [ms] (mean)
    Time per request:       0.355 [ms] (mean, across all concurrent requests)
    Transfer rate:          91966.57 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   0.9      1       6
    Processing:    22   34  11.2     34    1052
    Waiting:       19   33  11.1     32    1051
    Total:         22   35  11.2     35    1053

    Percentage of the requests served within a certain time (ms)
      50%     35
      66%     37
      75%     39
      80%     39
      90%     42
      95%     44
      98%     47
      99%     50
     100%   1053 (longest request)
    ```

2. `systemd restart haproxy.service` 실행한 경우

    ```
    $ ab -r -c 100 -n 100000 -l http://haproxy-lb.host/
    ab: illegal option -- l
    This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking haproxy-lb.host (be patient)
    Completed 10000 requests
    Completed 20000 requests
    Completed 30000 requests
    Completed 40000 requests
    Completed 50000 requests
    Completed 60000 requests
    Completed 70000 requests
    Completed 80000 requests
    Completed 90000 requests
    Completed 100000 requests
    Finished 100000 requests


    Server Software:        Apache
    Server Hostname:        haproxy-lb.host
    Server Port:            80

    Document Path:          /
    Document Length:        32987 bytes

    Concurrency Level:      100
    Time taken for tests:   36.536 seconds
    Complete requests:      100000
    Failed requests:        104
       (Connect: 0, Receive: 2, Length: 100, Exceptions: 2)
    Write errors:           0
    Total transferred:      3337159500 bytes
    HTML transferred:       3295401300 bytes
    Requests per second:    2737.02 [#/sec] (mean)
    Time per request:       36.536 [ms] (mean)
    Time per request:       0.365 [ms] (mean, across all concurrent requests)
    Transfer rate:          89198.02 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    2  31.7      1    1004
    Processing:     3   35   6.9     34     214
    Waiting:        0   33   6.8     33     213
    Total:          3   36  33.2     35    1078

    Percentage of the requests served within a certain time (ms)
      50%     35
      66%     37
      75%     38
      80%     39
      90%     41
      95%     44
      98%     47
      99%     50
     100%   1078 (longest request)
    ```

3. `systemctl reload haproxy` : socket 설정을 추가하기 전 reload

    ```
    $ ab -r -c 100 -n 100000 -l http://haproxy-lb.host/
    ab: illegal option -- l
    This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking haproxy-lb.host (be patient)
    Completed 10000 requests
    Completed 20000 requests
    Completed 30000 requests
    Completed 40000 requests
    Completed 50000 requests
    Completed 60000 requests
    Completed 70000 requests
    Completed 80000 requests
    Completed 90000 requests
    Completed 100000 requests
    Finished 100000 requests


    Server Software:        Apache
    Server Hostname:        haproxy-lb.host
    Server Port:            80

    Document Path:          /
    Document Length:        32987 bytes

    Concurrency Level:      100
    Time taken for tests:   35.514 seconds
    Complete requests:      100000
    Failed requests:        3
       (Connect: 0, Receive: 1, Length: 1, Exceptions: 1)
    Write errors:           0
    Total transferred:      3340466595 bytes
    HTML transferred:       3298667013 bytes
    Requests per second:    2815.79 [#/sec] (mean)
    Time per request:       35.514 [ms] (mean)
    Time per request:       0.355 [ms] (mean, across all concurrent requests)
    Transfer rate:          91855.95 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   0.9      1       8
    Processing:     0   35   7.1     34     239
    Waiting:        0   33   6.9     32     238
    Total:          0   35   7.0     35     239

    Percentage of the requests served within a certain time (ms)
      50%     35
      66%     37
      75%     38
      80%     39
      90%     42
      95%     44
      98%     47
      99%     50
     100%    239 (longest request)
    ```

4. `systemctl reload haproxy` : `socket expose-fd listeners` 설정을 추가한 뒤의 reload

    ```
    $ ab -r -c 100 -n 100000 -l http://haproxy-lb.host/
    ab: illegal option -- l
    This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking haproxy-lb.host (be patient)
    Completed 10000 requests
    Completed 20000 requests
    Completed 30000 requests
    Completed 40000 requests
    Completed 50000 requests
    Completed 60000 requests
    Completed 70000 requests
    Completed 80000 requests
    Completed 90000 requests
    Completed 100000 requests
    Finished 100000 requests


    Server Software:        Apache
    Server Hostname:        haproxy-lb.host
    Server Port:            80

    Document Path:          /
    Document Length:        32987 bytes

    Concurrency Level:      100
    Time taken for tests:   36.077 seconds
    Complete requests:      100000
    Failed requests:        0
    Write errors:           0
    Total transferred:      3340500000 bytes
    HTML transferred:       3298700000 bytes
    Requests per second:    2771.86 [#/sec] (mean)
    Time per request:       36.077 [ms] (mean)
    Time per request:       0.361 [ms] (mean, across all concurrent requests)
    Transfer rate:          90423.65 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   0.9      1       8
    Processing:    22   35   6.9     34     223
    Waiting:       20   34   6.8     33     222
    Total:         22   36   6.9     35     224

    Percentage of the requests served within a certain time (ms)
      50%     35
      66%     37
      75%     39
      80%     40
      90%     43
      95%     46
      98%     49
      99%     52
     100%    224 (longest request)
    ```

5. 요약 (동시 100개 10만 커넥션)

    Test case| Time taken for tests | Failed connection
    --------|------|------
    nomal  | 35.472s|0
    restart | 36.536s|104
    reload without socket | 35.514s|3
    reload with socket | 36.077s|0


### 이슈 해결 소감.

Haproxy 는 초기 설계에서 잦은 재시작을 고려하지 않았었다. 그러나 Haproxy를 사용하는 환경이 변화하면서 `seamless reload` 에 대한 사용자의 필요가 더 많아졌고,
결국 1.8에서 완전한 해결책을 제시하고 있다. 나의 경우에는 처음에 접한 버전이 1.8 버전대 였지만, 공식 매뉴얼을 찾기 보다 레퍼런스를 구해서 설치하다가, 시행착오를 겪은 케이스이다.

마지막에 테스트까지 완료하면서 이상없이 재시작되는 것을 보니 마음이 참 뿌듯했다. 또한 개발팀에서 이 하나의 이슈를 완전히 해결하기까지, 무려 10여년동안 내부 아키텍처를 개선해오면서 노력했다는 점이 가장 인상깊다.
나도 어떤 문제를 끝까지 해결하기 위해서 노력하는 개발자가 되어야겠다는 마음가짐을 다시금 새겨본다.


### 참고

- https://www.haproxy.com/blog/truly-seamless-reloads-with-haproxy-no-more-hacks/
- http://git.haproxy.org/?p=haproxy-1.8.git;a=blob_plain;f=contrib/systemd/haproxy.service.in
