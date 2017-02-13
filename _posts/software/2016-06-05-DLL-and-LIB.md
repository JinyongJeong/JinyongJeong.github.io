---
layout: post
title: 'DLL파일과 LIB파일 차이'
tags: [software]
description: >
  DLL파일과 LIB파일 차이
---
<본 포스팅은 [다음](http://it4all.tistory.com/16)을 참고하였습니다>


라이브러리 만드는 이유는 소스의 외부 유출 없이, 자신이 만든 모듈을 외부와 공유하기 위해서다. 또한 미리 컴파일 되어 있어서 컴파일 시간도 단축된다.
이때 라이브러리는 동적 라이브러리와 정적 라이브러리가 있는데, 각각의 특징을 살펴보자.
아래의 특징을 보고 동적 라이브러리와 정적 라이브러리 중 어느 것을 사용할 지를 결정하면 된다.

# 1. 동적 라이브러리(Dynamic Link Library)

* 프로그램 실행 시 필요시만 외부 DLL 파일에서 함수를 참조
* 프로그램 실행 시 프로그램 로딩 시간이 단축
* 함수 업그레이드 시 해당 DLL만 수정 배포한다
* 소스 외부 유출 방지 효과
* 실행 파일 만들때 필요한 파일: *.h,*.lib (*.dll 참조 용)
* 프로그램 실행할 할 때 필요한 파일: *.dll (배포할 때 *.dll 필요)
* dll 제작 시 lib도 같이 생성됨

# 2. 정적 라이브러리(Static Link Library)
 
* 필요한 함수를 프로그램 코드에 붙여 프로그램 자체에서 참조
* 프로그램 실행 후 빠른 처리시간
* 프로그램 실행 파일만 있으면 실행(하나의 파일만 있으면 됨)
* 소스 외부 유출 방지 효과
* 실행 파일 만들때 필요한 파일: *.h,*.lib (별도의 *.dll 필요 없음)


따라서 실행파일 자체의 크기를 비교하면 정적라이브러리를 포함한 프로그램의 크기가 더 크다. 

 
### 참조 사이트
* [http://wolfsi.egloos.com/2302024](http://wolfsi.egloos.com/2302024)
* [http://blog.naver.com/PostView.nhn?blogId=clover6652&logNo=100063061060](http://blog.naver.com/PostView.nhn?blogId=clover6652&logNo=100063061060)
* [http://jym2568.egloos.com/1186568](http://jym2568.egloos.com/1186568)
* [http://blog.naver.com/PostView.nhn?blogId=ssib0228&logNo=110013563400&redirect=Dlog&widgetTypeCall=true](http://blog.naver.com/PostView.nhn?blogId=ssib0228&logNo=110013563400&redirect=Dlog&widgetTypeCall=true)
