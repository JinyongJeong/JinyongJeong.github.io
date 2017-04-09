---
layout: post
title: 'Ubuntu 16.04.2 bluetooth load fail 문제'
tags: [ubuntu]
description: >
    최신 intel wireless chipset을 사용하는 노트북에서 ubuntu 부팅시 bluetooth를 인식하지 못하는 문제 해결방법
---

인텔 최신 칩셋인

Intel® Dual Band Wireless-AC 8265

Intel® Dual Band Wireless-AC 8260 

과 같은 무선칩셋을 사용하는 노트북의 경우 ubuntu 16.04가 부팅할 때 다음과 같은 오류 메세지를 띄우면서 부팅이 안되는 경우가 발생한다. 

```
Intel 8265 Dual-band Wireless Bluetooth Drivers Failing to Load
```

위와 같은 최신 인텔 칩셋은 16.04.1에서는 지원하지 않으며 16.04.02의 경우 wifi는 잘 연결된다. 
16.10은 문제없이 모두 지원하는 것 같으나 LTS버전이 아니기 때문에 본인은 16.04버전을 선호한다. 

위와 같이 오류 메세지가 뜨면서 부팅이 안되는 경우 아래의 Ubuntu forum글을 참고하면된다. 

[참고 url](https://ubuntuforums.org/showthread.php?t=2354736)

해결방법:

```
sudo apt-get install linux-generic-hwe-16.04-edge xserver-xorg-input-libinput-hwe-16.04 && wget http://mirrors.kernel.org/ubuntu/pool/main/l/linux-firmware/linux-firmware_1.161.1_all.deb && sudo dpkg -i linux-firmware_1.161.1_all.deb
```

위의 명령어를 sudo 권한으로 실행시킨다. 본인의 경우 dual 부팅을 사용하고 있는데 바로 ubuntu를 실행하는 경우에는 오류가 뜨고, 윈도우를 한번 실행시켰다가 다시 ubuntu로 부팅하는 경우는 정상적으로 부팅이 되는 상황이라 위의 해결방법으로 해결하였다. 
