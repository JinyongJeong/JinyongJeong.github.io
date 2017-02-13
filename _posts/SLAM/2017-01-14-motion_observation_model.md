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

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 부분을 지적해주시면 확인 후 수정토록 하겠습니다.**

이번 글에서는 SLAM의 framework에서 중요한 Motion model과 Observation model에 대해서 설명한다. 이전 post([bayes filter](http://jinyongjeong.github.io/2017/01/13/SLAM_bayes_filter/))에서 설명한 것 처럼 SLAM은 다음과 같이 recursive bayes filter의 식으로 표현할 수 있다.

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

$$\overline{bel}(x_t)$$ 는 prediction 단계의 state를 나타낸다. prediction step은 Control input 데이터($$u_t$$)와 이전 step의 로봇의 state에 대한 데이터($$x_{t-1}$$)를 이용하여 현재의 로봇 state($$x_t$$)의 확률을 추정하는 과정이다. 이 과정에서 현재 state의 확률을 계산하는 $$p(x_t \mid x_{t-1}, u_{t})$$ 는 입력($$u_t$$)에 의한 로봇의 움직임을 추정하여 현재의 state의 확률을 계산하는 model이기 때문에 **Motion model** 이라고 부른다.

### correction step

$$
bel(x_t) = \eta p(z_t \mid x_t)\overline{bel}(x_t)
$$

Correction step은 prediction step에서 예상한 로봇의 위치를 보정하는 단계이다. 즉, input data를 이용하여 현재의 로봇의 위치를 예측하고, 그 위치에서 얻어진 센서 데이터로 부터 예측된 로봇의 위치를 보정하는 단계이다.
이때 $$p(z_t \mid x_t)$$ 는 **observation model** 이라 하며, 현재 예측한 state($$x_t$$)에서 실제 얻어진 센서 데이터($$z_t$$)가 얻어질 확률을 의미한다.

### Motion model



### Observation model



**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
