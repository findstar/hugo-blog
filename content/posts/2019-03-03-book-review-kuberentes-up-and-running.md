---
date: '2019-03-03 01:03:31 +09:00'
group: blog
image: /images/posts/book/kubernetes_up_and_running.png
tags:
- "book"
- "쿠버네티스 시작하기"
- "kubernetes"
title: "[북 리뷰] 쿠버네티스 시작하기(kubernetes up & running)"
url: /2019/03/03/book-review-kubernetes-up-running
type: post
summary: "쿠버네티스 스터디에서 학습을 위해서 읽은 `쿠버네티스 시작하기(원서 kubernetes up & running)`의 리뷰이다. 쿠버네티스 스터디로는 두번째인데, 첫번째 스터디에서 마무리까지 참가하지 못해서
  못내 아쉬운 마음에 팀 동료들과 두번째 스터디를 다시 열었다. 에이콘에서 출간한 번역서로 원서는 O-REILLY가 출판사이다." 
---

# [북 리뷰] 쿠버네티스 시작하기(kubernetes up & running)

## 책 소개

원서는 오라일리에서 출간한 `Kubernetes Up & Running` 라는 제목이다. 에이콘 출판사에서 번역해서 내놓은 책인데, 번역서 특유의 번역체(?)가 좀 거슬린다.
가격은 2만원으로 저렴한 편이라고 생각된다. 책을 구매당시에는 평점 확인이 힘들었는데, 지금 보니 yes24 평점이 4점이다. 평점이 낮을만한 이유는 아래에서 설명하겠다.

{{< imageFull src="/images/posts/book/kubernetes_up_and_running_kr.jpg" title="Kubernetes up and running" border="false" >}}

## 책 구매 이유

도커와 함께 쿠버네티스를 실무에서 사용하기 위해서 공부가 필요했다. 쿠버네티스를 주제로한 외부 스터디에서 참석했었던 경험이 있었는데 업무가 바빠져 완료를 하지 못했었다.
아쉬움에 팀 동료들이 쿠버네티스 스터디를 진행하기로 하여 이 책을 선정해서 함께 요약발표하기로 했다.
다음은 선정 기준.

- 최대한 최신 버전의 쿠버네티스를 이야기 할것!
- 상대적으로 책이 얇을것.(익숙해 지는 단계라는 점을 고려했다.)
- 대상 독자가 쿠버네티스 입문자일것.

## 소감

책을 다 읽기 까지는 1주에 2~3시간씩 약 5주정도 소요되었다. 초반에는 그냥 무난했는데, 뒤로 가면 갈수록 번역체 특유의 거슬림(?)이 있다.
특히나 챕터 7장의 `서비스 탐색` 이라는 용어는 정말이지 끝까지 적응안되는 용어였는데, 그냥 원문 `Service discovery` 라고 쓰는게 나았다고 생각한다.
후반부의 급격한 번역의 퀄리티 저하는 그렇다 치고, 원저자가 의도한 것인지는 모르겠지만
`Deployment` 개념을 12장에 가서야 설명한다. 그런데 정작 12장 이전에 군데군데 계속 개념적으로 연결된 내용들이 나온다.
따라서 12장에 도달하기도 전에 이미 `Deployment`는 구글링을 통해서 알게된 상태가 된다.


마지막으로 쿠버네티스는 그 개념이 요소요소 마다 방대한데, 책에서 챕터별로 이를 축약하다보니, 생략된 부분이 많고, 아무리 가볍게 보려고 해도
핵심적인 개념전달 보다, 장황하게 풀어쓴 느낌이 들었다. 그래서 책을 다 읽은 뒤에도 뭔가 시간낭비한 느낌이 들었다. 책에서 그나마 나은점을 꼽으라면
실습을 위한 [yml 파일 모음](http://acornpub.co.kr/download/kubernetes-up-and-running) 을 제공한다는 점이다.

차라리 이전에 읽었던 원서 : [The DevOps 2.3 Toolkit: Kubernetes](https://leanpub.com/the-devops-2-3-toolkit)가 더낫다.

## 한줄 요약

* "쿠버네티스 입문용으로 아쉬운 책. 비추천"

### 참고
 * 에이콘 링크 : http://acornpub.co.kr/book/kubernetes-up-and-running


