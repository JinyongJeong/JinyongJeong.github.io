---
layout: post
title: '[MAC OS]MAC OS keyboard customize (맥 OS 키보드 커스텀, 한영전환 쉽게하기)'
tags: [software]
description: >
  MAC OS keyboard customize (맥 OS 키보드 커스텀, 한영전환 쉽게하기)
---

맥북의 OS인 MAC OS는 한영 전환이 매우 귀찮다. 

보통 command + space 를 이용해서 한영전환을 한다. 

기존에 윈도우나 리눅스를 사용하던 사람은 한영전환 키가 없어서 매우 불편하다. 

이러한 단점을 해결하기 위해서 `karabiner`라는 프로그램을 사용하면 좋다. 

## 1. karabiner 설치

우선 [https://pqrs.org/osx/karabiner/](https://pqrs.org/osx/karabiner/)에 접속해서 설치파일을 다운받아서 설치한다. 

## 2. 한영전환 키 설정

맨 오른쪽 탭인 `simple modifications`설정에서 Add item을 눌러서 키를 추가해준다.

from key를 right_command로 to key를 caps_lock으로 설정한다. 

왜  caps_lock으로 설정하면 한영전환이 되냐하면

기본 키보드 설정에서 `입력소스` 탭의 아래쪽을 보면

`Caps Lock 키로 U.S 입력소스 전환`  이라는 옵션이 있다. 이 옵션이 켜져있을 때 caps lock키로 한영 전환이 되는데

이 키를 오른쪽 command 키로 맵핑하는 것이다. 실제 caps lock 키를 누르고 있을 때는 길게 눌러주면 된다. 

추가적으로 설정한 키는 `fn`키를 `left_command`로 변경하고 `right_option`키를 `fn`키로 변경하였다.

윈도우와 리눅스에서 cnt + c,v 가 Mac OS에서는 command + c,v이기 때문에 비슷하게 사용하기 위해서 이렇게 설정하였는데

이것은 본인이 원하는 대로 설정하면 된다.

## 3. 기타 원하는 키 추가

개인적으로 리눅스를 쓸 때 터미널을 많이 열기 때문에 ctl +  alt + T 버튼으로 터미널을 여는 것에 익숙해져 있는데

Mac OS에는 이러한 기능이 없어서 매우 불편했다.

이런 부분을 해결하기 위해서 Complex  modifications 탭에서 이러한 기능을 추가할 수 있다. 

아래쪽에 `Add rule` 을 누르고  위쪽에 `import more rules from the internet` 을 눌러서 기본 정의되어 있는 룰을 가져올 수 있다.

그러면 여러 룰을 검색할 수 있는 사이트로 연결이 되는데 검색창에 `launch`를 검색하면

`Launch apps`를 찾을 수 있고 `import`버튼을 통해 추가 가능하다.

이 룰은 오른쪽 command 버튼과 T를 누르면 터미널을, F를 누르면 finder를 킬 수 있으며 몇가지 기능이 더 있다. 

이거 뿐만 아니라 다양한 룰들이 있으니 원하는  것을 검색해서 추가해보자.

