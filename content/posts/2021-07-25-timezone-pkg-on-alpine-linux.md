---
date: 2021-07-25T23:28:36+09:00
group: blog
image: /images/posts/alpine/alpine-logo.png
tags: ["alpine linux", "timezone"]
title: "Alpine 리눅스에서 timezone 설정하기"
url: /2021/07/25/timezone-pkg-on-alpine-linux
type: post
summary: "도커 빌드에서 자주 사용되는 `Alpine` 리눅스에서 Timezone을 지정하는 방법을 살펴보았다."
---

# Alpine 리눅스에서 Timezone 설정하기
도커를 빌드할 때 주로 사용하는 것이 바로 Alpine 리눅스이다. 도커를 사용할 때 경량의 이미지를 기반으로 하면 이미지 pull 속도가 빨라 빠른 배포에 도움이 되어 자주 사용하고 있다. 다만 너무 경량이다 보니 필요한 패키지를 추가로 설치해줘야 하는 경우가 있는데 이미지에 timezone 을 추가로 지정하는 방법을 살펴보았다. 

## Alpine Linux 

Alpine 리눅스는 보안, 단순함, 적은 자원을 사용하는 리눅스로 [위키 링크](https://wiki.alpinelinux.org/) 에서 대부분의 매뉴얼을 찾을 수 있다. [도커 허브](https://hub.docker.com/_/alpine)에서는 베이스 이미지로 제공하고 있으며 로컬 머신에서 이미지를 다운로드 받아보면 얼마나 작은 사이즈를 유지하는지 확인할 수 있다. 다음처럼 확인했을 때 불과 5.6MB 에 이미지 사이즈를 확인할 수 있었다. 

```shell
$ docker image pull alpine:latest
Using default tag: latest
latest: Pulling from library/alpine
5843afab3874: Already exists
Digest: sha256:234cb88d3020898631af0ccbbcca9a66ae7306ecd30c9720690858c1b007d2a0
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest

$ docker images
alpine    3.14.0    d4ff818577bc   5 secods ago     5.6MB
```

## timezone 설정하기

정확한 내용은 [위키 문서](https://wiki.alpinelinux.org/wiki/Setting_the_timezone)를 참고하였다. 일단 alpine 에서는 yum, apt 와 같은 패키지 매니저를 사용할 수 있는데 바로 `apk` 명령어다. 타임존을 설정하기 위해서는 `tzdata` 패키지가 필요하다.

```shell
apk add tzdata

```

패키지를 설치하는 것에 더해 `/etc/timezone` 을 지정해줘야한다.

```shell
echo "Asia/Seoul" >  /etc/timezone
```

## 도커에서 설정하기

도커파일에서는 다음과 같이 사용하고 있다.

```shell
ENV TZ=Asia/Seoul
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone
```

# 참고자료

- [알파인 리눅스 도커 허브 이미지](https://hub.docker.com/_/alpine)
- [timezone 지정 위키 문서](https://wiki.alpinelinux.org/wiki/Setting_the_timezone)
