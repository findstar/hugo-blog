---
date: '2016-11-06'
description: composer speed up
group: blog
image: /images/posts/composer-boost.jpg
tags:
- tips
- composer
- speedup
title: 컴포저의 속도를 올리는 방법
url: /2016/11/06/composer-speed-up
type: post
---


PHP 의 의존성 관리도구인 [컴포저](https://getcomposer.org/ "컴포저(composer) 공식 사이트") 를 사용할 때, `composer install` 이나 `composer update` 시에 속도가 느려서 답답함을 느낄 때가 한두번이 아니다. 구글에서 `composer speed up` 으로 검색을 해보면 많은 사람들이 동일한 답답함을 느끼고 있다는 것을 확인할 수 있다.

몇가지 방법을 통해서 조금 더 쾌적하게 사용할 수 있는 방법을 알아보자.

<!--more-->

컴포저의 속도를 개선할 수 있는 방법은 플러그인과, 커맨드 사용 패턴, 옵션 설정 등이 있는데 하나씩 살펴보자.

## Prestissimo Plugin 설치하기

[prestissimo](https://github.com/hirak/prestissimo "prestissimo github 프로젝트 페이지")는 컴포저의 global 플러그인중 하나로 컴포저의 다양한 의존성 패키지들을 병령로 다운받을 수 있게 해주는 녀석이다.

내부적으로 curl multi 옵션을 통해서 의존성 패키지들을 다운받도록 해서 속도 향상에 큰 기여를 하는 방식이다. 컴포저를 자주 사용한다면 설치하지 않을 이유가 없다!

설치방법도 간단하다

> $ composer global require "hirak/prestissimo:^0.3"

이렇게 하면 global composer 디렉토리에 설치가 된다.

github에서 제작자는 벤치마크 결과가 288s -> 26s 로 줄어들었다고 말하고 있다.

## –prefer-dist 옵션 사용하기

composer update 또는 composer install 시에  *–prepfer-dist* 사용하면 패키지에 따라서 조금 더 나은 속도 향상을 기대할 수 있는데, 패키지가 `dist` 용도로 구성되어 있다면 소스를 일일이 받는 것 보다 빠르기 때문이다.

좀더 부연 설명을 하자면 패키지를 다운로드 받을 때는 *dist* 와 *source* 두가지 방식이 있는데, *dist*는 안정화 버전, *source* 버그 픽스를 위한 용도라고 생각하면 된다.

일반적으로는 *-prefer-dist* 옵션이 활성화 되어 있다.

## xdebug disable 시키기

php-xdebug 가 활성화 되어 있으면 컴포저의 속도가 느리다. 초기에는 이를 알수가 없었는데 최근에는 컴포저 자체에서 경고를 보여주는 것 같다.

만약 php 설정이 cli 와 구분되어 있다면 cli 모드에서는 xdebug 를 활성화 시키지 않도록 하자.

## dns lookup 줄이기

속도에 얼마나 영향을 끼칠지는 의문이지만, packagist.com 의 dns lookup 속도는 느리다고 할 수 있다. (ping packagist.com 해보면 알 수 있다.) 따라서, /etc/hosts 에 packagist.com 의 ip를 추가하여 DNS lookup time 을 조금이나마 줄일 수 있다.

> 87.98.253.214 packagist.com


### 참고

 - [컴포저 한글 매뉴얼](http://xpressengine.github.io/Composer-korean-docs/doc/03-cli.md/#install "컴포저 한글 매뉴얼")
 - [composer manual](https://getcomposer.org/doc/03-cli.md#install "composer 매뉴얼 cli 부분")
 - [prestissimo plugin](https://github.com/hirak/prestissimo "prestissimo 프로젝트 페이지")
 - [xdebug disable warning](https://getcomposer.org/doc/03-cli.md#composer-disable-xdebug-warn "composer 매뉴얼 xdebug 부분")