---
aliases:
    - /til/2018-01-05-TIL
categories:
- TIL
date: '2018-01-05'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- phpstorm
- speed up
- vm options
title: 느려진 PHPStorm에서 Heap Memory를 늘리는 방법
url: /2018/01/05/increse-heap-memory-on-phpstorm
type: post
---


# 1월 05일 (금) TIL

이전에, phpstorm에서 사용하지 않는 플러그인을 비활성화 시켜서 약간의 속도 향상을 가져왔다면, 이제는 아예 Heap Memory Size를 늘려보기로 했다.
IDE를 사용하다가 보면 열어둔 Tab이 많아지면서 슬슬 Heap memory size가 차기 시작하는데, 이건 예전에 Eclipse를 쓸 때 부터, Intellij, Webstorm, Phpstorm 가리지 않고 나타나는 증상이다.
커서가 렉 걸린 것 처럼 느리게 이동하기 시작하면, 우측 하단에 Memory Indicator를 바라보고는, 여기서 매번 Heap Memory Size 를 확인하고 클릭해주면서 한번씩 정리가 되는데, 그러고 나면 다시 괜찮아지고는 했다.
아예 Heap Memory Size 설정을 변경하기 위해서 설정을 변경해봤다.

<!--more-->

먼저 heap memory 를 정확하게 확인하기 위해서 memory indicator를 확인해보자.

1  먼저 설정의 appearance 에서 window option의 show memory indicato를 켜자.

{% include image_caption.html imageurl="/images/posts/til/2018-01-05/phpstorm_memory_indicator.png" title="phpstorm show memory indicator" %}


2  그러면 에디터 우측 하단에 요렇게 메모리가 표시된다

{% include image_caption.html imageurl="/images/posts/til/2018-01-05/phpstorm_show_memory.png" title="phpstorm show memory" %}


3  그런다음의 help 메뉴의 Edit Custom Vm Option을 선택하자

{% include image_caption.html imageurl="/images/posts/til/2018-01-05/phpstorm_help_menu.png" title="phpstorm help menu" %}


4  이제 Custom Option을 지정하면 되는데 핵심은 `Xmx` 부분인데 maximum memory 이다.
이걸 머신의 ram 에 따라서 늘려주자 내가 쓰는 맥북의 RAM은 16G 인데 Xmx 를 처음에는 500M 에서 2G 로 변경해줬다.

{% include image_caption.html imageurl="/images/posts/til/2018-01-05/phpstorm_custom_vm_option.png" title="phpstorm custom vm option" %}

이제 IDE를 재시작하면 늘어난 memory 를 indicator 에서 확인가능하고, 버벅거림이 훨~~~씬 줄어들었다.