---
layout: post
title: '[SLAM] Graph-based SLAM (Pose graph SLAM)'
tags: [SLAM]
description: >
  Graph SLAM에 대해서 설명한다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이 글에서는 Least square를 이용하여 실제로 어떻게 SLAM에 적용이 되는지 알아볼 것이다. 아직 least squre에 대해서 익숙하지 않다면 이전 글인 [least squre(최소자승법)](http://jinyongjeong.github.io/2017/02/26/lec12_Least_squarees/)을 참고하기 바란다. 이번 글에서의 표기법은 이전글의 표기법을 그대로 사용한다.

## Graph-based SLAM

Graph-based SLAM은 로봇의 위치와 움직임을 graph처럼 node와 edge로 표현함을 의미한다.

* Node: Graph의 node는 로봇의 pose를 의미한다.
* Edge: 두 node사이의 edge는 로봇의 위치 사이의 odometry정보이며 constraint라고 한다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/graph1.png" width="100%">

로봇이 움직이며 odometry 정보를 이용하여 node와 constraint를 생성한다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/graph2.png" width="100%">

로봇이 위 그림과 같이 이전에 방문했던 지역을 다시 방문할 경우 주변 환경에 대한 정보를 이용하여 같은 위치임을 인식하고 연속적이지 않은(non-successive) node사이에 constraint를 추가하고, graph를 최적화 함으로써 measurement에 최적화된 로봇의 위치를 계산할 수 있다.

아래의 youtube 동영상은 pose graph SLAM을 이용한 SLAM의 예이다.


<iframe width="480" height="360" src="https://youtu.be/JLjFxDMovnc" frameborder="0" allowfullscreen> </iframe>

동영상에서 차량이 움직이는 경로의 실선은 constraint이며, 실선 중간에 생성되는 네모는 graph의 node이다. 위 application은 도로 상의 marking정보를 이용하여 장소를 인식하고 non-successive node사이의 constraint를 추가함으로써 odometry의 error를 보정한다. 동영상에서 이러한 constraint를 추가함으로써 계속적으로 graph가 최적화 되는 모습을 볼 수 있다.




**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
