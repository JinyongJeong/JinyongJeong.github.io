---
layout: post
title: '[SLAM] Graph-based SLAM with Landmark'
tags: [SLAM]
description: >
  Landmark가 있는 상황에서의 graph-based SLAM에 대해서 설명한다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

[graph-based SLAM](http://jinyongjeong.github.io/2017/02/26/lec13_Least_square_SLAM/)에서는 landmark가 없는 환경에서 로봇의 위치 간의 관계만을 이용하여 graph를 최적화 시키는 graph-based SLAM에 대해서 설명하였다. 이번글에서는 landmark가 있는 환경에서 이 landmark들을 이용하여 로봇간의 위치정보를 알 수 있을 때 graph-based SLAM이 어떻게 달라지는지 설명할 것이다.

## Graph-based SLAM with Landmark

아래 그림은 Global 좌표계에서 로봇의 위치와 landmark의 위치를 보여주는 그림이다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/landmark1.png" width="100%">

Landmark가 존재하는 실제 환경은 아래와 같다. 아래 그림과 같은 공원의 경우 공원에 있는 나무들이 landmark가 되고 나무들의 위치정보를 이용하여 SLAM을 적용한다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/landmark2.png" width="100%">

로봇의 위치와 landmark와의 관계를 도식적으로 그려보면 다음과 같다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/landmark3.png" width="100%">

이제 graph에서 node는 로봇의 위치뿐만 아니라 landmark의 위치도 포함하게 된다.

* Node
  * 로봇의 위치
  * landmark의 위치
* Edge
  * 로봇의 Odometry measurement
  * 로봇의 Landmark observation

위의 정보들로 구성된 graph의 error를 최소화 하는 landmark와 로봇의 위치를 계산함으로써 로봇의 위치를 구할 수 있다. Landmark가 없는 환경에서 "virtual measurement"라는 것을 언급했었다. "virtual measurement"는 각 로봇의 위치에서 얻어진 센서 데이터를 이용하여 계산된 두 로봇간의 상대 위치(relative pose)를 의미한다. 이렇게 계산된 "virtual measurement"는 두 노드 사이의 constraint가 된다. Landmark가 있는 환경에서는 위 그림과 같이 두 로봇간의 비연속적인(non-successive) edge는 발생하지 않으며, landmark를 통해서만 두 로봇이 연결된다.

### Rank of Information matrix






**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
