---
layout: post
title: 'Static 변수와 Extern변수'
tags: [software]
description: >
  Static 변수와 Extern변수
---
<본 포스팅은 [다음](http://thrillfighter.tistory.com/255)을 참고하였습니다>


Static과 Extern의 개념 정리에 도움이 될듯 하다.

* a.h 라는 헤더파일이 main.cpp와 a.cpp에서 include 되어 있고 a.h에 int a=3; 이라는 변수가 전역으로 선언 되어 있을 경우 object file을 잘 생성이 되나 링크시 에러가 발생하며, a라는 전역변수가 프로그램 내에서 두번 중복되기 때문이다. 이를 방지하기 위해서는 전역변수를 지역변수로 만드는 것이다.(함수외부에 변수를 선언하면 전역적 특징을 갖게 된다.)

* 함수 선언에 static이 붙어 static int a=3; 이 되면 main.cpp와 a.cpp는 같은 a.h를 include하지만 static 이기 때문에 각 cpp파일에서 사용하는 a는 각각의 지역변수가 되어 따로 존재하게 된다.

* 각 cpp파일에서 동일한 a를 사용할 목적이라면?
-> 각 cpp파일에 int a = 3; 이라고 선언 후 헤더(h)파일에 extern int a; 라고 선언하면 된다.

* 자세한 내용은 [다음](http://thrillfighter.tistory.com/255)의 포스팅을 참고하자.
