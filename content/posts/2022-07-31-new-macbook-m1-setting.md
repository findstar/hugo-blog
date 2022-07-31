---
date: '2022-07-31 14:01:03 +09:00'
group: blog
image: /images/posts/apple/M1-apple-chip.jpg
tags: ["m1 macbook pro"]
title: "새로운 M1 macbook 장비 개인 셋팅"
url: /2022/07/31/new-macbook-pro-max-configuration
type: post
summary: "기다리고 기다리던 새로운 맥북 PRO M1 MAX를 받았다. 반도체 대란(?!)의 영향으로 6개월을 기다린 끝에 새로 받은 장비였다. 새로 받은 맥북의 환경설정을 정리해보았다. "
---

# 새로운 맥북 "M1 Max"

6개월을 기다린 끝에 새로운 맥북을 받았다. 작년 출시 이후에 12월에 MAX로 장비를 신청했는데, 반도체 대란의 영향인지(?!) 대기가 너무 길어져서 결국 7월이 되어서야 새로운 맥북을 받을 수 있었다.
M1 프로세서의 좋은점은 귀가 따갑게 들어왔던지라 "역시 조용하고 쾌적하군" 이라며 행복해 하고 있는데, 그런 감상을 떠나서 새로운 장비가 왔으면 새로운 셋팅을 해야되지 않겠냐며 팔을 걷어부치고 새장비 셋팅에 나섰다.
이전에 사용하던 맥북이 2017년형이라 맥북 환경 설정을 한지는 너무 오래되어서, 새로운 마음으로 설정하면서 내용들을 정리해보았다.

# 개인취향의 셋팅
참고로 아래의 애플리케이션과 환경 셋팅은 지극히 개인적인 내용이라, 다른 분들이 적용하기에는 호불호가 있을 수 있음을 먼저 알린다.

## 키보드를 설정해보자. 

1. **보조키 설정 변경** : 새로운 맥북에서 기본셋팅된 키 설정이 내가 사용하던 설정과 달라서 보조키 설정을 바꿔주었다. (나는 "한/A(이전의 CapsLock)" 키를 "Command" 키로 사용한다.)
2. **펑션키 기능 옵션 변경** : F1~F12 까지의 키를 바로 사용할 수 있도록 체크박스를 활성화 하였다. 
{{< imageFull src="/images/posts/new-macbook/system-keyboard.png" title="system-keyboard" border="false" caption="시스템 환경설정 / 키보드">}}
{{< imageFull src="/images/posts/new-macbook/keyboard-subkey.png" title="keyboard-subkeys" border="false" caption="키보드 / 보조키">}}
{{< imageFull src="/images/posts/new-macbook/keys.png" title="keys" border="false" caption="보조키">}}
3. **키보드 탐색 컨트롤 포커스 활성화** : 이 옵션을 켜두면 Confirm 또는 Alert 화면에서 탭으로 포커스를 이동할 수 있다.
{{< imageFull src="/images/posts/new-macbook/keyboard-focus-options.png" title="focus option" border="false" caption="키보드 탐색 초첨 이동 활성화">}}

## 애플리케이션을 설치해보자.

이제 자주 사용하는 앱들을 설치해본다.

