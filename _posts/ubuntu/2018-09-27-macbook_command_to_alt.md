---
layout: post
title: '[Mac OS, Ubuntu] Macbook command key to alt in Ubuntu 16.04 (Ubuntu16.04에서 command키 alt로 사용)'
tags: [ubuntu]
description: >
    Macbook command key to alt
---

맥북에 ubuntu를 설치해서 사용하면 command키가 은근 거슬린다. 

원래 alt자리에 command키가 있기 때문에 평소처럼 command키를 alt키로 사용하고 싶다.

그럴때는 `gnome-tweak-tool` 을 이용하자

```
sudo apt-get update
sudo apt-get install gnome-tweak-tool
```

설치를 한 후 실행한다. 터미널을 열고 실행

```
gnome-tweak-tool
```

실행 후 `typing` 탭에서 `alt/win key behavior`을 조정해 주면되는데

`alt is mapped to win keys`로 설정해 주면 command키가 alt키로 변경된다.



