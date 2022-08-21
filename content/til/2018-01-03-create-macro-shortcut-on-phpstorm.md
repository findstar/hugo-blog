---
aliases:
    - /til/2018-01-03-TIL
categories:
- TIL
date: '2018-01-03'
group: til
permalink: /til/:year-:month-:day-TIL
tags:
- phpstorm
- jetbrains
- macro key mapping
title: PHPStorm 에서 매크로 단축키를 지정하는 방법
url: /2018/01/03/create-macro-shortcut-on-phpstorm
type: post
---
# 1월 03일 (수) TIL
- 참고 매뉴얼 : [Jetbrains Recording Macros](https://www.jetbrains.com/help/idea/recording-macros.html)

phpstorm의 버전을 올렸는데 지정해놓은 macro key mapping 이 이상해졌다. 지우고 새로 만들려니 macro 설정 하는 방법을 까먹어서 기억을 더듬어서 다시 한번 정리해본다.

<!--more-->

1 먼저 phpstorm 의 Edit -> Macros 를 살펴보자.

{{< imageFull src="/images/posts/til/2018-01-03/phpstorm_edit_window.png" title="phpstorm edit widdow" >}}

2 다음으로 Macros 에서 Start Macro Recording 을 선택하자.

{{< imageFull src="/images/posts/til/2018-01-03/phpstorm_start_macro_recording.png" title="phpstorm start macro recording" >}}

<!--more-->

3 그럼 아무런 표시 없이 커서만 깜빡 거릴텐데 우측 하단에 보면 Recodring 이라고 표시가 되고 있다.

{{< imageFull src="/images/posts/til/2018-01-03/phpstorm_macro_recording.png" title="phpstorm macro recording" >}}

4 이제 자신이 저장하고 싶은 Custom 한 액션들을 수행하면 된다.
내 경우에는 Option + F1 (Select In)을 누른다음, 숫자 키 '1'을 눌러서 project view로 이동하는 액션을 취했다.

이렇게 하면 원할 때 현재 편집중인 파일의 위치로 project view의 선택된 라인을 이동시킬 수 있다.
`autoscroll from source` 라는 기능으로도 동일한 needs 를 해소 할 수 있지만 내가 원하는건, 항상이 아닌 특정 액션을 수행할 때만 이었기 때문에 macro 로 만들었다.

5 그 다음에 다시 Macro 매뉴에서 Stop Macro Recording을 선택하자.

{{< imageFull src="/images/posts/til/2018-01-03/phpstorm_stop_macro_recording.png" title="phpstorm stop macro recording" >}}

6 레코딩이 진행되는 동안 수행한 액션들을 하나의 매크로로 만든다. 이제 이름을 지정하자

{{< imageFull src="/images/posts/til/2018-01-03/phpstorm_enter_macro_name.png" title="phpstorm enter macro name" >}}

7 이름까지 저장하고 나면, Edit -> Macros -> 아래쪽에 새로운 이름의 Macro 들을 확인할 수 있다.

{{< imageFull src="/images/posts/til/2018-01-03/phpstorm_macro_list.png" title="phpstorm macro list" >}}

8 이제 `Command + ,` 를 눌러 설정 창에서 keymap 을 누른 다음 macros 에 보면 저장된 매크로의 리스트를 확인할 수 있다. 여기서 내가 원하는 매크로에 키보드 단축키를 지정하면 끝!

{{< imageFull src="/images/posts/til/2018-01-03/phpstorm_custom_key_binding_on_macro.png" title="phpstorm key binding on macro" >}}

이제 마음껏 내가 원하는 대로 조합한 매크로를 단축키로 사용하면 된다.
