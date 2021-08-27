---
date: '2018-07-27 08:08:32 +09:00'
group: blog
image: /images/posts/haproxy/haproxy-logo.png
tags:
- haproxy
- install
- "haproxy 1.8"
title: Haproxy 설치해서 로드 밸런서로 활용하기
url: /2018/07/27/install-haproxy
type: post
---

Load Balancer 로 활용할 수 있는 Reverse Proxy 인 Haproxy 의 설치와 설정 방법을 정리해보았다.

<!--more-->

Haproxy 는 Load balancer 로 활용할 수 있으며, 다양한 설정이 가능하고, nginx reverse proxy 에 비해서 active health check가 가능하기 때문에
더 안정적으로 운영할 수 있다. 다만 설정이 조금 복잡해서 익숙해 지는데 시간이 걸렸다.

### 설치 환경

1. haproxy 를 설치한 서버는 Centos 7.4 서버였다. yum 을 사용할것인지 아니면 컴파일을 해서 사용할지 결정해야 했는데 결국은 컴파일해서 설치하기로 했다. 그 이유는 yum 기본 repo 에 있는 haproxy 버전이 낮고, 대부분의 레퍼런스에서 컴파일해서 설치하고 있기에 이대로 진행했다.
 (다만, 컴파일에서 옵션을 잘못 지정해서 나중에 엄청난 삽질을 하게되었는데, 이건 따로 이야기 하기로 하고 설치 내역을 정리해보았다)

2. 필요한 yum 패키지들을 설치해주자.

    ```
    yum install gcc openssl openssl-devel pcre-static pcre-devel systemd-devel
    ```

### Haproxy 다운로드 & 컴파일

1. haproxy 소스 파일을 다운로드 받는다. 설치하는 현재 stable 버전은 1.8.12 버전이다.

    ```
    wget http://www.haproxy.org/download/1.8/src/haproxy-1.8.12.tar.gz
    tar zxvf haproxy-1.8.12.tar.gz
    ```

2. 컴파일을 하자, 이때 옵션을 지정해줘야 한다. TARGET 정보는 `README` 파일에서 확인하자. SSL 인증서를 활성화 하기 위해서 `USE_OPENSSL` 를 enable 해주고, systemd 를 통해서 서비스를 컨트롤 하기 위해서 `USE_SYSTEMD` 도 활성화 해주었다.

    ```
    make TARGET=linux2628 USE_OPENSSL=1 USE_PCRE=1 USE_ZLIB=1 USE_SYSTEMD=1
    ```

3. 컴파일을 완료했다면 설치하자 `/usr/local/sbin/haproxy` 에 실행파일이 설치된다.

    ```
    make install
    install -d "/usr/local/sbin"
    install haproxy  "/usr/local/sbin"
    install -d "/usr/local/share/man"/man1
    install -m 644 doc/haproxy.1 "/usr/local/share/man"/man1
    install -d "/usr/local/doc/haproxy"
    for x in configuration management architecture peers-v2.0 cookie-options lua WURFL-device-detection proxy-protocol linux-syn-cookies network-namespaces DeviceAtlas-device-detection 51Degrees-device-detection netscaler-client-ip-insertion-protocol peers close-options SPOE intro; do \
            install -m 644 doc/$x.txt "/usr/local/doc/haproxy" ; \
    done
    ```

4. 정상적으로 설치가 되었는지 확인하자

    ```
    sudo haproxy -v
    ```

### Haproxy systemd 등록

