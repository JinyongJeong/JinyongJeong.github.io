---
layout: post
title: '[Mac OS, Ubuntu] Make function key reverse in ubuntu(Macbook에 ubuntu 설치했을 때 function키 반전)'
tags: [ubuntu]
description: >
    Make function key reverse in ubuntu(Macbook에 ubuntu 설치했을 때 function키 반전)
---

맥북에 듀얼부팅으로 ubuntu를 설치하였을 때 문제는 fuction키가 default로 켜져있다는 것이다. 

윈도우처럼 alt + f4를 동작시키기 위해서는 alt + fn + f4를 눌러야 한다. 

윈도우처럼 동작하게 하기 위해서는 function 키를 반전시켜야 한다. 

다양한 방법을 해봤지만 가장 잘되는 방법은 다음과 같다. 

```
echo options hid_apple fnmode=2 | sudo tee -a /etc/modprobe.d/hid_apple.conf
sudo update-initramfs -u -k all
sudo reboot # optional
```

재부팅을 해도 계속 적용된다. 



