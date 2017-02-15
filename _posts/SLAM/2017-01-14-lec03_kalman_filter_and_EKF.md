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

* KF (Kalman Filter)와 EKF (Extended Kalman Filter)는 공통적으로 Gaussian 분포를 가정한다. 즉, 위의 Bayes filter는 모든 확률분포에 대한 식이며, 그 중에서 KF와 EKF는 모든 분포(control input, observation 등)를 gaussian으로 가정한다.

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

Gaussian 분포는 single variable과 multi variable의 Gaussian 분포로 표현할 수 있으며, SLAM에서는 Vector를 이용하여 로봇의 상태(state), 센서 입력, 관찰 값 등을 표현하므로 multi variable의 Gaussian을 많이 사용한다. 따라서 Multi variable의 Gaussian 식은 숙지하는 것이 좋다.

### 선형 모델에서 Gaussian 분포의 변환 (Linear transformation of Gaussian distribution)

가장 기본적인 선형 모델은 다음과 같다.

$$
Y = AX+B
$$

이 때, 확률변수 X가 Gaussian 분포를 갖고 있으며 다음과 같을 때,

$$
X \sim \mathcal{N}(\mu_x,\Sigma_x)
$$

선형 변환 후의 확률변수인 Y의 분포는 다음과 같다.

$$
Y \sim \mathcal{N}(A \mu_x+B,A \Sigma_x A^T)
$$

##### 유도과정

X의 평균인 $$\mu_x$$ 는 선형 변환에 의해서 $$A\mu_x+B$$가 되는 것은 직관적으로 이해 할 수 있다. 그렇다면 covariance matrix은 어떻게 유도가 될까? 우선 covariance matrix의 정의로 부터 시작한다.

$$
\begin{aligned}
\Sigma_y &= E((y-\mu_y)(y-\mu_y)^T)\\
         &= E((y-(A\mu_x+B))(y-(A\mu_x+B))^T)\\
         &= E(((AX+B)-(A\mu_x+B))((AX+B)-(A\mu_x+B))^T)\\
         &= E([A(X-\mu_x)][A(X-\mu_x)]^T)\\
         &= E(A(X-\mu_x)(X-\mu_x)^TA^T)\\
         &= AE((X-\mu_x)(X-\mu_x)^T)A^T\\
         &= A \Sigma_x A^T
\end{aligned}
$$

위의 유도처럼 Gaussian 분포의 선형변환에서의 covariance는 covariance의 정의로부터 $$\Sigma_y = A \Sigma_x A^T$$ 로 정의된다. 이 관계는 Kalman filter 뿐만 아니라 Gaussian을 사용하는 여러 분야에서 자주 사용되므로 기억해 두는것이 좋다.

### Kalman Filter (KF)

* Kalman filter는 선형 모델(Linear model)에서 uncertainty의 분포를 Gaussian으로 가정하였을 때의 solution이다.
* Kalman filter는 motion 모델과 observation 모델을 선형으로 가정한다.
* 노이즈는 평균(mean)이 0인 Gaussian 분포로 가정한다.

 $$
\begin{aligned}
x_t &= A_t x_{t-1} + B_t u_t + \epsilon_t\\
z_t &= C_t x_t + \delta_t
\end{aligned}
 $$

 * $$A_t$$ : control input($$u_t$$)과 노이즈($$\epsilon_t$$)가 없을 때 t-1와 t의 state가 어떻게 관계되어 있는지를 의미하는 n x n matrix.
 * $$B_t$$ : control input($$u_t$$)이 어떻게 state 변화에 영향을 미치는지를 나타내는 n x l matrix.
 * $$C_t$$ : 현재 로봇의 상태를 나타내는 state($$x_t$$)와 센서의 관측 정보(observation)이 어떤 관계인지를 나타내는 k x n matrix.
 * $$\epsilon_t, \delta_t$$ : 평균이 0이며 covariance가 각각 $$R_t, Q_t$$인 확률변수이며, process noise와 measurement noise를 의미한다.







### Extended Kalman Filter (EKF)





**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
