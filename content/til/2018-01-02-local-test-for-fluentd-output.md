---
title: Fluentd 를 로컬에서 테스트 해보는 방법
date: '2018-01-02'
group: til
tags:
    - fluentd
aliases:
    - /til/2018-01-02-TIL
categories:
    - TIL
comments: true
permalink: /til/:year-:month-:day-TIL
type: post
---


# 1월 02일 (화) TIL
- [Fluentd](https://www.fluentd.org/)

fluentd (이하 td-agent)를 설치후에 conf를 설정하고 나면 동작을 잘 하는지 확인해야 되는 경우가 있는데 아래와 같이 테스트 해볼 수 있었다.
`/etc/td-agent/td-agent.conf` 가 다음과 같을 때:

```
<source>
.....
.....
.....
</source>

# 결과 값에 hostname 을 추가로 덧붙임.
<filter accesslog>
  @type record_transformer
  <record>
    source "#{Socket.gethostname}"
    path ${record["host"]}
  </record>
</filter>

# Local 에서 테스트 해볼 때 사용할 수 있는 match
<match accesslogs>
 type stdout
</match>
```

<!--more-->

> $ td-agent -c /etc/td-agent/td-agent.conf

위와 같이 입력하면 콘솔상에서 standard out을 통해서 match 되는 결과를 확인할 수 있다. 만약 여기에서 error 나 warning 을 확인한다면,
td-agent.conf 파일의 source 와 filter를 적절하게 수정해서 다시 테스트 해서 설정을 맞추면 된다.