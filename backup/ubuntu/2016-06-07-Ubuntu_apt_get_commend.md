---
layout: post
title: 'Ubuntu apt-get 명령어 정리'
tags: [ubuntu]
description: >
  Ubuntu apt-get 명령어 정리
---
apt-get(Advanced Packaging Tool)은 우분투(Ubuntu)를 포함안 데비안(Debian)계열의 리눅스에서 쓰이는 팩키지 관리 명령어 도구입니다. 우분투에는 GUI로 되어 있는시냅틱 꾸러미 관리자도 있기는 하지만 이런 저런 개발관련 패키지를 설치할 때는 커맨드기반인 apt-get이 더 편하기도 합니다. sudo는 superuser권한으로 실행하기 위함입니다.

## 1. 패키지 인덱스 인덱스 정보 업데이트

apt-get은 인덱스를 가지고 있는데 이 인덱스는/etc/apt/sources.list에 있습니다. 이곳에 저장된 저장소에서 사용할 패키지의 정보를 얻습니다.

``` 
sudo apt-get update
```

## 2. 설치된 패키지 업그래이드 

설치되어 있는 패키지를 모두 새버전으로 업그래이드 합니다.

```
sudo apt-get upgrade
```

## 3. 의존성검사하며 설치하기

```
sudo apt-get dist-upgrade
```

## 4. 패키지 설치

```
sudo apt-get install 패키지이름
```

## 5. 패키지 재설치

```
apt-get --reinstall install 패키지이름
```

## 6. 패키지 삭제

설정파일은 지우지 않음

```
sudo apt-get remove 패키지이름
```

## 7. 설정파일까지 모두 지움

```
sudo apt-get --purge remove 패키지이름
```

## 8. 패키지 소스코드 다운로드

```
sudo apt-get source 패키지이름
```

## 9. 위에서 받은 소스코드를 의존성있게 빌드

```
sudo apt-get build-dep 패키지이름
```

## 10. 패키지 검색

```
sudo apt-cache search 패키지이름
```

## 11. 패키지 정보 보기

```
sudo apt-cache show 패키지이름
```

apt를 이용해서 설치된 deb패키지는 /var/cache/apt/archive/ 에 설치가 됩니다.
