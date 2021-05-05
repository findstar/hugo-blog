---
date: '2021-04-21 22:43:03 +09:00'
group: blog
image: /images/posts/confluence/confluence-logo.png
tags:
- "confluence"
- "sidebar search"
- "customizing"
title: "컨플루언스 사이드바에 검색 기능 추가하기"
url: /2021/04/21/confluence-sidebar-search
type: post
summary: "위키(컨플루언스)에 새로운 스페이스를 생성하면서, 사이드바에 검색 기능을 추가하는 방법을 살펴보았다."
---
# 컨플루언스 사이드바에 검색 기능 추가하기

## 배경

신규 프로젝트를 진행하면서 위키로 사용중인 컨플루언스에 새로운 스페이스를 생성하게 되었다. 
동료들의 권한설정을 마치고 몇가지 컨벤션을 정한 다음 스페이스의 레이아웃을 살펴보았다. 
뭔가 허전함을 느껴서 그게 뭘까 하고 생각하던 중에, 이전 스페이스에서 사용하던 사이드바 검색 기능이 빠져있는 것을 발견하였다. 
막상 설정하려다 보니, 금방 할 줄 알았는데 한참이나 헤메게 되어서 나중에 까먹지 않으려고 정리해보았다. 

### 사이드바 검색이 필요한 이유

컨플루언스 위키에는 기본적으로 상단 헤더영역에 검색기능이 있지만, 여기서 제공하는 검색은 `전체 통합 검색`이다. 
그래서 스페이스가 많을 때는 특정 스페이스만 지정하여 검색하려면 불편하다. 따라서 사이드바에 검색기능을 추가하면
해당 스페이스의 문서만 검색이 쉬워지므로 사용성이 좋아진다.

## 설정 방법

1. 먼저 스페이스에 진입후 왼쪽 하단의 `공간 도구`를 눌러 `레이아웃(모양새)`를 선택하자  
{{< imageFull src="/images/posts/confluence/sidebar-search-in-confluence1.png" title="sidebar-search-in-confluence1" border="false" >}}

2. 다음으로 `사이드바` 영역을 선택하자 
{{< imageFull src="/images/posts/confluence/sidebar-search-in-confluence2.png" title="sidebar-search-in-confluence2" border="false" >}}

3. 사이드바 설정에 다음과 같이 매크로를 등록하자
{{< imageFull src="/images/posts/confluence/sidebar-search-in-confluence3.png" title="sidebar-search-in-confluence3" border="false" >}}

4. 끝이다. 이제 사이드바에 검색 기능이 잘 나오는지 확인한다.
{{< imageFull src="/images/posts/confluence/sidebar-search-in-confluence4.png" title="sidebar-search-in-confluence3" border="false" >}}

## 참고

https://community.atlassian.com/t5/Confluence-questions/Sidebar-Search-in-a-confluence-Space/qaq-p/888718



