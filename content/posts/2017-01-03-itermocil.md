---
date: '2017-01-03'
description: SSH connect to multiple server by itermocil
group: blog
image: /images/posts/itermocil.png
tags:
- ssh
- iterm
- macro
title: itermocil을 사용해서 다수의 서버에 SSH 접속하기
url: /2017/01/03/itermocil
type: post
---


얼마전 작업하는 서비스의 서버의 OS 교체를 진행했다. 이 과정에서 사용하게된 itermocil 툴을 소개해본다.

<!--more-->

처음 계획으로는 Docker 기반의 배포 환경을 구성하려고 했는데, 시간이 모자라 진행하지는 못했고,

예전 보다 조금 더 나은 수준의 배포환경을 구성하는 데 만족해야만 했다.

Centos7 환경을 구성하였는데, 설치 한 뒤로도 자잘하게 환경설정과, 몇가지 권한설정을 추가적으로 진행했었다.

이때 불편한 점이 하나 있었는데, 그것은 바로 `한번에 다수의 서버에 SSH 접속하는 방법` 이다.

내가 사용하는 터미널 프로그램은 [iterm](https://www.iterm2.com/) 인데, Mac 에서 개발은 하는 사람치고 [iterm](https://www.iterm2.com/)을 사용하지 않는 사람을 볼 수가 없을 만큼 사랑받는 프로그램이다.

물론 tmux 나 기타 방법으로도 다수의 서버에 쉽게 접속해 볼 수 있지만, 나는 alias 형태로 predefined 된 macro 같은 툴을 원했고,

이과정에서 [itermocil](https://github.com/TomAnthony/itermocil) 이라는 툴을 발견했다.

## itermocil 소개

이름에서 알 수 있듯이 iterm 에서 사용하는 일종의 매크로라고 할 수 있다.

"pre-configured layouts of windows and panes" 라고 되어 있는데 말 그대로 사전에 정의된 윈도우와 패널 레이아웃 관리자 이다.

사용법도 간단한데

```
itermocil layout-name
```

이렇게 하면 지정된 레이아웃에 정의된 명령어들을 수행하여 준다.

## 설치방법

brew 를 통해서 설치하고 홈 디렉토리에 `.itermocil` 디렉토리를 만들어 준다.

이후에 yml 포맷의 레이아웃 파일을 만들면 된다.

처음에는 레이아웃파일이 없을 테니 샘플을 구성할 수 있게 지원하고 있다


```
# brew 로 설치
$ brew install TomAnthony/brews/itermocil

# 레이아웃을 저장하기 위한 .itermocil 디렉토리 생성
$ mkdir ~/.itermocil

# sample 레이아웃 편집하기
$ itermocil --edit sample

# sample 레이아웃 실행하기
$ itermocil sample
```

## 레이아웃 파일 구성방법

레이아웃 파일을 구성하는 방법을 다양하게 지원하고 있는데, 자세한건 매뉴얼에서 설명하고 있으니, 문서를 참고하길 바란다.

내 경우에는 약 20대의 서버에 접속하기 위해서 다음과 같이 처리했다.

```
windows:
  - name: web-server
    root: ~/project/working-directory
    layout: tiled
    panes:
      - ssh findstar@service-web1.our-product.io
      - ssh findstar@service-web11.our-product.io
      - ssh findstar@service-web2.our-product.io
      - ssh findstar@service-web12.our-product.io
      - ssh findstar@service-web3.our-product.io
      - ssh findstar@service-web13.our-product.io
      - ssh findstar@service-web4.our-product.io
      - ssh findstar@service-web14.our-product.io
      - ssh findstar@service-web5.our-product.io
      - ssh findstar@service-web15.our-product.io
      - ssh findstar@service-web6.our-product.io
      - ssh findstar@service-web16.our-product.io
      - ssh findstar@service-web7.our-product.io
      - ssh findstar@service-web17.our-product.io
      - ssh findstar@service-web8.our-product.io
      - ssh findstar@service-web18.our-product.io
      - ssh findstar@service-web9.our-product.io
      - ssh findstar@service-web19.our-product.io
      - ssh findstar@service-web10.our-product.io
      - ssh findstar@service-web20.our-product.io
```

위에서 내가 사용한 레이아웃 속성은 tiled 인데, 이밖에도 다양한 레이아웃을 지원하니 [문서](https://github.com/TomAnthony/itermocil/blob/master/LAYOUTS.md)를 참고하자.

저렇게 구성한 파일을 web-server.yml로 저장하고 itermocil web-server 라고 입력하면 20대의 iterm panel 등장하는데, 제법 멋져보인다.

{{< imageFull src="/images/posts/itermocil.gif" title="example usage" border="true" >}}

물론 나와 같이 itermocil을 사용하여 다수의 서버에 접속하길 원하는 사람이 얼마나 되는지는 알 수 없지만,

나로써는 유용하게 사용하여 기록으로 남겨놓는다.