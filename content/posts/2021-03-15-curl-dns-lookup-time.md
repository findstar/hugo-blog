---
date: '2021-03-15 22:19:13 +09:00'
group: blog
image: /images/posts/curl/curl-logo.svg.png
tags:
- "curl"
title: "curl 로 측정하는 dns lookup time"
url: /2021/03/15/curl-cnd-lookup-time
type: post
---

k8s 클러스터에서 서비스를 구동하던중 서비스 사이의 dns lookup time 을 확인해야할 필요가 있었다. 
curl 에서 dns lookup time 을 확인하는 방법을 살펴보았다. 

<!--more-->

# curl 로 dns lookup time 까지 얼마나 걸리는지 확인하기

## 배경

k8s 클러스터 서비스들 사이에 dns lookup time 이 얼마나 되는지 확인해야될 일이 있었다. 
`curl` 이 가장 익숙해서 `curl` 로 dns lookup time 을 확인할 수 있는 방법을 알아보았다.  

## 방법 

`curl`은 `--write-out` 옵션을 가지고 있는데 이 옵션을 사용하면 http 요청의 다양한 결과값을 확인할 수 있다.

```
http_code           http 응답의 상태 코드

time_appconnect     요청이 시작되어 SSL/SSH/etc 연결 또는 핸드쉐이크가 완료되었을 때까지의 시간 (초단위)

time_connect        요청이 시작되어 TCP 연결이 되었을때까지의 시간 (초단위)

time_namelookup     DNS 조회가 완료되었을 때까지의 시간 (초단위)

time_pretransfer    파일 전송이 시작되었을때까지의 시간(초단위)

time_starttransfer  첫번째 바이트가 전송되었을 때까지의 시간(초단위)

time_total          전체 작업이 완료되었을 때까지의 시간(초단위)

```

이 중에서 dns lookup time 까지의 시간을 측정하려면 `time_namelookup` 을 사용하면 된다. 

다음과 같이 실행할 수 있다. 

```
curl --write-out '%{time_namelookup}' https://google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.co.kr/">here</A>.
</BODY></HTML>
0.001519%
```

요청 결과로 확인되는 응답은 보고 싶지 않으니 다음과 같이 결과 값을 표시하지 않도록 변경한다. 
`-o /dev/null` 을 추가한다. 

```
curl -o /dev/null --write-out '%{time_namelookup}' https://google.com
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   220  100   220    0     0    814      0 --:--:-- --:--:-- --:--:--   814
0.002023%
```

응답 결과를 제거하였지만 진행 상황을 나타내는 프로그레스가 아직 출력되어 지저분하다. 이것도 줄여보자.
`-s` 을 추가한다. (silent)

```
curl -o /dev/null -s --write-out '%{time_namelookup}' https://google.com
0.001426%
```

단 한번의 응답 값으로는 확인하기 어려우니 여러번 수행한 결과를 확인해보자. 
간단하게 스크립트를 작성했다. 

```shell
#!/bin/sh

URL=${1}
LIMIT=${2:-1000}


if [[ -z "$URL" ]]; then
  echo "ERROR:
  Useage : {url} {times}
  "
  exit 1
fi

echo "$LIMIT times dns_time check for $URL"

function get_time() {
  url=$1
  t=$(curl -s -o /dev/null -w '%{time_namelookup}' $url)
  echo $t
}

sum=.0
count=0
while [ "$count" -le "$LIMIT" ]
  do
    t=$(get_time $URL)
    count=$((count + 1))
    sum=$(echo "$sum + $t" | bc| sed 's/^\\./0./;s/0*$//;s/\\.$//')
    if [ "$count" -ge $LIMIT ]
      then
        break
    fi
  done

total_time=$(echo "scale=10; $sum * 1000" | bc | sed 's/^\\./0./;s/0*$//;s/\\.$//')
avg_time=$(echo "scale=10; $total_time / $count " | bc | sed 's/^\\./0./;s/0*$//;s/\\.$//')
echo "total taken: $total_time ms, avg_time : $avg_time ms"
```

커맨드라인에서 다음과 같이 입력하면 결과를 확인할 수 있다. 

```
./time.sh http://google.co.kr 10
10 times dns_time check for http://google.co.kr
total taken: 16.338 ms, avg_time : 1.6338 ms
```

dns lookup time만을 확인하기 위해서는 다른 좋은 방법이 있겠지만 
나의 경우에는 k8s 클러스터 내부에서 각 서비스들 사이의 http 요청에 대해서 dns를 변경했을 때
내부에서 소요되는 시간을 측정하는게 목표라서 위와 같이 curl 로 간단하게 확인해 보았다. 
게다가 alpine 리눅스라 이것저것 해볼 여지가 없었다. 

### 참고 
- https://ops.tips/gists/measuring-http-response-times-curl/

