---
date: 2021-08-16T18:40:01+09:00
group: blog
image: /images/hugo-logo.png
tags: ["hugo", "rss template"]
title: "Hugo에서 RSS 템플릿을 지정하는 방법"
url: /2021/08/16/hugo-rss-template
type: post
summary: "그 동안 작성된 포스트를 rss 링크에서 확인한 결과 태그가 정상적으로 노출되지 않고 있다는 것을 발견하였다. 그래서 hugo 에서 지원하는 RSS 템플릿 기능을 사용해서 RSS 에서 포스트의 태그를 확인할 수 있게 작업을 진행해보았다."
---

# 개요

지난주에 커뮤니티에서 친분이 있는 분이 이 블로그의 RSS 링크에서 포스트의 태그가 확인되지 않는다고 제보해 주셨다. 확인 결과 rss 링크 "https://findstar.pe.kr/index.xml" 에서 포스트에 등록한 태그가 노출되지 않고 있었다. 그래서 RSS 스펙을 확인하여 hugo 템플릿을 수정해서 해결하였다.

## 템플릿 파일 생성하기

[매뉴얼](https://gohugo.io/templates/rss/#the-embedded-rss-xml) 을 확인하면 먼저 RSS 템플릿의 파일 매칭 순서를 확인할 수 있다.
내가 사용하는 테마파일에서는 별도의 RSS 템플릿을 지정하지 않았기 때문에 내장된 기본값이 사용된다. 기본값은 [링크](https://github.com/gohugoio/hugo/blob/master/tpl/tplimpl/embedded/templates/_default/rss.xml)에서 내용을 확인할 수 있다.

내장된 기본값은 다음과 같다. 

```xml
{{- $pctx := . -}}
{{- if .IsHome -}}{{ $pctx = .Site }}{{- end -}}
{{- $pages := slice -}}
{{- if or $.IsHome $.IsSection -}}
{{- $pages = $pctx.RegularPages -}}
{{- else -}}
{{- $pages = $pctx.Pages -}}
{{- end -}}
{{- $limit := .Site.Config.Services.RSS.Limit -}}
{{- if ge $limit 1 -}}
{{- $pages = $pages | first $limit -}}
{{- end -}}
{{- printf "<?xml version=\"1.0\" encoding=\"utf-8\" standalone=\"yes\"?>" | safeHTML }}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ if eq  .Title  .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{.}} on {{ end }}{{ .Site.Title }}{{ end }}</title>
    <link>{{ .Permalink }}</link>
    <description>Recent content {{ if ne  .Title  .Site.Title }}{{ with .Title }}in {{.}} {{ end }}{{ end }}on {{ .Site.Title }}</description>
    <generator>Hugo -- gohugo.io</generator>{{ with .Site.LanguageCode }}
    <language>{{.}}</language>{{end}}{{ with .Site.Author.email }}
    <managingEditor>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</managingEditor>{{end}}{{ with .Site.Author.email }}
    <webMaster>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</webMaster>{{end}}{{ with .Site.Copyright }}
    <copyright>{{.}}</copyright>{{end}}{{ if not .Date.IsZero }}
    <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
    {{- with .OutputFormats.Get "RSS" -}}
    {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
    {{- end -}}
    {{ range $pages }}
    <item>
      <title>{{ .Title }}</title>
      <link>{{ .Permalink }}</link>
      <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
      {{ with .Site.Author.email }}<author>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</author>{{end}}
      <guid>{{ .Permalink }}</guid>
      <description>{{ .Summary | html }}</description>
    </item>
    {{ end }}
  </channel>
</rss>
```

이제 나의 고유한 RSS 템플릿 파일을 작성해보자. 위의 내장된 기본값을 바탕으로 약간의 수정된 내용을 추가하면 된다. 나의 경우에는
`layouts/_default/index.xml` 파일을 생성하였다. 만약 값이 적용이 안된다면 [매뉴얼](https://gohugo.io/templates/rss/#the-embedded-rss-xml)에서 매칭 순서를 다시 확인해보자. 우선순위가 높은 파일이 이미 존재할 수도 있다. 

생성한 파일은 다음과 같이 `<category>` 태그를 추가하였다. 포스트에서는 `TAG` 이지만, 이를 표한혀는 [RSS 스펙](https://validator.w3.org/feed/docs/rss2.html)이 `Category` 이기 때문이다. 이를 `Taxonomy` 라고 한다. 

```xml
{{- $pctx := . -}}
{{- if .IsHome -}}{{ $pctx = .Site }}{{- end -}}
{{- $pages := slice -}}
{{- if or $.IsHome $.IsSection -}}
{{- $pages = $pctx.RegularPages -}}
{{- else -}}
{{- $pages = $pctx.Pages -}}
{{- end -}}
{{- $limit := .Site.Config.Services.RSS.Limit -}}
{{- if ge $limit 1 -}}
{{- $pages = $pages | first $limit -}}
{{- end -}}
{{- printf "<?xml version=\"1.0\" encoding=\"utf-8\" standalone=\"yes\"?>" | safeHTML }}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
    <channel>
        <title>{{ if eq  .Title  .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{.}} on {{ end }}{{ .Site.Title }}{{ end }}</title>
        <link>{{ .Permalink }}</link>
        <description>Recent content {{ if ne  .Title  .Site.Title }}{{ with .Title }}in {{.}} {{ end }}{{ end }}on {{ .Site.Title }}</description>
        <generator>Hugo -- gohugo.io</generator>{{ with .Site.LanguageCode }}
        <language>{{.}}</language>{{end}}{{ with .Site.Author.email }}
        <managingEditor>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</managingEditor>{{end}}{{ with .Site.Author.email }}
        <webMaster>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</webMaster>{{end}}{{ with .Site.Copyright }}
        <copyright>{{.}}</copyright>{{end}}{{ if not .Date.IsZero }}
        <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
        {{- with .OutputFormats.Get "RSS" -}}
        {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
        {{- end -}}
        {{ range $pages }}
        <item>
            <title>{{ .Title }}</title>
            <link>{{ .Permalink }}</link>
            <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
            {{ with .Site.Author.email }}<author>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</author>{{end}}
            <guid>{{ .Permalink }}</guid>
            <description>{{ .Summary | html }}</description>
            {{ range .Params.tags }}
            <category>{{ . }}</category>
            {{ end }}
        </item>
        {{ end }}
    </channel>
</rss>
```

실제 기본 템플릿 내용에서 추가한 부분은 다음과 같다. 

```xml
<item>
    ..생략..
    {{ range .Params.tags }}
    <category>{{ . }}</category>
    {{ end }}
</item>
```

템플릿 파일에서는 `.Params` 값을 사용할 수 있다. 템플릿에서 사용할 수 있는 값들은 [매뉴얼](https://gohugo.io/variables/page/)을 참고하자. 
추가한 내용을 해석하면 포스트의 `tags` 라는 값을 확인해서 `category` 라는 RSS 태그를 추가한 것이다. 따라서 `category` 는 여러개가 될 수 있다.
물론 위의 RSS 태그가 표시되려면 개별 포스트는 tags 라는 속성값이 존재해야한다. 

```markdown
...
tags: ["hugo", "rss template"]
...
```

## 결과 확인

이제 rss 결과를 확인하자. 

```xml
<item>
<title>Spring 프로젝트 Maven을 사용할 때 도커라이즈 캐싱방법</title>
<link>https://findstar.pe.kr/2021/08/06/dockerize-maven-project/</link>
<pubDate>Fri, 06 Aug 2021 20:08:09 +0900</pubDate>
<guid>https://findstar.pe.kr/2021/08/06/dockerize-maven-project/</guid>
<description>spring 프로젝트에서 maven 을 사용할 때 도커라이즈에서 레이어를 캐싱하여 빌드 속도를 향상시키는 방법을 살펴보았다.</description>
<category>dockerize</category>
<category>spring boot</category>
<category>maven</category>
</item>
```

### 참고 
- https://gohugo.io/templates/rss/#the-embedded-rss-xml
- https://gohugo.io/variables/page/
- https://validator.w3.org/feed/docs/rss2.html