1. **[Fantastical](https://flexibits.com/fantastical)** : 유료 캘린더 앱이다. 월 구독료 6500원을 내고 있지만, 개인과 회사 일정을 관리하는데 이만한게 없다.
   {{< imageFull src="/images/posts/new-macbook/fantastical.png" title="fantastical" border="false" caption="fantastical : 구독료 월 6500원">}}
2. **커뮤니케이션 툴들** : 커뮤니케이션용으로 사용하는 툴들은 제법 많은데 리스트만 나열해보았다.
   - 슬랙 : 대체 몇개의 슬랙 채널을 보고 있는 걸까, 새로 설치한 김에 몇개는 로그아웃 해버렸다.
   - 카카오톡 : 가족과의 대화에 빠질 수 없는 앱
   - 카카오 워크 : 협업하는 몇몇 파트너사들에서 사용하고 있어서 설치했다. 
   - 디스코드 : 이건 활동하는 몇몇 커뮤니티에서 사용하고 있어서 설치했다.
3. **IDE & Editing** : 개발하는데 필요한 IDEA와 코드 에디팅 프로그램이다. 
   - [Jetbrains ToolBox](https://www.jetbrains.com/ko-kr/lp/toolbox/) : 이전 맥북에서는 Jetbrains의 제품을 All Product 라이선스를 구독하고 있는 데다가, 하다보니 "IntelliJ", "DataGrip", "PyCharm", "GoLand", "PhpStorm", "Clion"까지 사용했었다. 
      새로 설치하면서 "IntelliJ"와 "DataGrip", "Goland"만 남겨놓았다.
   - [vscode](https://code.visualstudio.com/) : 나의 경우에는 간단한 markdown 메모용으로 많이 쓰지만(?!) 가벼워서 애용하고 있다. extension 은 [Jetbrains Key Map](https://marketplace.visualstudio.com/items?itemName=isudox.vscode-jetbrains-keybindings)만 설치해서 사용한다.
     (난 Jetbrains이 좋더라, Jetbrains에서 가벼운 텍스트 에디터 내놓으면 이것도 사고 싶다.)
4. **Note 앱** : 이전에는 Notion, Obsidan, [Craft](https://www.craft.do/) 를 사용했지만, 지금은 Craft 만 남겨놓았다. 생각해보니 이것도 유료 구독제다.
    {{< imageFull src="/images/posts/new-macbook/craft.png" title="craft" border="false" caption="craft : 구독료 월 6900원">}}
5. **브라우저** : 브라우저를 깜빡했다. Google Chrome 과 Google Chrome Beta 를 설치하였고 다음은 몇가지 추가한 extension이다.
   - Google Chrome 은 개인용 , Google Chrome Beta 는 회사용이다. 계정을 나눠서 관리하고 있다.
   - [Adblock 확장](https://chrome.google.com/webstore/detail/adblock-%E2%80%94-best-ad-blocker/gighmmpiobklfepjocnamgkkbiglidom) : 미안하지만 광고가 너무 많다.  
   - [유튜브용 Adblock](https://chrome.google.com/webstore/detail/adblock-for-youtube/cmedhionkhpnakcndndgjdbohmhepckk) : 여전히 광고는 좀 덜 봤으면.
   - [Classic Cache Killer](https://chrome.google.com/webstore/detail/classic-cache-killer/kkmknnnjliniefekpicbaaobdnjjikfp) : 웹 페이지 테스트 할 때 캐시를 없애주는 확장기능
   - [Json Formatter](https://chrome.google.com/webstore/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa) : Json 응답을 예쁘게 보여주는 확장 기능
     {{< imageFull src="/images/posts/new-macbook/json_formatter.jpg" title="json formatter" border="false" caption="json formatter chrome extension">}}
6. **암호 관리 프로그램 [Enpass](https://www.enpass.io/)** : 암호 관리용 프로그램으로 Enpass를 사용하고 있다. 구독제로 변경되기전 막차를 타서 Pro 를 한번 구매한 뒤에 평생 사용할 수 있는 권한을 얻었다.
   - [chrome extension](https://chrome.google.com/webstore/detail/enpass-password-manager/kmcfomidfpdkfieipokbalgegidffkal) 도 지원해서 연결해서 편하게 사용하고 있다. 게다가 암호 저장 Vault 를 개인용 아이클라우드에 저장할 수 있어서 더 맘에 든다.
   {{< imageFull src="/images/posts/new-macbook/enpass.png" title="enpass" border="false" caption="enpass">}}
7. **Rectangle** : 애플리케이션의 윈도우 사이즈를 키보드 단축키로 편하게 이동시킬 수 있는 앱이다. 이전에는 [Spectacle](https://www.spectacleapp.com/)을 사용하고 있었는데 더이상 업데이트가 되지 않고 있어서 이번에 변경했다. Rectangle 기본 설정 방식이 Spectacle 과 유사해서 큰 부담없이 적응할 수 있었다.
   {{< imageFull src="/images/posts/new-macbook/rectangle.gif" title="rectangle" border="false" caption="rectangle">}}
8. **터미널 [Iterm2](https://iterm2.com/)** : 하루에도 수십번씩 열어보는 터미널앱
   {{< imageFull src="/images/posts/new-macbook/iterm2.png" title="iterm2" border="false" caption="iterm2">}}
9. **[Alfred 4](https://www.alfredapp.com/)** : 기본 내장된 스팟라이트 대신 사용하는 Alfred 4 이다. 새로 설치하려고 보니 Alfred 5 가 나왔는데 그냥 이전 버전 파워팩 라이선스를 가지고 있어서 4 버전으로 설치했다.
   {{< imageFull src="/images/posts/new-macbook/alfred4.png" title="alfred4" border="false" caption="alfred4">}}
10. **[Paw](https://paw.cloud/)** : API 테스팅을 도와주는 도구로 PAW를 사용하고 있다. 다른 분들은 [Postman](https://www.postman.com/)을 많이 쓰던데 나는 그냥 개인 취향으로 이걸 쓰고 있다.
    {{< imageFull src="/images/posts/new-macbook/paw.png" title="paw" border="false" caption="paw">}}
11. **[eul](https://github.com/gao-sun/eul)** : 맥북의 시스템 자원 모니터링 툴이다. 에전에는 stats를 쓰다가 지금은 eul을 쓰고 있다. 이런 앱은 워낙 종류가 많지만 한글이 잘 나와서 애용하고 있다.
    {{< imageFull src="/images/posts/new-macbook/eul.jpg" title="eul" border="false" caption="홈페이지 스크린샷이라 한글이 없지만 한글 메뉴 잘 나온다.">}}
12. **[Stem](https://store.steampowered.com/)** : 이건 맥이라도 거부할 수가 없다.
    {{< imageFull src="/images/posts/new-macbook/steam.jpg" title="steam" border="false" caption="맥북이라도 스팀은 피할 수 없다.">}}

## 터미널 환경 셋팅
필요한 애플리케이션들을 설치했으니 이제 터미널 환경을 설치할 순서다. 

1. **[homebrew](https://brew.sh/index_ko) 설치** : 처음에는 역시 Homebrew 다.
    ```shell
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
2. **[oh-my-zsh](https://ohmyz.sh/) 설치** : 테마를 바꾸고 싶어서 oh-my-zsh를 설치했다. 
    ```shell
    sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" 
    ```
3. **[Cobalt2 Theme](https://github.com/wesbos/Cobalt2-iterm) 설정** : 내가 좋아하는 Cobalt2 테마를 설정했다.
   - github 에서 `cobalt2.zsh-theme` 파일들을 다운받아 `~/.oh-my-zsh/themes/` 디렉토리에 넣어주고
   - `~/.zshrc` 파일의 테마 부분을 `ZSH_THEME=cobalt2` 와 같이 변경해준다.
   - 일부 디렉토리에서 Git branch 를 예쁘게 보여주기 위한 폰트가 필요하다. powerline 폰트를 설치한다.
   ```shell
    git clone https://github.com/powerline/fonts
    cd fonts
    ./install.sh
    ```
   - iterm 설정을 변경해준다.
   {{< imageFull src="/images/posts/new-macbook/iterm_preferences.png" title="item preferences profile text" border="false" caption="profiles > text">}}
   {{< imageFull src="/images/posts/new-macbook/iterm_color.png" title="item preferences profile color" border="false" caption="profiles > color 에서 cobalt2.itermcolors 파일을 import 한다.">}}

## IDE 환경 설정

이제 가볍게 IntelliJ 환경 설정을 진행해보았다.

1. **폰트 변경** : 이제는 좀 큰 글씨가 좋다. 어쩔 수 없는 노화의 영향인가.
   {{< imageFull src="/images/posts/new-macbook/intellij_font.png" title="intelliJ font" border="false" caption="사이즈 15라니... ㅠㅠ">}}
2. **힙 메모리 사용량 증가** : IntelliJ 가 사용하는 힙 메모리 량을 늘려주자.
   {{< imageFull src="/images/posts/new-macbook/intellij_custom_vm_option.png" title="intelliJ custom vm option" border="false" caption="vm options">}}
   - 다른 옵션을 더 추가할 수 있지만 일단 Max Heap Memory Size 를 4G로 늘려줬다. (이 기능은 Jetbrains ToolBox 에서도 설정 가능하다) 
   ```
    -Xms2048m
    -Xmx4096m
   ```
3. **개인적으로 사용하는 단축키 등록** : 프로젝트 뷰의 포커스를 현재 보고 있는 파일의 위치로 이동시키는 단축키가 없어서 커스컴하게 매크로로 만들어서 사용하고 있다. 참고 : [매크로 생성방법](https://findstar.pe.kr/2018/01/03/create-macro-shortcut-on-phpstorm/)
   {{< imageFull src="/images/posts/new-macbook/select_in_project_macro.png" title="macro shortcut" border="false" caption="직접 만들어서 쓰는 단축키">}}


# 기타

## 트러블 슈팅

1. 처음에 Jetbrains Toolbox 를 사용해서 IntelliJ 를 설치했는데 왠지 모르게 기대와 다른 느림의 미학을 보여주고 있었다. 순간 향로님의 예전글 [M1 맥북 개발환경 세팅](https://jojoldu.tistory.com/571) 을 본 기억이 있어서
   IntelliJ 와 Goland 는 수동 설치했다. ㅠㅠ ToolBox 는 라이선스 관리용도로만 활용되는 현실.
2. 키보드 한영 변경을 "Shift + Space"를 사용하고 있었는데 불가능해졌다. [Andrew Note - macOS Monterey 업그레이드 후 한영 변환키 설정 [ shift + space key ]](https://andrewpage.tistory.com/95)를 참고해서 plist 를 변경해주었다.

## 라이선스와 계정 연결
새로운 애플리케이션을 설치하였더니 새로 실행할 때마다 계정연동 및 라이선스 확인을 새로 해주어야 했다. 일부는 자동으로 소셜로그인을 통해서 해결이 되었지만 일부는 이메일을 뒤져서 라이선스 키를 찾아서 등록해주었다. 

## 남은 이야기
사실 새로운 맥북을 구동할 때 이전 맥북에서 마이그레이션 옵션을 사용하면 이런 수고로움을 덜 수도 있었겠지만, 왠지 몇년 만에 바꾼 장비다 보니 새롭게 Fresh 한 느낌을 가져보고 싶어서 하나하나 설치해보았다. 
게다가 M1 으로 바뀐 뒤에 얼마나 속도가 빨라졌을지 기대하면서 설정 작업들을 진행했는데, 확실히 이전 머신보다 빠르고 조용하고 쾌적했다. (I Love Apple Silicon) 당분간은 새로운 머신에 적응하면서 또 자잘한 설정들을 계속 해나가겠지만, 
큰 작업들은 마무리 되었고, 이 참에 새롭게 개발환경을 다듬어 보고 싶어서 위의 작업들만 셋팅하고 나머지는 차차 진행해보기로 마음 먹었다. 여기까지 진행하고 나니, 회사 동료들은 어떤 앱들과 환경을 셋팅해서 사용하실지 궁금해져서
동료들에게 물어보고 싶어졌다. 다음 티타임에 질문을 해보아야겠다. 그럼 이제 스팀을 실행하고 게임을 해봐야겠다. ^^; 


