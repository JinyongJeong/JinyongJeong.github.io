---
layout: post
title: 'Ubuntu 한글 키보드 설정'
tags: [ubuntu]
description: >
  Ubuntu 에서 한글 키보드를 사용하기 위한 설정방법을 설명합니다.
---
<본 포스팅은 [다음](http://blog.daum.net/bagjunggyu/154)을 참고하였습니다>

[여기](http://blog.daum.net/bagjunggyu/154)에서 한글 설정 및 다양한 Ubuntu 설정에 대한 내용을 찾을 수 있다. 

# Ubuntu 14.04 한글 설정(uim)

uim은 지금까지 알려진 한글 입력기 중에서 쓰기 가장 무난한 입력기입니다. 다만, Qt5 어플 (포커스 라이터 1.5 -  Focuswriter 1.5 등)에서 한글 입력이 제대로 되지 않습니다.

## 1. 설치하기

터미널에서 설치하실 분들은 아래 명령어를 이용하세요

```
sudo apt install uim uim-byeoru 
```

소프트웨어 센터를 이용하실 분들은  uim 이라고 검색하시면 나옵니다. 
설치하신 후에, 이번엔 uim-byeoru를 검색하셔서 설치합니다.

## 2. 구성키 설정 하기

노트북을 사용하는 경우 구성키 설정으르 추가로 해줘야 한다. 그렇지 않으면 한/영 키가 왼쪽 alt키로 인식되기 때문이다. Desktop 사용자는 두번째 단계를 건너서 세번째로 바로 가셔도 좋습니다.

노트북사용자는 시스템설정(system setting)-키보드(keyboard)-바로가기(shortcuts) 로 들어갑니다.

첫번째로 Launchers(가장 위) 메뉴에서 `key to show the HUD` 키를 `Alt L` 로 변경해 줍니다. 

원래 `Alt L` 로 되어 있는 경우는 넘어갑니다. 

두번쨰로 Typing(자판입력) 탭에서 Compose Key(구성키)를 `Right Alt`로 변경해 줍니다.

## 3. uim 설정

uim 이라고 검색하셔서 설정 도구를 실행하세요

* Global setting(전체적인 설정)에서 Specify default IM(디폴트 입력기)를 체크 후 Byeoru(벼루)로 설정합니다.
* 그리고, Byeoru key bindings1(벼루키 설정 1)에서 on/off를 원하시는 키를 지정합니다. 보통 한글 변환이기 때문에 한/영 키로 설정합니다. `Edit..` 를 눌러 원하는 키를 `Grab..` 하신 후 추가해 줍니다.
* system setting(시스템 설정) - Language Support(언어 지원)을 실행해서, 키보드 입력기를 uim으로 설정한 후  로그아웃 & 로그인 하시면 됩니다.


