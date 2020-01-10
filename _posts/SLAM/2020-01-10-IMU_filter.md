---
layout: post
title: '[SLAM] IMU Filter (AHRS)'
tags: [SLAM]
description: >
    IMU Filter 관련 정리
sitemap :
  changefreq : weekly
  priority : 1.0
---

IMU는 Angular velocity를 측정하는 Gyroscope, 가속도를 측정하는 Accelerometer, 자기장을 측정하는 Magnetometer 센서를 포함하고 있다. IMU Filter (AHRS)는 이 데이터를 이용해서 센서의 Global 회전 (Orientation)을 계산한다.

관련 ROS package [http://wiki.ros.org/imu_filter_madgwick](http://wiki.ros.org/imu_filter_madgwick)

관련 Report [https://www.x-io.co.uk/res/doc/madgwick_internal_report.pdf](https://www.x-io.co.uk/res/doc/madgwick_internal_report.pdf)

<iframe src="//www.slideshare.net/slideshow/embed_code/key/rhqDlzVDed1uNT" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/JinyongJeongSteve/imu-filter-ahrs-218245013" title="IMU Filter (AHRS)" target="_blank">IMU Filter (AHRS)</a> </strong> from <strong><a href="https://www.slideshare.net/JinyongJeongSteve" target="_blank">Jinyong Jeong Steve</a></strong> </div>



