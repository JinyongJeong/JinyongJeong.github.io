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

<iframe width="640" height="480" src="https://www.youtube.com/embed/JLjFxDMovnc" frameborder="0" allowfullscreen> </iframe>

동영상에서 차량이 움직이는 경로의 실선은 constraint이며, 실선 중간에 생성되는 네모는 graph의 node이다. 위 application은 도로 상의 marking정보를 이용하여 장소를 인식하고 non-successive node사이의 constraint를 추가함으로써 odometry의 error를 보정한다. 동영상에서 이러한 constraint를 추가함으로써 계속적으로 graph가 최적화 되는 모습을 볼 수 있다.

## Overall SLAM system

Graph SLAM은 아래 그림과 같이 크게 Front-end와 Back-end로 나눌 수 있다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/SLAM_overall.png" width="100%">

위의 동영상을 예로 들면, Front-end는 camera로 부터 얻어지는 이미지로 부터 데이터 처리를 통해 sub-map을 만들고, 생성된 sub-map을 통해서 다시 방문한 장소를 인식함으로써 각 node사이의 constraint를 생성하는 과정을 말한다. 이렇게 생성된 constraint들과 node들의 정보를 이용하여 최적화된 node의 위치를 계산하는 과정은 graph SLAM의 back-end에서 수행한다. 최근 많이 사용되는 graph SLAM의 back-end로는 [iSAM](http://ieeexplore.ieee.org/abstract/document/4682731/)과 [g2o](http://ieeexplore.ieee.org/abstract/document/5979949/)가 있다.


## Graph

이 글에서 로봇이 이동하는 환경은 2 dimension으로 가정하며, 로봇의 위치는 다음과 같이 3 DOF로 표현한다.

$$
\mathbf{x}_i = (x_i, y_i, \theta_i)
$$

Graph는 이러한 로봇의 위치를 표현하는 $$\mathbf{x}_i$$로 구성된 vector이다.

$$
\mathbf{x} = \mathbf{x}_{1:n}
$$

Graph를 구성하는 노드 $$\mathbf{x}_i$$와 $$\mathbf{x}_{i+1}$$ 사이에는 odometry 센서로부터 얻어진 measurement가 constraint/edge로 생성된다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/odometry.png" width="100%">

만약 로봇이 같은 장소를 다시 방문했을 때 두 로봇의 위치에서 얻은 measurement를 이용하여 두 노드 사이의 상대적인 위치(relative pose)를 계산할 수 있다. 아래 그림은 다른 위치에서 획득한 같은 물체의 센서 데이터를 보여준다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/virtual.png" width="100%">

두 센서데이터의 매칭을 통해서 두 node사이의 상대위치를 계산할 수 있고, 이 상대위치가 두 node사이의 constraint가 된다. 이때 직접적으로 node사이의 관계를 구한 것이 아닌 센서 measurement로부터 계산하였기 때문에 virtual measurement라고 부른다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/virtual2.png" width="100%">


## Pose graph

아래 그림은 Pose graph SLAM에서 measurement에 대한 error를 그림으로 표현하여 보여준다.  

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/pose_graph.png" width="100%">






**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
