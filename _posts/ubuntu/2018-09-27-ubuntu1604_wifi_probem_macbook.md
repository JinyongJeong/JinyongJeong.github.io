---
layout: post
title: '[Mac OS, Ubuntu] Ubuntu16.04 Wifi problem in Macbook(맥북 ubuntu16.04 Wifi 문제)'
tags: [ubuntu]
description: >
    Ubuntu 16.04 Wifi problem in Macbook
---

Macbook에 듀얼부팅으로 ubuntu16.04를 설치하였을 때 Wifi가 정상적으로 잡히지 않는 문제가 발생한다.

구글링을 해보면 다음과 같은 답변이 많이 나오는데 나의경우에는 해결되지 않았다. 

```
sudo apt-get install firmware-b43-installer
sudo modprobe -r b43
sudo modprobe b43
```

위와 같은 방법 말고 bcmwl 커널소스를 재 설치 하는 방법으로 해결되었다. 

혹시 이글을 보고 있다면 위에 코드 실행하기 전에 아래 코드를 먼저 실행해 보기 바란다. 

```
sudo apt-get update
sudo apt-get install --reinstall bcmwl-kernel-source
```

기타 Mac 에 우분투를 설치시 다양한 문제들에 대한 정보는 

[다음 링크](https://help.ubuntu.com/community/MacBookPro5-4/Lucid#Keyboard%20Functions)에서 찾아볼수 있다.
