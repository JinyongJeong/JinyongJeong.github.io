---
layout: post
title: '[SLAM] Extended Kalman Filter(EKF) 예제'
tags: [SLAM]
description: >
  Robot의 실제 모델을 이용하여 EKF를 설명.

sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이번 글에서는 이전 글에서 설명한 Extended Kalman Filter를 실제 모델을 이용해서 설명한다.

<img align="middle" src="/images/post/SLAM/lec04_EKF_example/velocity_model.png" width="100%">


**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
