---
layout: post
title: '[SLAM] Motion & Observation model'
tags: [SLAM]
description: >
  Motion model과 observation model 설명.

sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이번 글에서는 SLAM의 framework에서 중요한 Motion model과 Observation model에 대해서 설명한다. 이전 post([bayes filter](http://jinyongjeong.github.io/2017/01/13/lec01_SLAM_bayes_filter/))에서 설명한 것 처럼 SLAM은 다음과 같이 recursive bayes filter의 식으로 표현할 수 있다.

$$
\begin{aligned}
bel(x_t)  &= p(x_t \mid z_{1:t},u_{1:t}) \\
          &= \eta p(z_t \mid x_t)\int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t}) bel(x_{t-1}) dx_{t-1}\\
\end{aligned}
$$

bayes filter식은 **prediction step** 과 **correction step** 으로 나눌 수 있다.

### prediction step

$$
\overline{bel}(x_t) = \int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t}) bel(x_{t-1}) dx_{t-1}
$$

$$\overline{bel}(x_t)$$ 는 prediction 단계의 state를 나타낸다. prediction step은 control input 데이터($$u_t$$)와 이전 step의 로봇의 state에 대한 데이터($$x_{t-1}$$)를 이용하여 현재의 로봇 state($$x_t$$)의 확률을 추정하는 과정이다. 이 과정에서 $$p(x_t \mid x_{t-1}, u_{t})$$ 는 입력($$u_t$$)에 의한 로봇의 움직임을 추정하여 현재의 state의 확률을 계산하는 model이기 때문에 **Motion model** 이라고 부른다.

### correction step

$$
bel(x_t) = \eta p(z_t \mid x_t)\overline{bel}(x_t)
$$

Correction step은 prediction step에서 예상한 로봇의 위치를 보정하는 단계이다. 즉, input data($$u_t$$)를 이용하여 현재의 로봇의 위치를 예측하고, 그 위치에서 얻어진 센서 데이터($$z_t$$)로 부터 예측된 로봇의 위치를 보정하는 단계이다.
이때 $$p(z_t \mid x_t)$$ 는 **observation model** 이라 하며, 현재 예측한 state($$x_t$$)에서 실제 얻어진 센서 데이터($$z_t$$)가 얻어질 확률을 의미한다.

### Motion model

$$
p(x_t \mid x_{t-1}, u_{t})
$$

Motion model은 control input($$u_t$$)이 이전 state($$x_{t-1}$$)을 현재 state($$x_t$$)로 변화시킬 사후확률(posterior probability)를 의미한다. 실제 로봇의 응용에서 motion model은 크게 2가지로 분류된다.

* odometry-based model
* velocity-based model

odometry model은 로봇 혹은 자동차의 바퀴에 달린 wheel encoder의 센서 데이터를 이용한 모델이며, velocity model은 imu와 같은 관성 센서를 이용한 model이다. velocity model은 wheel encoder와 같은 odometry model을 사용할 수 없을 때 주로 사용하며, odometry model이 velocity model보다 더 정확한 편이다.

<img align="middle" src="/images/post/SLAM/lec02_motion_observation_model/odometry_based.png" width="700">

위의 그림은 motion model에 의한 state의 uncertainty를 나타낸다. 점이 많이 분포되거나, 어두운 부분일 수록 확률이 높은 부분이다. 로봇의 진행방향과 회전방향에 대한 uncertainty의 크기에 따라서 다른 확률 분포를 보인다. 맨 왼쪽의 그림은 고르게 분포되었을 경우이며, 가운데 그림은 진행방향의 uncertainty가 더 큰경우이며 마지막의 오른쪽 그림은 회전방향의 uncertainty가 진행방향보다 큰 경우를 보여주고 있다.



### Observation model

$$
p(z_t \mid x_t)
$$

observation model은 현재 state에서의 센서 데이터의 확률을 의미한다. 로봇 분야에서 많이 사용하는 Laser Scanner데이터를 이용하여 model을 표현하면 다음과 같다.

$$
\mathbf{z_t} = \{z_t^1,...,z_t^k\}
$$

$$\mathbf{z_t}$$ 는 각 laser scan 데이터의 그룹을 나타내는 vector이며, $$z_t^1$$ ~ $$z_t^k$$는 t 시점에서의 각 scan 데이터를 의미한다. 따라서 최종 observation model은 다음과 같이 표현된다.

$$
p(z_t \mid x_t) = \prod_{i=1}^{k} p(z_t^i \mid x_t)
$$

즉 observation은 현재 state에서의 각 센서 데이터들의 확률의 곱으로 표현된다.

자세한 Motion & Observation model에 대해서는 [Freiburg 강의-bayes filter](https://www.youtube.com/watch?v=5Pu558YtjYM)를 참조하기 바란다.

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
