---
layout: post
title: 'Ubuntu and Window Keyboard&Mouse sharing(Synergy)'
tags: [ubuntu]
description: >
  Ubuntu and Window Keyboard&Mouse sharing
---
Synergy는 두 컴퓨터를 인터넷망으로 연결하여 Input device를 공유하도록 하는 프로그램이다. 이 프로그램의 장점은 윈도우와 리눅스에서 동시에 사용 가능하다는 점이다. 즉 데스크탑 한대(리눅스)와 노트북(윈도우)를 한개의 Input device로 control이 가능하다. 즉, 윈도우로 사용 가능한 여유분의 노트북이 있다면 리눅스에 kakao talk을 설치하려고 삽질하는 노력을 하지 않아도 된다는 뜻이다.

현재 Synergy는 유료프로그램으로 전환 되었으나 1.4.18버전을 사용한다면(구버전) 무료로 사용 가능하다. 최신 무료버전을 다운받고 싶으면 [여기](http://storycompiler.tistory.com/37)의 포스팅을 참고한다. 참고로 뒤에 얘기하지만 포스팅을 쓰고 있는 현 시점에 Synergy 1.4.18버전은 한글입력 문제를 위한 패치를 1.4.10버전에 비해 구하기 쉽지 않다.

현재 구성되어 있는 시스템은 리눅스(서버), 윈도우(클라이언트)인데 접속은 잘 되지만 윈도우에서 한글이 안되는 문제가 발생한다. 이 문제는 서버의 문제가 아닌 클라이언트 쪽의 문제이다. 아무리 찾아도 1.4.18버전의 한글 패치 버전을 찾기 힘들다. 직접 빌드를 해도 되지만 한글 패치가 있는 버전으로 내려서 설치하였다. 찾아보니 1.4.10버전의 한글패치가 가장 많은 듯 하여 1.4.10버전으로 설치하고 패치까지 완료하니 정상작동된다. 한글패치 파일을 클라이언트쪽(윈도우)의 경로에 붙여넣어준다. 보통 C:\Program Files\Synergy 이다. 자세한 내용은 [다음](http://blog.shgeum.com/entry/Synergy-1410-x86-x64-%ED%95%9C%EA%B8%80-%ED%8C%A8%EC%B9%98) 포스팅을 참고한다. 이 글을 보는 분들을 위해 synergy 1.4.10 window(64bit), ubuntu(64bit), 그리고 한글 패치파일 총 3개 파일 첨부한다.리눅스 파일을 설치할 때 오류가 뜨는데 무시하고 설치하니 잘 동작한다. [여기](/Download/Synergy/1.4.10.zip)에서 파일을 다운 받을 수 있다.
