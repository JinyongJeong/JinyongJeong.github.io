---
layout: post
title: '[SLAM] Particle Filter and Monte Carlo Localization'
tags: [SLAM]
description: >
  Particle Filter를 이용한 localization을 설명한다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이번 글에서는 Particle filter를 이용한 localization방법에 대해서 설명한다. EKF나 EIF와 같은 filter들은 Gaussian 분포만을 이용하여 시스템을 모델링하였다. Particle filter의 목적은 아래 그림과 같이 Gaussian이 아닌 임의의 분포를 다루기 위한 접근 방법이다.

<img align="middle" src="/images/post/SLAM/lec11_particle_filter/arbitrary_dist.png" width="100%">

### Particle selection

가중치를 갖는 particle들의 set은 다음과 같이 정의된다.

$$
\chi = \{<x^{[j]},w^{[j]}>\}_{j=1,2,\cdots, J}
$$

$$x^{[j]}$$는 particle의 위치, $$w^{[j]}$$는 particle의 weight를 의미한다. Posterior는 다음과 같이 표현된다.

$$
p(x) = \sum_{j=1}^{J} w^{[j]}\delta_{x^{[j]}}(x)
$$

위 식에서 $$\delta$$는 dirac delta 함수로 함수의 인자로 주어진 값 이외의 출력은 0이며 전체 합(Integral)은 1인 함수이다.

왼쪽 그림은 Gaussian 분포를, 오른쪽 그림은 임의의 분포를 보여준다.

<img align="middle" src="/images/post/SLAM/lec11_particle_filter/compare.png" width="100%">

이러한 분포에서 sample은 어떻게 추출될까?

<img align="middle" src="/images/post/SLAM/lec11_particle_filter/gaussian.png" width="100%">

Gaussian 분포의 경우는 위와 같이 $$-\sigma$$와 $$\sigma$$사이에서 12개의 임의의 값을 뽑고 합을 구함으로써 Gaussian 분포에 맞는 sample을 추출할 수 있다. 하지만 임의의 분포의 경우에는 위의 방법으로 sample을 추출할 수 없다.

<img align="middle" src="/images/post/SLAM/lec11_particle_filter/non_gaussian.png" width="100%">

위 분포는 Gaussian 분포가 아닐 때 sampling 하는 방법을 설명한다. proposal(x)는 실제 분포인 target(x)를 Gaussian 분포로 근사한 함수이다. 근사된 Gaussian 분포인 proposal(x) 함수에서 sample을 추출하고, 각 sample에 weight를 할당함으로써 실제 분포와 유사한 의미를 부여할 수 있다. 이때 할당되는 weight 는

$$
w = \frac{f}{g}
$$

이며, f는 target, g는 proposal의 함수값이다. 위 그림에서와 같이 실제 sample의 수는 Gaussian 분포를 따르지만 weight가 실제 함수의 확률을 반영한다.

### Particle Filter Algorithm and Monte Carlo Localization

위의 sampling 방법을 적용한 particle filter 알고리즘은 다음과 같다.

<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle_filter_algorithm.png" width="100%">

이제 particle filter를 localization에 적용시켜보자. Monte Carlo 방법은 난수를 이용하여 함수의 값을 확률적으로 계산하는 알고리즘이다. 계산하려는 값이 close form(정확한 수식으로 계산되는 형태)이 아니거나 너무 복잡하여 근사적으로 구해야 할 때 주로 사용된다. particle filter는 이러한 Monte Carlo 방법이다. 이제 particle filter를 이용한 localization 알고리즘은 알아보자.

<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle_filter_localization.png" width="100%">

달라진 부분은 sample을 구하는 3번과, weight를 계산하는 4번이다. sample을 구하는 3번식은 로봇의 motion model이 되며, 각 particle의 weight는 센서의 observation model이다. 7번부터 10번까지는 resampling 과정이다.

### Resampling

Resamping 과정은 확률이 작은 particle은 제거하고, 확률이 큰 파티클은 여러개로 나눈다. Resampling을 하는 이유는 우리가 사용할 수 있는 point의 갯수가 한정적이기 때문이며(computation문제와 memory문제 때문에) resampling 전과 후의 particle 갯수는 같아야 한다. 또한 resampling 전에 particle들의 weight는 다르지만, resampling 후에는 모두 같아진다. 아래 그림은 resampling을 하는 두가지 방법을 보여준다.

<img align="middle" src="/images/post/SLAM/lec11_particle_filter/resampling.png" width="100%">

우선 노란색 원은 각 particle의 weight에 맞게 영역을 할당하여 원으로 표현한 것으로 생각할 수 있다. 즉 particle의 weight가 클수록 차지하는 영역이 크므로 랜덤하게 선택될 가능성이 크다.

Roulette wheel 방법의 경우 한번의 1개씩 random하게 particle을 추출한다. Stochastic universal sampling 방법은 random하게 추출함과 동시에 일정한 간격으로 particle을 추출한다. stochastic universal sampling 방법의 장점으로는 계산시간이 선형적이며, 모든 particle의 weight가 같은 경우 이전과 같은 형태로 particle이 resampling 된다는 점이다.

### Particle Filter Localization Example

particle filter localization에서 초기상태는 아래 그림과 같다. 모든 particle은 균일하게 분포되어 있으며 동일한 weight를 갖는다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle1.png" width="100%">

control input에 의해서 모든 particle은 motion update가 진행된다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle2.png" width="100%">

로봇의 센서로부터 주변 환경에 대한 데이터를 수집한다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle3.png" width="100%">

센서 정보를 이용하여 각 particle의 weight를 update한다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle4.png" width="100%">

계산된 weight를 기준으로 resampling과정을 거친다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle5.png" width="100%">

resampling된 particle들을 이용하여 다시 motion update를 한다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle6.png" width="100%">

로봇 센서로부터 주변환경에 대한 데이터를 수집한다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle7.png" width="100%">

데이터로 부터 각 particle의 weight를 update한다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle8.png" width="100%">

update된 weight를 기준으로 다시 resampling을 한다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle9.png" width="100%">

resampling된 particle을 기준으로 motion이 update된다.
<img align="middle" src="/images/post/SLAM/lec11_particle_filter/particle10.png" width="100%">

위 그림과 같이 Particle filter는 motion update, measurement, weight update, resampling과정을 반복하며 로봇의 위치를 추정해 나간다.

### Particle filter 정리

* particle filter는 non-parametric filter이며, recursive bayes filter이다.
* posterior는 weighted samples들의 set으로 표현된다.
* low-dimensional space에서 좋은 결과를 얻을 수 있다.
* particle들은 motion model에 의해서 update 된다.
* observation의 likelihood에 의해서 weight가 결정된다.
* Monte-Carlo localization(MCL)이라고도 부른다.
* MCL은 mobile robot의 위치 추정 분야에서 가장 많이 사용되는 알고리즘 중에 하나이다.


**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
