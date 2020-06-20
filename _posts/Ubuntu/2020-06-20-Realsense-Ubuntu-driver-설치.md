---
layout: post
title: 'Realsense Ubuntu driver 설치'
tags: [Ubuntu]
comments: true
description: >
  Realsense Ubuntu (16.04, 18.04) driver 설치
sitemap :
  changefreq: weekly
  priority : 1.0
---
Ubuntu 에서 Realsense 카메라 설치하기

1. Realsense driver 설치

```jsx
# server's public key 등록:
sudo apt-key adv --keyserver keys.gnupg.net --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
# 서버 추가
# ubuntu 16.04
sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main" -u
# ubuntu 18.04
sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo bionic main" -u
# 라이브러리 설치
sudo apt-get install librealsense2-dkms
sudo apt-get install librealsense2-utils
# developer, debug package 설치 (optional)
sudo apt-get install librealsense2-dev
sudo apt-get install librealsense2-dbg

```

realsense driver 설치 후 `realsense-viewer` 를 terminal에 입력하면 뷰어가 실행된다.

2. ROS package 설치

```jsx
# 기본 ROS package 설치
sudo apt-get install ros-$$ROS_VER-realsense2-camera
```

Ros camera node르 실행하기 위해서는 터미널에서 다음 명령 실행

```jsx
roslaunch realsense2_camera rs_camera.launch
```

### Reference

[IntelRealSense/realsense-ros](https://github.com/IntelRealSense/realsense-ros)

[IntelRealSense/librealsense](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md)