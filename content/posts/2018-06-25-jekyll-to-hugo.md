---
date: '2018-06-25'
group: blog
tags:
- hugo
- "jekyll to hugo"
- migration
title: jekyll 에서 hugo 로 전환한 소감
url: /2018/06/25/jekyll-to-hugo-thoughts
type: post
---

사용하던 github pages 기반 툴을 `jekyll` 에서 `hugo` 로 전환해보았다. 아래는 그 과정에서 느낀 점을 정리해보았다.

<!--more-->

## 전환하면서 느낀 점

### `hugo` 전환한 이유

`jekyll` 에서 `hugo` 로 전환한 이유는 아래와 같다.

- `hugo` 가 반응속도가 훨~~씬 빠르다는 것. (jekyll 은 2 s, hugo 는 70ms )
- theme 를 커스터마이징 하기가 손쉽다는 것.
- 매뉴얼이 잘 구성되어 있고, 심지어 영상으로 소개해준다는 것.
- 그리고 무엇보다 `jekyll-paginate v2` 를 쓰다가 `github page` 에서 동작을 안해서.. (사실 이게 제일 크다)


### `hugo` 에서 불편한 점 

- `jekyll` 에서는 plugin 으로 구현 가능한 것들이 직접 핸들링 해줘야 하는 부분들이 있었다.
- `github page` 가 native 로 연결되지 않아서 별도로 관리해주어야 한다는 점.
- 템플릿 문법이 상대적으로 낯설어서 적응하는데 애를 먹었다. (RTFM)


## 전환 작업하면서 알아야 했던 것들

* theme 는 별도로 구성하는게 편하다

    매뉴얼에는 별도의 theme 프로젝트를 git submodule 로 관리하는걸 안내하고 있다. 내 경우는 아예 새로 만들었다. 

* 기본 템플릿 

    `hugo` 에는 list 템플릿, single 템플릿 두 가지가 기본 템플릿이다. 테마를 구성한다면 요 차이를 명확하게 구분하자.

* pagination 은 jekyll 이 더 편해보인다.

* seo, tag archive 같은 기능은 별도로 custom page 를 만들어서 쓰는게 편했다.

* shortcode 라는 별도의 custom 단축기능을 만들 수 있다. 내 경우는 CSS class 와 맞추기 위해서 아래처럼 구성해서 썼다. 

    ```html
    <figure class="full-width caption" {{ with .Get "border" }} style="border: 1px solid #ededed;" {{ end }}>
        <img src="{{ .Get "src" }}" alt="{{ .Get "alt" }}"/>
        {{ with .Get "caption" }}
        <figcaption class="caption-text">{{ . }}</figcaption>
        {{ end }}
    </figure>
    ```

* related post 를 구성하는 건 옵션이 많으니 입맛에 맞게 잘 맞춰야 한다.


## 전체적인 소감

`hugo`는 `github page` 를 직접 지원하지 않아서 별도의 **build & push** 를 해줘야 하지만 이건 크게 문제되지 않는 것 같다. 오히려 빨라진 반응 속도가 더 쾌적한 느낌이 들고, 
빠른 업데이트와 함께 매뉴얼이 잘 마련되어 있다는 점이 긍정적이라고 느껴진다. 찔끔찔끔 마이그레이션 하느라 시간은 2주 정도 걸렸는데 하고나니 잘 했다는 느낌이다. 이제는 테마는 그만좀 고치고 글을 좀 더 써야 할텐데..

  