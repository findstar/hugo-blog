---
aliases:
    - /til/2018-01-04-TIL
categories:
- TIL
date: '2018-01-04'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- phpstorm
- speed up
- unused plugins
title: PHPStorm에서 사용하지 않는 플러그인 비활성화 시키기.
url: /2018/01/04/disable-unused-plugins-phpstorm
type: post
---


# 1월 04일 (목) TIL

phpstorm이 조금 버벅거리면서 느린 느낌이 들었다. 내 경우에는, JavaScript 와 php 가 같이 들어 있는 템플릿 파일을 수정하거나, 마크다운MD 파일을 수정할 때 그런 증상들이 심해졌다.
일단 사용하지 않는 플러그인들을 비활성화 시켜서 속도가 조금 개선되는지 확인해 보기로 했다.

아래의 플러그인들은 사용하지 않음.

1. ASP : ASP 코딩 할일이 없어서 해제
2. Behat : BDD framework 인 Behat을 사용하지 않아서 해제
 (이걸 uncheck 하면 연결된 codeception framework 도 해제된다)
3. CoffeeScript : 커피스크립트 코딩 할일이 없어서 해제
4. CVS : subversion 이전에 활약하던 CVS 이다. 나는 git만 사용하므로 해제
5. Drupal support : Drupal 프레임워크를 사용하지 않으므로 해제
6. Gherikin  : 스타트랙을 보긴했지만. 이건 그냥 위트용이다. 해제
7. Google App Engine Support : GAE 연동을 하지 않으므로 해제 (난 AWS...)
8. Haml : 사용하지 않아 해제
9. Handlebars/Mustache : 이건 사용하는 사람은 제법 있을 수 있지만, 나는 사용하지 않는 템플릿이라 해제
<!--more-->
10. hg4idea : Mercurial version control system을 사용하지 않아 해제.
11. Joomla : Joomla 프레임워크도 사용하지 않으므로 해제
12. Perforce : perforce VCS 사용하지 않으므로 해제
13. Phing : PHP 빌드툴인 Phing 이지만 사용하지 않으므로 해제(참고로 apache ant 기반의 빌드 툴이다.)
14. PHPSpec : Behat 처럼 BDD framework 이지만 사용하지 않아 해제
15. Subversion : Git 만 쓴다.. subversion 도 안녕~ 해제.
16. TextMate : TextMate 도 사용하지 않아 해제.
17. Twig : Twig 템플릿도 사용하지 않아 해제
18. WordPress Support : Wordpress 도 사용하지 않는다. 해제.
19. Vagrant : Vagrant 를 사용할 수 있지만, IDE 레벨에서 연동하지 않는다. 해제.
20. tslint : Type Script Lint 사용하지 않아 해제.

사용하지 않는 플러그인들을 해제하고 apply 하면 IDE를 재시작하겠냐고 물어보는데 바로 재시작해줬다.
약간(?) 쾌적해진 듯 느껴지지만, 플라시보 인것 같기도.. 다음은 Heap Size를 늘리도록 VM Options 을 조정해봐야겠다.