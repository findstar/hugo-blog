---
date: '2018-08-14 23:17:17 +09:00'
group: blog
image: /images/posts/haproxy/haproxy-logo.png
tags:
- haproxy
- "haproxy reload fail"
- "systemd reload"
- "haproxy 1.8"
title: Haproxy를 reload fail 해결하기
url: /2018/08/14/fix-haproxy-reload-fail
type: post
---

Haproxy를 처음 설치했을 때 로그를 위해서 rsyslog 와 logrotate 를 함께 설정했었다. 그런데 새벽만 되면 (정확히는 logrotate가 실행되고 나서)
Haproxy 프로세스가 죽은 다음에 살아나지를 않았다. 이를 해결하기 위해서 수행했던 몇일간의 삽질(!)아닌 삽질기를 정리해보았다.

<!--more-->

### 문제의 발단.

처음에는 분명히 서비스가 동작하고 있었던것 같은데 다음날 작업해 보려고 하면 서비스가 죽어 있었다. 내가 어제 마지막에 프로세스를 내려 놓았었나? 하고, conf 파일 구성에 신경쓰느라 인지를 못했던 것이었다.
원하는 haproxy 설정을 마무리 하고, 실제 production 에 적용하기 전에, log 파일 및 logrotate 설정을 살펴보고 있었는데,
다음날만 되면 어김없이, haproxy 프로세스가 죽은 상태로 다시 살아나지를 않았다.

### 원인 찾기

무엇이 문제일까 고민하다가 일단 journalctl 을 뒤져보기로 했다.

```
# journalctl -xe
```

그랬더니 아래와 같은 메세지를 확인할 수 있었다.

```
 7월 24 03:41:01 haproxy-lb CROND[18789]: (root) CMD (/usr/lib64/sa/sa1 1 1)
 7월 24 03:41:03 haproxy-lb sshd[19005]: Connection closed by 10.41.20.81 port 45638 [preauth]
 7월 24 03:41:55 haproxy-lb sshd[19231]: Connection closed by 10.41.20.81 port 48616 [preauth]
 7월 24 03:42:01 haproxy-lb CROND[19235]: (root) CMD (/usr/lib64/sa/sa1 1 1)
 7월 24 03:42:01 haproxy-lb CROND[19236]: (root) CMD (/opt/SE/update_k5login)
 7월 24 03:42:01 haproxy-lb anacron[1389]: Job `cron.daily' started
 7월 24 03:42:01 haproxy-lb run-parts(/etc/cron.daily)[19244]: starting logrotate
 7월 24 03:42:01 haproxy-lb haproxy[19255]: Shutting down haproxy: [FAILED]
 7월 24 03:42:01 haproxy-lb logrotate[19272]: ALERT exited abnormally with [1]
 7월 24 03:42:01 haproxy-lb run-parts(/etc/cron.daily)[19274]: finished logrotate
 7월 24 03:42:01 haproxy-lb run-parts(/etc/cron.daily)[19276]: starting man-db.cron
 7월 24 03:42:01 haproxy-lb run-parts(/etc/cron.daily)[19285]: finished man-db.cron
 7월 24 03:42:01 haproxy-lb run-parts(/etc/cron.daily)[19287]: starting mlocate
 7월 24 03:42:02 haproxy-lb run-parts(/etc/cron.daily)[19296]: finished mlocate
 7월 24 03:42:02 haproxy-lb anacron[1389]: Job `cron.daily' terminated (produced output)
```

새벽에 cron.daily 작업에 걸려 있던 logrotate 작업이 비정상적으로 종료된것을 확인했다. `ALERT exited abnormally` 라는 문구가 바로 그것이다.
`sudo logrotate -f /etc/logrotate.d/haproxy` 와 같이 테스트해본 결과 증상이 재현되었다. 구동중인 haproxy 가 재시작 되지 않고 죽어버렸다.

### logrotate 의심하기

무엇이 문제일까? logrotate 에 설정되어 있는 haproxy `/etc/logrotate.d/haproxy`을 살펴보았다.

```
/var/log/haproxy/*log {
    create 0644 root root
    daily
    rotate 90
    missingok
    notifempty
    compress
    sharedscripts
    postrotate
        /bin/kill -USR1 `cat /run/haproxy.pid 2>/dev/null` 2>/dev/null || true
        chown -R root:root /var/log/haproxy/*.log 2>/dev/null
    endscript
}
```

처음 구성되어 있던 logrotate 설정이다. kill -USR1 으로 실행하고 있었는데 여기서 다시 살아나지 않은것이 문제였다.
kill 명령어가 잘못되었을리는 없고, 혹시 몰라 postrotate 에 연결된 task를 다음처럼 바꿔봤지만 여전히 문제였다.

