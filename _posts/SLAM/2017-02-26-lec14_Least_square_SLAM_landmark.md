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

[graph-based SLAM](http://jinyongjeong.github.io/2017/02/26/lec13_Least_square_SLAM/)에서는 landmark가 없는 환경에서 로봇의 위치 간의 관계만을 이용하여 graph를 최적화 시키는 방법에 대해서 설명하였다. 이번글에서는 landmark가 있는 환경에서 이 landmark들을 이용하여 로봇간의 위치정보를 알 수 있을 때 graph-based SLAM이 어떻게 달라지는지 설명할 것이다.

## Graph-based SLAM with Landmark

아래 그림은 Global 좌표계에서 로봇의 위치와 landmark의 위치를 보여주는 그림이다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/landmark1.png" width="100%">

Landmark가 존재하는 실제 환경은 아래와 같다. 아래 그림과 같은 공원의 경우 공원에 있는 나무들이 landmark가 되고 나무들의 위치정보를 이용하여 SLAM을 적용한다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/landmark2.png" width="100%">

로봇의 위치와 landmark와의 관계를 도식적으로 그려보면 다음과 같다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/landmark3.png" width="100%">

이제 graph에서 node는 로봇의 위치뿐만 아니라 landmark의 위치도 포함하게 된다. 이때 landmark는 방향없이 x,y좌표로만 구성된다.

* Node
  * 로봇의 위치
  * landmark의 위치
* Edge
  * 로봇의 Odometry measurement
  * 로봇의 Landmark observation

위의 정보들로 구성된 graph의 error를 최소화 하는 landmark와 로봇의 위치를 계산함으로써 로봇의 위치를 구할 수 있다. Landmark가 없는 환경에서 "virtual measurement"라는 것을 언급했었다. "virtual measurement"는 non-successive한 로봇의 위치에서 얻어진 센서 데이터를 이용하여 계산된 두 로봇간의 상대 위치(relative pose)를 의미한다. 이렇게 계산된 "virtual measurement"는 non-successive한 노드 사이의 constraint가 된다. Landmark가 있는 환경에서는 위 그림과 같이 non-succesive한 로봇간의 direct한 edge는 발생하지 않으며, landmark를 통해서만 연결된다.

### Rank of Information matrix

Landmark를 통한 graph SLAM의 경우 센서 특성에 따른 information matrix의 rank를 살펴보아야 한다. 여기서는 *bearing only observation* 센서모델로 로봇의 위치에서 landmark까지의 각도만을 측정할 수 있는 센서의 경우를 생각한다.
Bearing observation의 경우 observation function은 다음과 같다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/observation_fuction.png" width="100%">

이 observation function을 이용한 error term은 다음과 같다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/error_function.png" width="100%">

위 error term으로 구성되는 information matrix의 rank는 어떻게 될까?

$$
\mathbf{H}_{ij} = \mathbf{J}_{ij}^T \mathbf{\Omega}_{ij} \mathbf{J}_{ij}
$$

위 식에서 $$\mathbf{J}_{ij}$$는 error term을 편미분한 행렬인데, bearing only 센서의 경우 error term의 dimension은 1이므로 Jacobian의 rank는 1이다. 따라서 $$\mathbf{H}_{ij}$$의 rank도 1이 된다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/bearing_rank.png" width="100%">

rank가 1이라는 것은 무엇을 의미할까? rank가 1이라는 것은 위 그림과 같이 어떠한 landmark가 있을 때 로봇은 x-y plane에 어디든 존재할 수 있으며 단지 그 x-y좌표에 해당하는 heading 각도만 한정된다는 의미이다. 따라서 bearing only sensor의 경우 로봇의 위치를 정확히 알기 위해서는 3개 이상의 observation이 필요하다. 이러한 시스템을 "under-determined" 시스템이라고 부른다.

따라서 Information matrix의 rank는 로봇의 위치 중(x,y,heading) 몇가지의 정보를 한정할 수 있는지를 의미하며, 로봇의 위치에 대한 unique solusion을 계산하기 위해서는 full lank가 되어야 한다.

### under-determined system

이러한 under-determined system, 즉 information matrix의 rank가 full lank가 아닌 경우 이러한 상황을 해결하기 위한 방법이 "Levenberge Marquardt" method 이다. 즉 "damping factor"를 더함으로써 full rank의 matrix를 만들어 unique한 해를 찾는다. 원래의 식은 아래와 같다.

$$
\mathbf{H} \triangle \mathbf{x} = -\mathbf{b}
$$

damping factor가 추가된 식은 아래와 같다.

$$
(\mathbf{H} + \lambda \mathbf{I})\triangle \mathbf{x} = -\mathbf{b}
$$

damping factor는 error의 증감에 따라 크기를 변화시킨다. 전체적인 알고리즘은 다음과 같다.

<img align="middle" src="/images/post/SLAM/lec14_least_square_SLAM_llandmark/LM_algorithm.png" width="100%">

### 정리

이 글에서는 Landmark가 있는 환경에서의 Graph-based SLAM에 대해서 알아보았다. Landmark가 없는 환경에서는 센서 데이터를 이용하여 non-successive node간의 edge정보를 계산하였다. 이러한 edge는 node간의 직접적인 연결이다. 반면, landmark가 있는 환경에서는 각 node에서 바라보는 landmark의 위치가 observation 정보가 되며, non-successive한 node간의 연결은 landmark를 통해서 이루어진다. 이때 센서의 종류에 따라 information matrix의 rank가 결정되고 under-determined system이 되므로 이를 풀기위한 기법이 필요하게 된다(LM method).

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
