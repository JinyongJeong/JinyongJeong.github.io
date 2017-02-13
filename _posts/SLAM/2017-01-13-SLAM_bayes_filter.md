---
layout: post
title: '[SLAM] Bayes filter(베이즈 필터)/ motion and observation model'
tags: [SLAM]
description: >
  SLAM framework에서 Bayes filter와 motion, observation model에 대한 설명.
sitemap :
  changefreq : weekly
  priority : 1.0
---

<본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 부분을 지적해주시면 확인 후 수정토록 하겠습니다.>

## SLAM(Simultaneous Localization and Mapping)

SLAM은 simultaneous localization and mapping의 줄임말로 위치추정(localization)과 지도생성(mapping)을 동시에 하는 연구분야를 의미한다. 이는 닭이 먼저냐 달걀이 먼저냐의 문제와 비슷하다. 자기의 위치를 추정하기 위해서는 주변환경에 대한 정보가 필요하다. 반면에 로봇이 얻을 수 있는 데이터를 이용해서 지도를 만들기 위해서는 로봇이 자신의 위치가 어디에 있는지를 정확히 알아야 한다. 따라서 위치를 알 수 없으면 지도를 만들 수 없고, 반대로 지도가 없으면 위치를 알 수 없다. 이러한 문제를 풀기 위해서 지도의 생성과 위치 추정을 동시에 수행하는 것이 SLAM이다.

### State estimation

State estimation은 로봇에 주어지는 입력과, 로봇의 센서로부터 얻어지는 데이터로부터 현재의 로봇의 위치인 state와 주변환경에 대한 지도를 추정 방법이다.

$$ p(\mathbf{x}\mid \mathbf{z}, \mathbf{u})$$

위의 식은 기본적인 state estimation을 의미한다. $$\mathbf{x}$$ 는 로봇의 위치 및 지도(주변의 land mark들의 위치)를 의미하는 vector이며, $$\mathbf{z}$$ 는 로봇의 센서로부터 얻어지는 데이터로 observation이라고 부르며, $$\mathbf{u}$$ 는 센서의 움직임을 제어하는 입력으로 control input이라고 부른다. state estimation은 이러한 control input과 observation의 데이터를 통해 현재의 위치와 지도를 추정한다.

### bayes theorem

베이즈 정리는 확률론과 통계학에서 두 확률변수의 사전확률(prior)과 사후확률(posterior) 사이의 관계를 나타내는 정리이다.

$$
P(A \mid B) = \frac{P(B \mid A)P(A)}{P(B)}
$$

$$P(A)$$ 는 A의 prior로, 사건 B에 대한 어떠한 정보를 알지 못하는 것을 의미한다. $$P(A \mid B)$$ 는 B의 값이 주어진 경우 A의 posterior이다. $$P(B \mid A)$$ 는 A가 주어졌을 때 B의 조건부 확률이다.

bayes 정리의 자세한 내용은 [wiki](https://ko.wikipedia.org/wiki/%EB%B2%A0%EC%9D%B4%EC%A6%88_%EC%A0%95%EB%A6%AC)를 참고한다.

### Recursive bayes filter

위에서 설명한 state estimation은 bayes filter의 과정으로 설명할 수 있으며, 각 step의 state를 반복적으로 계산함으로써 계산할 수 있기 때문에 recursive bayes filter로 부른다. 전체적인 recursive bayes filter의 식은 다음과 같다.

$$
\begin{aligned}
bel(x_t) &= p(x_t \mid z_{1:t},u_{1:t}) \\
       &= \eta p(z_t \mid x_t, z_{1:t-1}, u_{1:t})p(x_t \mid z_{1:t-1},u_{1:t}) \\
       &= \eta p(z_t \mid x_t)p(x_t \mid z_{1:t-1},u_{1:t}) \\
       &= \eta p(z_t \mid x_t)\int_{x_{t-1}}p(x_t \mid x_{t-1}, z_{1:t-1}, u_{1:t})p(x_{t-1} \mid z_{1:t-1}, u_{1:t}) dx_{t-1}\\
       &= \eta p(z_t \mid x_t)\int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t})p(x_{t-1} \mid z_{1:t-1}, u_{1:t}) dx_{t-1}\\
       &= \eta p(z_t \mid x_t)\int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t})p(x_{t-1} \mid z_{1:t-1}, u_{1:t-1}) dx_{t-1}\\
       &= \eta p(z_t \mid x_t)\int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t}) bel(x_{t-1}) dx_{t-1}\\
\end{aligned}
$$

위의 식은 recursive bayes filter를 유도하는 과정을 모두 표현하고 있기 때문에 다소 복잡해 보인다. 우선 전체적인 식을 이해하기 위해서 맨 처음과 맨 마지막 식만을 보면 다음과 같다.

$$
\begin{aligned}
bel(x_t)  &= p(x_t \mid z_{1:t},u_{1:t}) \\
          &= \eta p(z_t \mid x_t)\int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t}) bel(x_{t-1}) dx_{t-1}\\
\end{aligned}
$$

$$bel(x_t)$$ 는 처음부터 현재까지의 observation( $$z$$ )와 control input( $$u$$ )을 알고 있을 때 현재 state( $$x_t$$ )의 확률을 의미한다. 위의 식에서 $$bel(x_t)$$ 의 식은 $$bel(x_{t-1})$$ 의 integral로 표현되어 있기 때문에 만약 $$p(z_t \mid x_t)$$ 와 $$p(x_t \mid x_{t-1}, u_t)$$ 에 대한 정보를 알고 있다면 반복적인 계산을 통해 현재 state의 확률을 계산할 수 있음을 알 수 있다. 이를 Recursive bayes filter라고 한다. Recursive bayes filter는 SLAM 문제를 푸는데 가장 기본적인 식이 되며, Kalman filter의 기본적인 식이다. 다음은 recursive bayes filter의 유도과정을 간단하게 살펴본다. 유도에 관심이 없고 전체적인 흐름만 보고자 한다면 다음 설명은 넘어가도 좋을 것 같다.

#### Recursive bayes filter의 유도과정