```
/bin/systemctl reload haproxy.service > /dev/null 2>/dev/null || true
```

### 의문사항

이 시점에서 두가지 의문점을 떠올렸는데

1. haproxy 는 rsyslog 로 (UDP) 로그를 전달하고 이를 저장하는 녀석은 rsyslog 이므로,

logrotate 작업은 haproxy 가 아니라, rsyslog 가 재시작되어야 하는게 아닌가? 라는 것.


2. 그건 그렇고 왜 haproxy 는 재시작이 되지 않는가?

일단 haproxy 가 reload 되지 않는 원인부터 찾아봤다


### Haproxy 설정 살펴보기

사용하는 haproxy 버전은 1.8.12 버전이다.

```
# haproxy -vv
HA-Proxy version 1.8.12-8a200c7 2018/06/27
Copyright 2000-2018 Willy Tarreau <willy@haproxy.org>

Build options :
  TARGET  = linux2628
  CPU     = generic
  CC      = gcc
  CFLAGS  = -O2 -g -fno-strict-aliasing -Wdeclaration-after-statement -fwrapv -fno-strict-overflow -Wno-unused-label
  OPTIONS = USE_ZLIB=1 USE_OPENSSL=1 USE_PCRE=1

Default settings :
  maxconn = 2000, bufsize = 16384, maxrewrite = 1024, maxpollevents = 200

Built with OpenSSL version : OpenSSL 1.0.2k-fips  26 Jan 2017
Running on OpenSSL version : OpenSSL 1.0.2k-fips  26 Jan 2017
OpenSSL library supports TLS extensions : yes
OpenSSL library supports SNI : yes
OpenSSL library supports : SSLv3 TLSv1.0 TLSv1.1 TLSv1.2
Built with transparent proxy support using: IP_TRANSPARENT IPV6_TRANSPARENT IP_FREEBIND
Encrypted password support via crypt(3): yes
Built with multi-threading support.
Built with PCRE version : 8.32 2012-11-30
Running on PCRE version : 8.32 2012-11-30
PCRE library supports JIT : no (USE_PCRE_JIT not set)
Built with zlib version : 1.2.7
Running on zlib version : 1.2.7
Compression algorithms supported : identity("identity"), deflate("deflate"), raw-deflate("deflate"), gzip("gzip")
Built with network namespace support.

Available polling systems :
      epoll : pref=300,  test result OK
       poll : pref=200,  test result OK
     select : pref=150,  test result OK
Total: 3 (3 usable), will use epoll.

Available filters :
        [SPOE] spoe
        [COMP] compression
        [TRACE] trace
```

다시한번 `sudo systemctl reload haproxy` 를 입력해서 증상을 재현하고

`sudo journalctl --unit=haproxy` 로 확인해보면 아래와 같이 실패한다.

```
7월 26 15:20:13 haproxy-lb systemd[1]: haproxy.service: main process exited, code=killed, status=9/KILL
7월 26 15:20:13 haproxy-lb haproxy[35552]: Shutting down haproxy: [FAILED]
```

구글링을 시도해보았다. "haproxy reload not working" 이라는 키워드에서 뭔가가 여러 결과가 나타났다. 그러던중 메일링 리스트에서 관련 이슈를 찾을 수 있다.
https://www.mail-archive.com/haproxy@formilux.org/

