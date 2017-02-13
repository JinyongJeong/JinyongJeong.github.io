---
layout: post
title: 'Ubuntu Gedit 한글 깨짐 수정'
tags: [ubuntu]
description: >
  Ubuntu Gedit 한글 깨짐 수정
---
우분투 기본 에디터인 gedit에서 자동 인식되는 인코딩에 cp949 방식을 추가합니다.
먼저 콘솔에서 dconf-editor를 설치합니다. 있다면 패스합니다.

```
$ sudo apt-get install dconf-editor
$ dconf-editor
```

창이 하나 뜨면 윈도우즈 레지스트리처럼 아래 경로로 따라갑니다.
org -> gnome -> gedit -> preferences -> encodings -> auto-detected

여기에 UHC라는 값을 추가 합니다. UHC(Unified Hangul Code)는 cp949와 동일한 통합 한글 코드를 의미합니다.

```
'UTF-8', 'UHC', 'CURRENT', 'ISO-8859-15', 'EUC-KR', 'UTF-16'
```

그리고 창을 찾고 해당 파일을 열어보시면 한글이 깨끗하게 열리는 것을 볼 수 있습니다.
혹시 dconf-editor가 안될 때는 콘솔에서 gsettings로 한번에 설정값을 변경할 수 있습니다.

```
$ gsettings set org.gnome.gedit.preferences.encodings auto-detected "['UTF-8', 'UHC', 'CURRENT', 'ISO-8859-15', 'EUC-KR', 'UTF-16']"
```

그리고 값 확인은 인자를 get으로 주면 값을 확인할 수 있습니다.

```
$ gsettings get org.gnome.gedit.preferences.encodings auto-detected
```

'UTF-8', 'UHC', 'CURRENT', 'ISO-8859-15', 'EUC-KR', 'UTF-16'   <==현재 시스템에 설정된 값 확인

이렇게 하면 우분투에서 콘솔이든 UI든지 윈도우에서 작업한 한글 텍스트를 보는데 문제가 없을 것입니다.
