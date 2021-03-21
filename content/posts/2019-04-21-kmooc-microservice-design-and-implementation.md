---
date: '2019-04-21 21:33:00 +09:00'
group: blog
image: /images/posts/kmooc/microservice-design-and-implementation.png
tags:
- "kmooc"
- "Microservice 설계 및 구현"
title: "[리뷰] Kmooc-Microservice 설계 및 구현 수강 후기"
url: /2019/04/21/kmooc-review-microservice-design-and-implementation
type: post
---

우연찮게 [[kmooc] Microservice 설계 및 구현](http://www.kmooc.kr/courses/course-v1:KAISTk+2018_K14+CS490/course/) 강의를 신청해서 듣게 되었다. 
큰 기대를 하지 않고 시작했는데 예상보다 개념을 정리하는데 도움이 되었기에, 수강 후기를 작성해보았다.

<!--more-->

## kmooc?

[kmooc](http://www.kmooc.kr)는 Korean MOOC(Massive, Open, Online, Course의 줄임말로 '오픈형 온라인 학습 과정') 의 약어이다. 
2015년도 부터 시작했다고 하는데, 주요 강좌들은 대학과 연계한 내용이 많아 보였다. 주로 교수님들의 온라인 강의를 수강할 수 있도록 정리되어 있었다. 
해외의 [coursera](coursera.org) 한국어판 이라고 생각하면 된다. 나도 이번에 처음 알게되었다.

## 'Micronservice 설계 및 구현' 강의

나는 kmooc 사이트에 관심을 가지고 있었던 것도 아니였고, 페이스북에서 우연히 "Microservice 설계 및 구현" 강의를 오픈한다는 광고(?)를 보고나서
최근에 마이크로 서비스 설계의 고민을 해결하는데 도움이 될까 해서 신청하게 되었다. 기간은 `19.02.18. ~ 19.04.28.` 강의 8주 + OT + 기말시험의 구성이었다.
정해진 기간안에 수강을 완료하면 수강증을 준다는데, 차후에도 청강형태로 영상을 볼 수 있는 시스템이었다. 수강증 보다는 내용이 도움이 되기를 바라면서 수강을 시작했다. 


## 시작하기 전에

1. 사실, 강의 시작전에 약간의 불안감(?)이 있었다. 왜냐하면, kmooc의 대상 유저가 글로벌(?)이 아닌 한국어 가능자만을 대상으로 하다보니, 
콘텐츠의 내용이 충분히 퀄리티가 좋을까? 내용이 만족스러울까 하는 우려가 있었기 때문이다. 특히나 내가 관심있는 IT 주제들은 영어로된 자료가 나오고 나서
한~~~참 뒤에서야 한국어로 번역되어서 전파되기 마련이라, 너무 뒤쳐진 이야기나 수박 겉핥기 내용만 나오지 않을까 걱정스러웠다. 
그렇지만 또 한편으로는 `마이크로서비스` 라는 키워드는 이미 많은 곳에서 이야기 되고 있었기에 내용 괜찮기를 바랬었다.

2. 강좌는 무료이기 때문에 부담없이 신청이 가능했지만, 무료인만큼 동기부여가 잘 되지 않으면 어떻하지 하는 걱정도 있었다. 
(유료라.. 돈이 아깝지 않으면 안듣게 되더라.. 하는..걱정)

## 시작하고 나서.

1. 강좌를 시작하고 나서 약간 놀란 것은 강좌를 진행하시는 분이 교수님이 아니라, SK에서 실무에 종사하는 분들(!) 이라는 것이었는데, 두명의 수석 or 책임 연구원(?)정도의
직책에 계신분들이 전반/후반을 나눠서 진행을 하셨다. 두분의 목소리뿐만 아니라, 얼굴(!)도 볼 수 있는데, 약간 굳어 있는 표정에서, (발표도 어려운데, 영상에서 얼굴이 나온다니..)
묘하게 안쓰러움이....

2. 강의는 후반의 약간의 [코드](https://github.com/haesiku/kmooc)를 보여주는 부분 이외에는 이론적 개념설명이 주를 이룬다. "설계 및 **구현**"에서 
구현 코드 따라하기를 기대했던 나는 잘못 생각한 것이었다(쳇..)

3. 강의 초반에는 솔직히 지루한 느낌이었다. 뻔한 이야기를 그대로 답습하는 느낌이랄까.. 솔직히 교재만 보고 후다닥 넘겨 버릴까 했었지만, 분량 자체가 그렇게 길지 않아서
바로바로 해치워 버린다(!)라는 생각으로 다 봤다. 강의 후 주차별로 나오는 퀴즈는 난이도가 높지 않아 강의를 듣지 않고도 풀 수 있는 정도이다. 초반의 지루함을 이겨내고
매주 수강을 이어갈 수 있었던건. 꼬박꼬박 월요일 마다 날아오는 강의 주차 알림 이메일 덕분이 아니였을까.. 

4. 8주차 중에서 4~5 주차 정도 지나서 어느정도 적응하고 나니, 내용이 제법 도움이 된다는 생각이 짙어졌다. 안그래도 업무에서 DDD, Event Driven에 고민이 있었는데
강의의 내용이 딱 그런 내용들이었다. 그동안에 몸으로 느끼고 있었던 것들의 이론적 배경을 되짚어보는 느낌이랄까. 개념들의 핵심 키워드를 빠르게 인지 할 수 있고, 
생각들을 환기 시킬 수 있어 괜찮았다.

5. 용어 설명은 너무 개괄적인 한계가 있었다. 짧은 시간에 무언가 깊이를 기대하기 어려운게 사실이고, 핵심 키워드 또한 한번더 검색을 해보는게 도움이 되었다.



## 후기

1. kmooc 에 대한 걱정반 기대반으로 시작한 강좌였지만, 나름 만족스러웠다. 결국 이런 제도들은 콘텐츠가 핵심인데, 적어도 나에게는 도움이 되었다. 앞으로도 괜찮은 강좌가 개설되면
더 찾아서 들어볼 생각이다. 일단 부담이 없으니까. 그렇지만 더 재미있는 내용이 많았으면 좋겠다. 

2. 실제 강좌에는 DDD 가 절반, 나머지 절반은 마이크로 서비스 설계인것 같다. 마이크로 서비스 보다 DDD 의 개념 정리하는데 도움이 되었다.

3. DDD와 관련된 복잡한 용어들이 많이 나오지만, 결국 자세히 생각해 보면, 당연한 이야기를 어렵게 한다는 느낌이 들었다. 용어에 매몰되기 보다, 현장에서의 적용을 생각하면 좀 더 쉽게 이해된다.
한편으로는 과거 디자인 패턴도 익히고 나니 결국 "코드의 패턴에 이름을 붙인것" 이라는 생각이 들었듯이, DDD 도 어떻게 설계할 것인가 이미 실무에서 진행중인 방법에 이름을 붙인거라는 생각이 든다.
따지고 보니, 이미 나는 유비쿼터스 언어로 서로 대화하고 있었고, 기획, 개발, 디자인, Product Owner 가 모두 모여 함께 설계하고 이야기 했었다. 
기획이 기획만 하고 개발이 개발한 하는건 나에게는 이미 아닌 이야기.

4. 전체 러닝 타임이 5시간이 조금 못 미치는데, 이 정도 시간을 투자해서 주요 용어들을 빠르게 정리해서 훑어 볼 수 있었다는 데 만족스럽다. 다만 DDD, 마이크로 서비스에 대해서 아무것도 모르는 사람이라면, 
크게 도움이 되지 않을것 같다. 

## 요약

- kmooc 생각보다 나쁘지 않았다.
- 코드 실습은 없다. 이미 어느정도 알고 있는 사람이 듣는다면, 개념과 용어 정리에 적합하다.
- 5시간 정도 투자해서 빠르게 훑어 볼 수 있는 시간이 괜찮음
- DDD, 마이크로 서비스 처음 익히는 사람이라면 도움되지 않을 듯.

### 참고
 * Microservice 설계 및 구현 강좌 링크 : http://www.kmooc.kr/courses/course-v1:KAISTk+2018_K14+CS490/course/

