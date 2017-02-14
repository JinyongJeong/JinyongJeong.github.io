---
layout: post
title: '[SLAM] Kalman filter and EKF(Extended Kalman Filter)'
tags: [SLAM]
description: >
  Kalman filter와 Extended Kalman filter에 대한 설명.

sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 부분을 지적해주시면 확인 후 수정토록 하겠습니다.**

이번 글에서는 Kalman filter와 Kalman filter의 확장판인 EKF(Extended Kalman Filter)에 대해서 설명한다. 앞의 글에서 설명한 Bayes filter는 로봇의 상태(state)를 추정하기 위한 방법 중에 한가지 이며, 예측(prediction)단계와 보정(correction)단계의 두 단계로 나뉘어 진다.

* Prediction step

$$
\overline{bel}(x_t) = \int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t}) bel(x_{t-1}) dx_{t-1}
$$

* Correction step

$$
bel(x_t) = \eta p(z_t \mid x_t)\overline{bel}(x_t)
$$

Bayes filter에 대한 자세한 설명은 [이전의 글](http://jinyongjeong.github.io/2017/01/14/lec02_motion_observation_model/)을 참고하기 바란다.

### Kalman Filter & EKF (Extended Kalman Filter)

* Kalman filter는 로봇의 state를 추정하기 위해 가장 흔히 사용되는 방법이며, Bayes filter이다. 즉 control input에 의한 prediction 단계와, 센서의 observation를 이용한 correction의 두 단계로 나누어 진다.

* KF (Kalman Filter)와 EKF (Extended Kalman Filter)는 공통적으로 Gaussian 분포를 가정한다. 즉, 위의 Bayes filter는 모든 확률분포에 대한 식이며, 그 중에서 KF와 EKF는 모든 분포(control input, observation 등)의 분포를 gaussian으로 가정한다.

* KF는 선형 Gaussian 모델의 경우이며, EKF는 비선형 Gaussian 모델이다.

### Gaussian 분포

* Gaussian distribution (normal distribution)

$$
p(x) = \frac{1}{\sqrt{2\sigma^2 \pi}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}
$$


* Multi variable Gaussian distribution

$$
p(\mathbf{x}) = \frac{1}{\sqrt{det(2\pi \Sigma)}}e^{-\frac{1}{2}(\mathbf{x}-\mu)^T\Sigma^{-1}(\mathbf{x}-\mu)}
$$

Gaussian 분포는 single variable과 multi variable의 Gaussian 분포로 표현할 수 있으며, SLAM에서는 Vector를 이용하여 로봇의 상태(state), 센서 입력, 관찰 값 등을 표현하므로 multi variable의 Gaussian을 많이 사용한다. 따라서 Multi variable의 Gaussian 식은 숙지해 두도록 하자.

### 선형 모델에서 Gaussian uncertainty의 propagation



$$
Y = AX
$$

$$
X \sim N(\mu_x,\Sigma_x)
$$

$$
Y \sim N(A \mu_x,A \Sigma_x A^T)
$$


### Kalman Filter (KF)

### Extended Kalman Filter (EKF)





**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
