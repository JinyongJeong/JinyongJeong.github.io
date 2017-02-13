---
layout: post
title: 'bash shell 환경변수'
tags: [ubuntu]
description: >
  bash shell 환경변수
---
<본 포스팅은 [다음](http://seven00.tistory.com/680)을 참고하였습니다>

Bash shell 관련 소개 내용

## 환경변수란?

OS레벨에서 "자주사용하는" 또는 "필요한 변수들을 미리 선언해 놓은 것
이 환경변수의 리스트를 보기 위해서는 export라는 명령어를 쓴다.
1개의 환경변수를 보기 위해서는 echo $환경변수(ex: echo $PATH)
$PATH 환경변수에 있는 file들은 어느 위치에서나 실행 가능.

## 환경변수 설정 방법

### 1. export 명령어를 통해 실행

```
export PATH=$PATH:$JAVA_HOME/bin
```

하지만 이 방법은 재부팅 시엔 초기화됨

### 2. .bashrc file 설정을 수정

`vi ~/.bashrc` 로 file을 연후

아래쪽에

```
export PATH="$PATH:/usr/프로그램 경로/bin"
```

를 추가해준다 

$PATH: 를 앞에 추가해 준 것은 기존 PATH 환경변수 뒤에 내용을 추가한다는 의미

이부분을 빼먹으면 기존 설정은 삭제된다.

## 환경변수 적용

컴퓨터 재시작을 하거나 또는

```
source .bashrc
```

와 같이 source 명령어를 이용하면 된다.
