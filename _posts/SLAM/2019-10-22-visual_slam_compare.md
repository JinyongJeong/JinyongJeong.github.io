---
layout: post
title: '[SLAM] KAIST Urban dataset의 stereo image를 이용한 Visual SLAM 비교 실험'
tags: [SLAM]
description: >
    KAIST Urban dataset의 stereo image를 이용하여 Open source들의 결과를 비교
sitemap :
  changefreq : weekly
  priority : 1.0
---

현재 [KAIST Urban Dataset](http://irap.kaist.ac.kr/dataset/)의 데이터는 각 센서들의 raw데이터, stereo image 데이터, 그리고 pointcloud data를 제공하고 있다. 


제공되는 데이터중 Urban39-pankyo 데이터의 stereo image들을 이용해서 여러가지 visual SLAM 알고리즘들의 결과를 비교해본다. (시간이 될때 추가적인 알고리즘 추가 예정)


## [ORB-Stereo2](https://github.com/appliedAI-Initiative/orb_slam_2_ros)

<iframe width="560" height="315" src="https://www.youtube.com/embed/zCxZ1Z7s9lc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

가장 안정적으로 odometry를 추정하는 것을 볼 수 있다. 하지만 사용되는 데이터가 10hz의 이미지이라 속도가 빠른경우 약간씩 tracking 실패를 보이기도 하며, 회전이 빠른경우 tracking에 실패한다. 일반적으로 moving object는 tracking과정에서 어느정도 filtering은 되는것으로 보이지만, 가끔씩 버스와 같은 큰 물체가 있는 경우 버스를 tracking하게 되면 모션이 이상하게 추정된다. 

## [VINS-fusion](https://github.com/HKUST-Aerial-Robotics/VINS-Fusion)

<iframe width="560" height="315" src="https://www.youtube.com/embed/Y28xTrQUOa4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

VINS-Fusion 알고리즘에서 IMU를 사용하지 않고 stereo module만 이용해서 알고리즘을 수행했다. ORB SLAM보다 불안정한 모습을 보이며 중간중간 pose가 튀는 에러가 발생하는 것을 알 수 있다. optimization 수행하는 과정에서 튀는 것으로 보인다. 어느정도 되는 것 처럼 보이다가 결국 발산한다. (발산하는 문제는 IMU가 재대로 들어가면 해결될 수도?)

IMU를 사용했을 경우 IMU data의 calibration 문제인지 발산하는 문제가 있어서 패스!


