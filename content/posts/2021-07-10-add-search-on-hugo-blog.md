---
date: 2021-07-10T22:33:16+09:00
group: blog
image: /images/posts/fuse-js/fuse-logo.png
tags: ["fuse.js", "search on hugo"]
title: "fuse.js를 사용하여 hugo 블로그에 검색 기능 추가 하기"
url: /2021/07/10/fuse-search-on-hugo
type: post
summary: "그 동안 이 블로그에 검색기능이 빠져 있었다. hugo를 기반으로한 정적 사이트 생성기라서 검색을 추가하기가 어렵다고 생각했었는데, fuse.js 를 사용하면 검색기능을 추가할 수 있어서 사이트에 적용해보았다"
---

# fuse.js 로 Hugo 블로그에 검색 기능 추가하기
그 동안 이 블로그에는 검색기능이 빠져 있었다. 😱 콘텐츠를 제공하는 사이트라면 기본적으로 제공하는 기능인데, 정적사이트에서 어떻게 하는지 몰라서 못하고 있었다. (사실 사이트 유입의 대부분인 구글 검색에 의존하고 있었다.) 그러다가 hugo 를 기반으로한 정적사이트에서 [fuse.js](https://fusejs.io/)를 사용하면 된다는 사실을 알게되어 바로 적용해보았다. 

## fuse.js 

fuse.js 는 작고 강력한 `fuzzy search` 라이브러리로 다른 라이브러리 의존성이 없다는 특징이 있다. 따라서 적용이 아주 쉬운 장점이 있다. [fuse.js 데모](https://fusejs.io/demo.html) 에서 확인해보면 한글 검색도 잘 지원한다.

## fuzzy search

`fuzzy search` 란 검색의 일종으로 완벽하게 일치하는 검색이 아닌 `흐린(fuzzy)` 검색이라는 뜻으로, 쉽게 생각해서 검색어가 완벽하게 일치하지 않더라도 대상 콘텐츠를 찾을 수 있는 검색을 의미한다. 유사 검색이라고도 하는데, 이를 구현하기 위한 알고리즘은 여러가지가 있고 세부적인건 복잡하기 때문에, 아주 쉽게 예를 들어 "떡국" 이라는 단어를 찾기 위해서 "떡" 이라는 검색어로 찾을 수 있는 경우가 바로 이 `fuzzy search` 라고 이해할 수 있다.  

## hugo 에 적용하기

`fuse.js` 가 아주 간단하게 fuzzy 검색 기능을 제공하지만 hugo 에 적용하려면 몇가지 작업이 필요하다. 직접 다 만들기는 어려우니 hugo theme 로 만들어진 자료를 참고하자. [참고](https://github.com/kaushalmodi/hugo-search-fuse-js) 를 확인하여 다음과 같은 순서로 적용할 수 있다. 

### 1. index.json 템플릿을 적용
 - config.toml 의 output 에 json 타입을 추가한다. (콘텐츠를 index.json 주소로 접근 가능하게 한다.)
 - layouts/_default/index.json 에 json 으로 표시될 콘텐츠의 템플릿을 적용한다. 
   ```markdown
    [{{ range $index, $page := .Site.Pages }}
    {{- if ne $page.Type "json" -}}
    {{- if and $index (gt $index 0) -}},{{- end }}
    {
    "permalink": "{{ $page.Permalink }}",
    "title": "{{ htmlEscape $page.Title}}",
    "tags": [{{ range $tindex, $tag := $page.Params.tags }}{{ if $tindex }}, {{ end }}"{{ $tag| htmlEscape }}"{{ end }}],
    "description": "{{ htmlEscape .Description}}",
    "contents": {{$page.Plain | jsonify}}
    }
    {{- end -}}
    {{- end -}}]
    ```
### 2. 검색 랜딩 페이지 구성
 - content/search/index.md 파일을 생성하여 검색 랜딩 페이지를 구성한다. 
    ```markdown
    ---
    title: Search Result
    layout: search
    ---
    ```
 - 참고자료의 fuse.js, search.js 를 적용한다.
   ```html
   <!--검색 결과가 출력되는 부분 id="search-results" 에 추가된다.-->
    <div class="inner">
        <article class="post-full post page no-image">
            <header class="post-full-header">
                <h1 class="post-full-title">Search Result</h1>
            </header>
            <section class="post-full-content" id="search-results">
    
            </section>
        </article>
    
    </div>
    <!--검색 결과가 출력되는 템플릿 search.js 에 의해서 다음과 같은 포맷으로 표시된다.-->
    <template id="search-result-template">
        <div class="search_summary">
            <h2 class="post-title no-text-decoration"><a class="search_link search_title" href=""></a></h2>
            <p><em class="search_snippet"></em></p>
            <small>
                <table>
                    <tr class="search_iftags">
                        <td><strong>Tags</strong></td>
                        <td class="search_tags"></td>
                    </tr>
                    <tr class="search_ifcategories">
                        <td><strong>Categories</strong></td>
                        <td class="search_categories"></td>
                    </tr>
                </table>
            </small>
        </div>
    </template>
    <!--필요한 js 파일을 추가한다.-->
    {{ $assetBusting := not .Site.Params.disableAssetsBusting }}
    <script type="text/javascript" src="{{"js/libs/fuse.js/3.2.1/fuse.min.js" | relURL}}{{ if $assetBusting }}?{{ now.Unix }}{{ end }}"></script>
    <script type="text/javascript" src="{{"js/libs/mark.js/9.0.0/mark.min.js" | relURL}}{{ if $assetBusting }}?{{ now.Unix }}{{ end }}"></script>
    <script type="text/javascript" src="{{"js/search.js" | relURL}}{{ if $assetBusting }}?{{ now.Unix }}{{ end }}"></script>
    
    ```
 - css 스타일링은 적당히 변경한다.

### 3. header 영역에 검색 입력창 추가
 - theme 에서 사용하는 header 부분에 검색 입력창을 추가한다. submit 하면 추가한 검색 랜딩 페이지로 연결되도록 한다.
    ```html
    <div id="search">
        <form method="get" action="/search">
        <fieldset >
            <div class="field">
                <label><span class="hide">검색</span>
                    <i class="fa fa-search"></i>
                    <input type="text" id="searchtext" class="input_text" name="q" value="" size="15" >
                </label>
            </div>
        </fieldset>
        </form>
    </div>
    ```

### 4. 검색 기능이 잘 되는지 확인한다.

## 기타

fuse.js 말고도 다른 방법을 사용하여 hugo 사이트에 검색 기능을 추가할 수 있다. 나의 경우에는 처음에 lunr 를 도입시도해보다가
한글 지원지 잘 되지 않아서 포기하고 fuse.js 로 검색되도록 작업을 완료하였다. 다른 검색 기능을 살펴본다면 [공식 사이트 매뉴얼](https://gohugo.io/tools/search/)을 살펴보자. 

# 소감

hugo 로 변경한 뒤에 내심 검색기능을 구글에만 의존하고 있어서 자체 검색을 추가해야지 해야지 하면서 시간만 보내고 있었는데 
간단하게 작업을 완료하고 나니 한결 보기 좋아졌다는 기분이 든다. 그 동안 왜 못한다고 생각하고 있었던 걸까?;; 앞으로 방문자들에게 조금이나마 더 편하게 글을 찾을 수 있도록 도움되길 바란다. 


# 참고자료

- [hugo 공식 매뉴얼의 검색 기능 안내](https://gohugo.io/tools/search/)
- [fuse.js](https://fusejs.io)
- [hugo fuse.js 도입 참고](https://github.com/kaushalmodi/hugo-search-fuse-js)