1. 이제 systemd 에 등록하고 서비스를 컨트롤할 수 있도록 해주어야 한다. [haproxy git repo 에서 제공하는 system file](http://git.haproxy.org/?p=haproxy-1.8.git;a=blob_plain;f=contrib/systemd/haproxy.service.in) 을 확인하자.
해당 파일을 `/etc/systemd/system/haproxy.service` 에 저장하고 `sudo systemd daemon-reload` 실행하자.

    ```
    sudo wget "http://git.haproxy.org/?p=haproxy-1.8.git;a=blob_plain;f=contrib/systemd/haproxy.service.in" -O /etc/systemd/system/haproxy.service
    sudo systemd daemon-reload
    ```

### 기타 작업

1. haproxy 관련 디렉토리를 만들자.

    ```
    sudo mkdir -p /etc/haproxy
    sudo mkdir -p /var/log/haproxy
    sudo mkdir -p /etc/haproxy/certs
    sudo mkdir -p /etc/haproxy/errors/
    ```

2. haproxy 데몬을 구동할 haproxy 계정을 만들자

    ```
    sudo useradd -r haproxy
    ```

### 설정 파일

1. example 디렉토리의 haproxy config 을 참고해서 원하는 형태의 cfg 를 구성해야 한다. 이게 제일 힘든 부분이긴 한데 고생한 결과물을 기록했다.

    ```
    # 다음은 80 과 443 으로 haproxy 를 reverse proxy 형태로 구성하여 LB로 사용하는 설정이다.
    global
        daemon

        # 연결할 수 있는 최대 connection 을 지정한다. 이걸 안하면 기본값이 2000 으로 설정된다.
        maxconn 81920

        # haproxy 프로세서를 구동할 user (gid/uid를 지정할 수도 있다.)
        user    haproxy

        # ssl 을 구성할 때 key size를 지정한다.
        tune.ssl.default-dh-param 2048

        # haproxy는 로그를 남기기 위해서 file io 를 직접 처리하지 않는다. rsyslog 로 UDP 전송을 한다.
        log 127.0.0.1 local0

        # ciphers 설정
        # WindowsXP SP3 IE7,8 호환이 필요하지 않을 경우 !3DES도 추가한다.
        ssl-default-bind-ciphers ECDH+AESGCM:ECDH+AES128:ECDH+AES256:DH+AES128:DH+AES256:DH+CAMELLIA128:DH+CAMELLIA256:DH+SEEDCBC:RSA:!aNULL:!MD5:!eNULL:!RC4

    defaults

        # Haproxy 에서 log format 은 총 5가지 (default, tcp, http, clf, custom) 을 지원한다
        # 1. 기본적인 log format
        # datetime host process_name[pid]: Connect from source_ip:source_port to dest_ip:dest_port (frontend_name/mode)
        # ex) Feb  6 12:12:09 localhost haproxy[14385]: Connect from 10.0.1.2:33312 to 10.0.3.31:8012 (www/HTTP)
        #
        # 2. `option tcplog` 가 지정된 경우
        # datetime host process_name[pid]: client_ip:client_port [accept_date] frontend_name backend_name/server_name tw/tc/tx bytes_read termination_state actionn/feconn/beconn/srv_conn/retries srv_queue/backend_queue
        # ex) Feb  6 12:12:56 localhost haproxy[14387]: 10.0.1.2:33313 [06/Feb/2009:12:12:51.443] fnt bck/srv1 0/0/5007 212 -- 0/0/0/0/3 0/0
        #
        # 3. `option httplog` 가 지정된 경우
        # HTTP 프록시를 구성하는 경우에 제일 많이 사용되는 로그 포맷
        # datetime host process_name[pid]: client_ip:client_port [request_date] frontend_name backend_name/server_name TR/Tw/Tc/Tr/Ta status_code bytes_read captured_request_cooie captured_response_cookie termination_state actionn/feconn/beconn/srv_conn/retries srv_queue/backend_queue {captured_request_headers} {captured_response_headers} "http_request"
        # ex)
        # Feb  6 12:14:14 localhost haproxy[14389]: 10.0.1.2:33317 [06/Feb/2009:12:14:14.655] http-in static/srv1 10/0/30/69/109 200 2750 - - ---- 1/1/1/1/0 0/0 {1wt.eu} {} "GET /index.html HTTP/1.1"
        #
        # 4. `log-format` 을 통해서 custom 구성을 한경우
        # `log-format` 을 지정하면 커스텀 로그 포맷 구성이 가능함
        # 자세한 변수는 http://cbonte.github.io/haproxy-dconv/1.8/configuration.html#8.2.4 참고
        #
        # 5. clf 는 (Common Logging Format)
        #
        # 이설정파일에서는 custom log format 을 사용했다.
        log       global
        mode      http
        option    httplog clf

        #Enable logging of null connections (헬스체크와 같이 시스템이 살아 있는지 확인하기 위해서 일정하게 접속하는 커넥션에 대한 로그 사용)
        option    dontlognull

        #Enable skip logging normal connection log 일반적인 http status 200 로그는 남기지 않는다.
        (https://www.slideshare.net/haproxytech/haproxy-best-practice) 참고
        option    dontlog-normal

        #request-요청을 서버로 보낼 때 `X-Forwarded-For` 를 헤더에 추가한다
        option    forwardfor

        # 기본적으로 HAProxy 는 커넥션 유지 관점에서 keep-alive 모드로 동작을 하는데, 각각의 커넥션은 request-요청과 reponse-응답을 처리하고나서
        # 새로운 request을 받기까지 connection idle 상태(유휴상태)로 양쪽이 연결되어 있다.
        # 이 동작모드를 변경하려면 "option http-server-close" "option forceclose" "option httpclose" "option http-tunnel" 의 옵션이 가능한데,
        # "option http-server-close" 는 클라이언트 사이드에서 HTTP keep-alive를 유지하고 파이프라이닝을 지원하면서 서버 사이드에 커넥션을 닫는 형태를 설정한다.
        # 이는 클라이언트 사이드에서 최저 수준의 응답지연을 제공하고 "option forceclose" 와 비슷하게 서버사이드에서 리소스를 재활용할 수 있게 되어
        # backend 에서 빠르게 세션을 재사용할 수 있도록 해준다.
        option    http-server-close


        timeout   http-request  10s
        timeout   client        20s
        timeout   connect       4s
        timeout   server        30s
        timeout   http-keep-alive   10s

    # haproxy 의 상태정보를 확인할 수 있는 기능을 활성화 한다. "stats"
    # 이 포트는 외부에 노출되지 않도록 주의하자.
    # 접근 가능한 사용자를 제한하기 위해서 auth 를 추가했다.
    listen stats
        bind :9000 # Listen on localhost:9000
        stats enable  # Enable stats page
        stats realm Haproxy\ Statistics  # Title text for popup window
        stats uri /haproxy_stats  # Stats URI
        stats auth    admin1:password1
        stats auth    admin2:password2


    frontend www

        # 기본적으로 80 번을 listen 한다.
        bind *:80

        # https 지원을 위한 443 listen
        # cert 파일은 /etc/haproxy/certs 디렉토리에 넣는다.
        # 키 파일과 cert 파일을 하나로 합쳐서 넣으면 된다.
        bind *:443 ssl crt /etc/haproxy/certs

        maxconn 81920

        # 압축 알고리즘 gzip 적용
        compression algo gzip
        compression type text/plain application/json application/xml

        # ACL "deny_useragent" 선언
        # 보안취약점 스캔 툴 같은 agent 들을 막기 위한 list 파일을 구성했다.
        # -i 옵션은 대소문자 구분하지 않음
        # -f 는 파일에서 매칭
        acl deny_useragent hdr_sub(user-agent) -i -f /etc/haproxy/deny_useragent.list

        # Private Ip 대역 확인용 ACL "private-network" 선언
        # 같은 VPC 대역이라던가..
        acl private-network src 10.10.0.0/16

        http-request deny if deny_useragent

        # 모니터링을 위한 주소 설정
        monitor-uri /monitor

        # 모니터링 주소는 같은 private ip 대역이 아니라면 실패 처리
        monitor fail  if !private-network

        # log format 을 위해서 header 정보를 capture 한다. custom format %hr 에서 사용되며 delimiter : '|'로 구분된다.
        capture request header Host len 128
        capture request header User-Agent len 64
        capture request header Referrer len 64

        # 다음은 custom format 을 위한 지
        # %ci : client ip
        # %trl : local date time
        # %HM : HTTP method (ex GET)
        # %HU : HTTP request URI (ex: /foo?bar=baz)
        # %HV : HTTP version (ex: HTTP/1.1)
        # %ST : Status Code
        # %B : bytes_read
        # %hr : header values (host, agent, referrer) 위에서 capture 한 값들.
        # %s : server_name
        # %b : backend_name
        # %TR : time to receive the full request from 1st byte
        # %Tw : total time in milliseconds spent waiting in the various queues
        # %Tc : total time in milliseconds spent waiting for the connection to establish to the final server, including retries
        # %Tr : total time in milliseconds spent waiting for the server to send a full HTTP response, not counting data
        # %Ta : time the request remained active in haproxy, which is the total time in milliseconds elapsed between the first byte of the request was received and the last byte of response was sent
        # %ac : total number of concurrent connections on the process when the session was logged
        # %fc : total number of concurrent connections on the frontend when the session was logged
        # %bc : total number of concurrent connections handled by the backend when the session was logged
        # %sc : total number of concurrent connections still active on the server when the session was logged
        # %rc : the number of connection retries experienced by this session when trying to connect to the server
        log-format "%ci\ [%trl]\ %HM\ \"%HU\"\ \"%HV\"\ %ST\ %B\ %hr\ %s\ %b\ %TR/%Tw/%Tc/%Tr/%Ta\ %ac/%fc/%bc/%sc/%rc"

        # 기본적으로 사용할 backend 를 지정한다.
        default_backend web-svr

    backend web-svr

        # 로드 밸런싱을 라운드 로빈으로 지정한다. 이밖에도 leastconn 을 많이 지정하는것 같다.
        balance roundrobin

        # 연결할 서버들이 active health check를 지정한다. (nginx 와 가장큰 다른점이라고 생각된다. nginx 는 active health check 가 plus(유료)버전에서만 가능하다)
        option  httpchk HEAD /_health HTTP/1.1\r\nHost:\ localhost

        # 에러 파일 설정
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

        # backend 서버들로 request-요청 을 proxy 로 전달할 때 Protocol(https) 와 Port 정보를 함께 전달
        # X-Forwarded-For 는 앞의 option 으로 전달됨 (option forwardfor)
        http-request set-header X-Forwarded-Port %[dst_port]
        http-request add-header X-Forwarded-Proto https if { ssl_fc }

        # 연결할 서버들의 리스트다. 3번 health check 가 실패하면 빼고, 2번 성공하면 다시 LB 대상에 포함시킨다.
        server  my-web-server-1-host   10.10.192.1:80  check fall 3 rise 2
        server  my-web-server-2-host   10.10.192.2:80  check fall 3 rise 2
        server  my-web-server-3-host   10.10.192.3:80  check fall 3 rise 2
        server  my-web-server-4-host   10.10.192.4:80  check fall 3 rise 2
        server  my-web-server-5-host   10.10.192.5:80  check fall 3 rise 2
        server  my-web-server-6-host   10.10.192.6:80  check fall 3 rise 2
        server  my-web-server-7-host   10.10.192.7:80  check fall 3 rise 2
        server  my-web-server-8-host   10.10.192.8:80  check fall 3 rise 2
        server  my-web-server-9-host   10.10.192.9:80  check fall 3 rise 2
        server  my-web-server-10-host   10.10.192.10:80  check fall 3 rise 2

    ```

2. 설정 파일이 완료되었다면 아래와 같이 이상이 없는지 체크할 수 있다.

    ```
    sudo haproxy -f /etc/haproxy/haproxy.cfg -c
    ```

### 구동

1. 이제 `systemd` 를 사용해서 구동해보자

    ```
    systemd start haproxy.service
    ```

2. 재시작은 `reload` 하면된다. 만약 컴파일시에 `USE_SYSTEMD` 옵션을 주지 않았다면 재시작에 실패할 수 있다.

    ```
    systemd reload haproxy.service
    ```


### 설치후 추가할일

1. rsyslog 에 haproxy 용 로그를 남기도록 설정한다. `/etc/rsyslog.d/haproxy.conf` 파일을 다음과 같이 구성한다.

    ```
    # Provides UDP syslog reception
    $ModLoad imudp
    $UDPServerRun 514
    $template Haproxy, "%msg%\n"
    #rsyslog 에는 rsyslog 가 메세지를 수신한 시각 및 데몬 이름같은 추가적인 정보가 prepend 되므로, message 만 출력하는 템플릿 지정
    # 이를 haproxy-info.log 에만 적용한다.

    # 모든 haproxy 를 남기려면 다음을 주석해재, 단 access log 가 기록되므로, 양이 많다.
    #local0.*   /var/log/haproxy/haproxy.log

    # local0.=info 는 haproxy 에서 에러로 처리된 이벤트들만 기록하게 됨 (포맷 적용)
    local0.=info    /var/log/haproxy/haproxy-info.log;Haproxy

    # local0.notice 는 haproxy 가 재시작되는 경우와 같은 시스템 메세지를 기록하게됨 (포맷 미적용)
    local0.notice   /var/log/haproxy/haproxy-allbutinfo.log
    ```

2. logroate 에 haproxy 설정을 추가한다. `/etc/logrotate.d/haproxy` 파일을 다음과 같이 구성한다. haproxy 는 재시작할 필요가 없으므로 rsyslog 를 재시작해준다.

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


### 설치 후기

* 익숙해 지는데 시간이 걸리긴 했지만, 성능도 좋고 원하는 대부분의 기능이 제공되므로 만족스러웠다. 다만 설정과 관련된 방식이 친절하지 못하다고 느껴지는건 단순히 익숙하지 않음 때문은 아닌듯.
그리고 여전히 문서를 잘 읽어야 한다는 생각이 든다. 특히나 소스 압축 풀고 `example` 폴더 한번만 주의 깊게 봤으면 고생을 덜 했을 텐데 싶다.

### 참고
  - https://cbonte.github.io/haproxy-dconv/1.8/configuration.html
  - http://git.haproxy.org/?p=haproxy-1.8.git;a=blob_plain;f=contrib/systemd/haproxy.service.in
  - https://www.upcloud.com/support/haproxy-load-balancer-centos/
  - http://blog.whitelife.co.kr/321
  - https://medium.com/@jinro4/%EA%B0%9C%EB%B0%9C-haproxy-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EA%B8%B0%EB%B3%B8-%EC%84%A4%EC%A0%95-f4623815622
  - http://sseungshin.tistory.com/77