제법 오래된 이슈인듯 한데, 이러한 경우를 "seamless reloads" 라고 지칭한다는 것을 알아내고 계속 리서칭해봤다. https://discourse.haproxy.org/t/seamless-reloads-dont-work-with-systemd/1954/16 에서의 사례가 참고가 많이 되었다.
그리고 [공식사이트의 블로그](https://www.haproxy.com/blog/truly-seamless-reloads-with-haproxy-no-more-hacks/)를 확인하고 나서 문제점을 파악할 수 있었다.

### 문제점

1. haproxy 를 seamless reload 하려면 haproxy -Ws 모드가 활성화된 master-worker 모드로 동작하도록 해야한다.

  - 이를 위해서 빌드 할 때 'systemd' 를 지원하도록 빌드되어야 한다.
  - 빌드 옵션은 `haproxy -vv` 에서 `USE_SYSTEMD` 가 활성화 되어 있는 것을 확인하면 된다.

2. haproxy 공식 git repo 에서 `systemd script` 를 사용하지 않고 구식의 init 스크립트를 사용했다는 점 (어느 설치 블로그에서 복사해옴)

### 해결방법

1. Haproxy 에서 systemd 를 사용하여 reload 를 하기 위해서는 make 를 진행할 때 `USE_SYSTEMD` 옵션을 활성화 해서 빌드하자

  - 나의 경우 `make TARGET=linux2628 USE_OPENSSL=1 USE_PCRE=1 USE_ZLIB=1 USE_SYSTEMD=1` 사용함.

2. [공식 git repo 의 systemd script](http://git.haproxy.org/?p=haproxy-1.8.git;a=blob_plain;f=contrib/systemd/haproxy.service.in) 를 사용하여 서비스 등록

  - @SBINDIR@ 은 "/usr/local/sbin/" 으로 대체해주었다.

  ```
  [Unit]
  Description=HAProxy Load Balancer
  After=network.target

  [Service]
  Environment="CONFIG=/etc/haproxy/haproxy.cfg" "PIDFILE=/run/haproxy.pid"
  ExecStartPre=@SBINDIR@/haproxy -f $CONFIG -c -q
  ExecStart=@SBINDIR@/haproxy -Ws -f $CONFIG -p $PIDFILE
  ExecReload=@SBINDIR@/haproxy -f $CONFIG -c -q
  ExecReload=/bin/kill -USR2 $MAINPID
  KillMode=mixed
  Restart=always
  SuccessExitStatus=143
  Type=notify

  # The following lines leverage SystemD's sandboxing options to provide
  # defense in depth protection at the expense of restricting some flexibility
  # in your setup (e.g. placement of your configuration files) or possibly
  # reduced performance. See systemd.service(5) and systemd.exec(5) for further
  # information.

  # NoNewPrivileges=true
  # ProtectHome=true
  # If you want to use 'ProtectSystem=strict' you should whitelist the PIDFILE,
  # any state files and any other files written using 'ReadWritePaths' or
  # 'RuntimeDirectory'.
  # ProtectSystem=true
  # ProtectKernelTunables=true
  # ProtectKernelModules=true
  # ProtectControlGroups=true
  # If your SystemD version supports them, you can add: @reboot, @swap, @sync
  # SystemCallFilter=~@cpu-emulation @keyring @module @obsolete @raw-io

  [Install]
  WantedBy=multi-user.target
  ```


### 추가 작업

* logrotate 에서 haproxy 를 재시작하지 않고, rsyslog 를 재시작해주도록 수정해주었다.

    ```
    /var/log/haproxy/*log {
        daily
        rotate 90
        create 0644 nobody nobody
        missingok
        notifempty
        compress
        sharedscripts
        postrotate
            /bin/systemctl restart rsyslog.service > /dev/null 2>/dev/null || true
        endscript
    }
    ```


### 정리


1. Haproxy 는 load balancer 로 사용하기 위해서 가급적이면 재시작하는 것을 지양하는 형태이다.
 - 이 시점에서 logrotate 를 위해서 haproxy 를 재시작하는 것이 얼마나 엉뚱한 행동이었는지 알게되었다.
 - haproxy 는 log 를 rsyslog 로 UDP 형태로 전송하기 때문에, 로그를 남기는 것은 rsyslog 이고,
 - 실제로 logroate 에서는 rsyslog 를 재시작해주는게 맞다.

2. haproxy 는 다음의 3가지 모드로 구동이 가능하다. 이중에서 내가 구동하는 모드는 `-D` 옵션으로 데몬 형태로 구동하고 있었는데 systemd 를 사용하여
  reload 하기 위해서는 `-W` 모드 또는 `-Ws` 모드가 필요하다. 나의 경우에는 `-Ws` 모드를 사용하도록 바꾸기로 했다. 그리고 이를 위해서 `USE_SYSTEMD` 옵션을 활성화 할 필요가 있다.


### 남은 이야기

작업 이후 `systemd reload haproxy` 명령어를 통해서 haproxy 가 정상적으로 다시 리로드 된다. 이로써, configuration을 바꾸었거나, 인증서를 교체하는 작업등을 수행할 때
restart 하지 않고서도 graceful 하게 재시작 할 수 있게 되었다. 그런데 여기서 의문이 하나 추가되었는데 reload 될때 seamless, 그러니까 커넥션의 유실없이 정상적으로 haproxy 가 다시 로딩되는지 문제가 없는지, 확인해보고 싶었다.
그래서 실제로 테스트를 수행하면서 보완적업을 하게 되었는데, 이건 [별도 포스트](/2018/08/15/seamless-reload-haproxy)로 정리해보았다.


### 참고

- 메일링 리스트에서 관련 이슈를 찾을 수 있다. 제법 오래된 이슈인듯. https://www.mail-archive.com/haproxy@formilux.org/
- 이 이슈를 해결하면서 가장 많은 참고가 되었던 discussion https://discourse.haproxy.org/t/seamless-reloads-dont-work-with-systemd/1954/16
- haproxy 공식 사이트에서 해결방법을 안내하고 있다. https://www.haproxy.com/blog/truly-seamless-reloads-with-haproxy-no-more-hacks/
