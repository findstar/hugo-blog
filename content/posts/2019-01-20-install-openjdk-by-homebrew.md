---
date: '2019-01-20 22:07:01 +09:00'
group: blog
image: /images/posts/openjdk/openjdk-logo.png
tags:
- "openjdk"
- "brew cask"
title: "homebrew로 openjdk 설치하기 "
url: /2019/01/20/install-openjdk-by-homebrew
type: post
---

사용하던 노트북을 신형 맥북프로로 변경하면서 이런저런 개발 환경을 다시 구성하게 되었다. 마침 OpenJDK로 jdk 환경을 변경해보고자 하였는데 이때 homebrew로 OpenJDK 를 설치해보았다.

<!--more-->

## OpenJDK

[OpenJDK(Open Java Development Kit)](https://openjdk.java.net/)는 자바 플랫폼, 스탠더드 에디션 (자바 SE)의 자유-오픈 소스 구현체이다. 최근 자바가 유료화 되면서 한층 주목받고 있는데, (자바가 유료라니..)
유료화에 대한 반발(?)로 주변의 많은 사람들이 OpenJDK를 설치하는 모습을 볼 수 있었다. 나는 새롭게 개발 환경을 구성하면서 OpenJDK를 설치해보고자 하였는데, 처음에는 생각없이 `brew` 명령어를 실행했었다.

```
brew install openjdk
```

하지만 보기 좋게 실패했다.(!!!)

찾아보니 아직은 공식적으로 brew 를 통해서 설치가 불가능하다. [brew issue link](https://discourse.brew.sh/t/how-to-install-openjdk-with-brew/712)

따라서 OpenJDK 를 brew로 설치하려면 공식이 아닌 비공식(?) 경로를 통해서 설치해야 한다.

## AdoptOpenJDK

AdoptOpenJDK는 사전에 prebuild 형태로 java binary를 제공하는 커뮤니티 그룹이다. [홈페이지](https://adoptopenjdk.net/) Mac 뿐만 아니라 윈도우, 리눅스 환경도 제공하고 있다.
공식적으로 OpenJDK를 설치하는건 직접 빌드해서 사용하는 방법이 있지만, 빌드 이외에도 자잘한 `JAVA_HOME` (빌드해서 사용했더니 "/usr/libexec/java_home" 이 동작하지 않았다..) 설정 문제라던가
버전업을 편하게 하기 위해서 homebrew를 사용해서 AdoptOpenJDK를 설치하도록 했다.

다음 [Github](https://github.com/AdoptOpenJDK/homebrew-openjdk) 에서 설치방법을 확인할 수 있다.

```
brew tap AdoptOpenJDK/openjdk
brew cask install <version>
```

OpenJDK 버전은 다음중 하나를 선택하면 된다.

- OpenJDK8 - `adoptopenjdk8`
- OpenJDK9 - `adoptopenjdk9`
- OpenJDK10 - `adoptopenjdk10`
- OpenJDK11 - `adoptopenjdk11`
- OpenJDK11 w/ OpenJ9 JVM - `adoptopenjdk11-openj9`

나는 JDK8를 사용하기 위해서 다음과 같이 진행했다.

```
brew tap AdoptOpenJDK/openjdk
brew cask install adoptopenjdk8
```

이후 빌드했을 때와 다르게 JAVA_HOME 문제도 없고, 다른 경로 문제도 없이 잘 동작하는 것을 확인할 수 있었다.

{{< imageFull src="/images/posts/openjdk/java_version_cli.png" title="Java Version" border="false" >}}


### 참고
 * openjdk란 위키 : https://ko.wikipedia.org/wiki/OpenJDK
 * OpenJDK 홈페이지 : https://openjdk.java.net/
 * AdoptOpenJDK 홈페이지 : https://adoptopenjdk.net/

