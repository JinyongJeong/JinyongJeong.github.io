---
layout: post
title: '[SLAM] Least Squares (최소자승법)'
tags: [SLAM]
description: >
  Graph SLAM의 접근방법인 least square 방법에 대해서 설명한다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

SLAM은 로봇의 위치를 추정함과 동시에 주변 환경에 대한 맵도 같이 생성하는 기술이다. SLAM의 방법은 크게 3가지로 나눌 수 있다.

* [Kalman Filter](http://jinyongjeong.github.io/2017/02/14/lec03_kalman_filter_and_EKF/)
* [Particle Filter](http://jinyongjeong.github.io/2017/02/22/lec11_Particle_filter/)
* Graph-based

이번글에서 부터는 Graph-based SLAM에 대해서 설명할 것이며, 이 글에서는 Graph-based SLAM의 접근방법인 Least square에 대해서 설명한다.

### Least square

* Least square는 "overdetermined system"의 해를 구하기 위한 방법이다. overdetermined system이란 미지수의 갯수보다 식의 수가 더 많기 때문에, 모든 식을 만족하는 해가 존재하지 않는 시스템을 말한다.
* Least square는 에러의 제곱합(sum of the squared error)을 최소화 하는 방법으로 해를 구한다.

### Problem of Least square

* $$\mathbf{x}$$는 state vector
* $$\mathbf{z}_i$$는 실제 measurement(실제 센서 측정값)
* $$\hat{\mathbf{z}}_i=f_i(x)$$는 현재 state vector $$\mathbf{x}$$에서 예상되는 measurement값
* 목표: 센서로부터 실제 얻어진 measurement($$\mathbf{z}_i$$)에 가장 적합한 $$\mathbf{x}$$ 추정

<img align="middle" src="/images/post/SLAM/lec12_least_square/least_square.png" width="100%">

위 그림은 least square를 그림으로 표현하였다. Least square는 센서로부터 측정된 measurement에 가장 적합한 로봇의 state를 계산하는 방법이다. Sensor observation model처럼 로봇의 상태를 알고 있을 때 예상되는 measurement를 예상할 수 있고($$\mathbf{\hat{z}}_i$$) 실제 measurement와 예상되는 measurement와의 차이를 최소화 함으로써 최적화된 로봇의 state를 계산할 수 있다.

### Error Function

Error($$\mathbf{e}_i$$)는 실제 measurement와 예상되는 measurement와의 차이를 나타내며, measurement와 같은 dimension을 갖는 vector이다.

$$
\mathbf{e}_i(\mathbf{x}) = \mathbf{z}_i - f_i(\mathbf{x})
$$

위 식에서 measurement의 noise는 평균이 0이며, Gaussian 분포를 따른다고 가정한다.
Information matrix($$\Omega_i$$)는 measurement의 covariance matrix의 inverse이며, measurement($$\mathbf{z}_i$$)와 dimension의 크기가 같은 matrix이다.

squared error($$e_i(\mathbf{x})$$)는 아래와 같으며, vector가 아닌 scalar값을 갖는다.

$$
e_i(\mathbf{x}) = \mathbf{e}_i^T(\mathbf{x}) \mathbf{\Omega}_i \mathbf{e}_i(\mathbf{x})
$$

### Find Minimum of Error Function

로봇의 최적화된 state를 계산하는 문제는, 주어진 모든 센서 measurement를 이용한 error function의 합이 최소화 되는 state를 찾는 문제이다. 식으로 표현하면 다음과 같다.

$$
\begin{aligned}
x* &= \text{argmin}_{\mathbf{x}} F(\mathbf{x})\\
&=\text{argmin}_{\mathbf{x}} \sum_i e_i(\mathbf{x})\\
&=\text{argmin}_{\mathbf{x}} \sum_i \mathbf{e}_i^T(\mathbf{x}) \mathbf{\Omega}_i \mathbf{e}_i(\mathbf{x})
\end{aligned}
$$

위 식에서 각 element들에 대해서는 error function에서 설명했다. 이 식을 보고 넘어갈 때 각 항들의 dimension이 어떻게 되는지 확인하고 넘어가도록 하자. 만약 센서 measurement의 dimension이 n일 때 $$\mathbf{e}_i$$는 $$n\times1$$, $$\mathbf{\Omega}_i$$는 $$n\times n$$의 dimension을 갖는다.
위와 같은 optimization 문제는 복잡하며, closed form(미지수의 해가 수식으로 정리되는 형태)이 아니기 때문에 수학적인 접근방법으로 해를 구한다.

Optimization 방법으로 최적의 로봇 state($$\mathbf{x}^* $$)를 구하는 순서는 다음과 같다.

1. 현재의 state를 기준으로 error term($$\mathbf{e(x)}$$)의 선형화
2. 선형화된 error term으로 계산되는 squared error function($$e_i(\mathbf{x})$$)의 1차 미분계산
3. squared error function은 quadratic form이므로 1차 미분값이 0이 되는 점이 최소값이므로 미분값이 0인 solution을 계산
4. 계산된 solution을 이용하여 state를 update
5. 위의 과정을 수렴할 때까지 반복

위의 과정을 통해 iterative하게 optimal한 state를 계산할 수 있다. 각 단계의 과정을 자세히 살펴보도록 하자.

#### 1. Error term의 선형화

vector의 형태를 갖는 error term은 현재 measurement와 예상되는 measurement의 차이이다.

$$
\mathbf{e}_i(\mathbf{x}) = \mathbf{z}_i - f_i(\mathbf{x})
$$

이때 $$f_i(\mathbf{x})$$는 observation model로 많은 경우에 비선형형태를 갖는다. observation model에 대해 익숙하지 않다면 이전 글인 [motion model and observation model](http://jinyongjeong.github.io/2017/02/14/lec02_motion_observation_model/)을 참고하기 바란다. 따라서 식을 풀기 위해서는 현재 state기준으로 선형화를 수행한다. 선형화 방법으로는 EKF와 마찬가지로 Tayor expansion방법을 사용한다.

$$
\begin{aligned}
\mathbf{e}_i(\mathbf{x}+\triangle\mathbf{x}) &\approx \mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})((\mathbf{x}+\triangle \mathbf{x})-\mathbf{x})\\
&= \mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})\triangle \mathbf{x}
\end{aligned}
$$

위 식에서 $$\mathbf{x}$$는 initialize의 기준이 되는 현재 state를 의미하며 $$\mathbf{x} = (\mathbf{x}_1, \mathbf{x}_2, \cdots, \mathbf{x}_n)$$이다. 이때 Jacobian $$\mathbf{J}_i$$는 다음과 같다.

$$
\mathbf{J}_i(x) = \begin{pmatrix} \frac{\partial f_1(x)}{\partial x_1} & \frac{\partial f_1(x)}{\partial x_2} & \cdots & \frac{\partial f_1(x)}{\partial x_n} \\
\frac{\partial f_2(x)}{\partial x_1} & \frac{\partial f_2(x)}{\partial x_2} & \cdots & \frac{\partial f_2(x)}{\partial x_n} \\
\cdots & \cdots & \cdots & \cdots \\
\frac{\partial f_m(x)}{\partial x_1} & \frac{\partial f_m(x)}{\partial x_2} & \cdots & \frac{\partial f_m(x)}{\partial x_n}
\end{pmatrix}
$$

#### 2. Squared Error 계산

이제 위에서 선형화한 error term을 이용하여 squared error를 계산해보자.

$$
\begin{aligned}
e_i(\mathbf{x}+\triangle\mathbf{x}) & =
\mathbf{e}_i^T(\mathbf{x}+\triangle\mathbf{x}) \mathbf{\Omega}_i \mathbf{e}_i(\mathbf{x}+\triangle\mathbf{x})\\
&\approx (\mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})\triangle \mathbf{x})^T \mathbf{\Omega}_i (\mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})\triangle \mathbf{x})\\
&= \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{e}_i + \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x} +  \triangle \mathbf{x}^T \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{e}_i + \triangle \mathbf{x}^T \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x}\\
&= \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{e}_i + 2 \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x}\\
&= c_i + 2 \mathbf{b}_i^T \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{H}_i \triangle \mathbf{x}
\end{aligned}
$$

계산된 squared error는 위와 같으며, 마지막 식에서 $$c_i, \mathbf{b}_i, \mathbf{H}_i$$는 다음과 같다.

$$
\begin{aligned}
c_i &= \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{e}_i\\
\mathbf{b}_i^T &= \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{J}_i\\
\mathbf{H}_i &= \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{J}_i
\end{aligned}
$$

$$\mathbf{H}_i$$는 선형화 후의 information matrix이다.

위에서 계산된 $$e_i(\mathbf{x}+\triangle\mathbf{x})$$는 1개의 measurement에 대한 error function이므로 모든 measurement에 대한 error function은 다음과 같다.

$$
\begin{aligned}
F(\mathbf{x}+\triangle \mathbf{x}) &\approx \sum_i (c_i + 2 \mathbf{b}_i^T \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{H}_i)\triangle \mathbf{x}\\
&= \sum_i c_i + 2 (\sum_i \mathbf{b}_i^T) \triangle \mathbf{x} + \triangle \mathbf{x}^T (\sum_i \mathbf{H}_i) \triangle \mathbf{x}\\
&= c + 2 \mathbf{b}^T \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{H} \triangle \mathbf{x}
\end{aligned}
$$

$$
\begin{aligned}
\mathbf{b}^T &= \sum_i \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{J}_i\\
\mathbf{H} &= \sum_i \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{J}_i
\end{aligned}
$$

그렇다면 이제 squared error function의 마지막 식을 살펴보자.

$$
\begin{aligned}
F(\mathbf{x}+\triangle \mathbf{x}) &\approx
c + 2 \mathbf{b}^T \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{H} \triangle \mathbf{x}
\end{aligned}
$$

위의 식은 $$\triangle \mathbf{x}$$에 대해서 quadratic한 형태를 띄고 있다. quadratic form은 일반 2차방정식의 형태로 생각할 수 있다. 즉, 함수의 극대 혹은 극소값은 함수의 미분이 0이 되는 지점을 계산함으로써 계산할 수 있다.

그럼 이제 위의 식의 1차 미분을 계산한다. 일반적으로 quadratic form의 미분은 다음과 같다(matrix cookbook, section 2.2.4참고).

$$
\begin{aligned}
f(x) &= \mathbf{x}^T\mathbf{H}\mathbf{x} + \mathbf{b}^T\mathbf{x}\\
\frac{\partial f}{\partial x} &= (\mathbf{H} + \mathbf{H}^T)\mathbf{x} +\mathbf{b}
\end{aligned}
$$

따라서 함수의 1차 미분은 다음과 같다.

$$
\frac{\partial F(\mathbf{x}+\triangle \mathbf{x})}{\partial \triangle \mathbf{x}} = 2 \mathbf{b} + 2 \mathbf{H}\triangle \mathbf{x}
$$

#### 3. global error function의 최소값 계산

2번단계에서 global error function의 1차 미분값을 계산하였다. global error function은 quadratic form이므로 미분값이 0인 지점이 함수의 극대 혹은 극소값이다. 우리가 계산하고 있는 least square의 목적은 error를 최소로 만드는 state를 계산하는 것이므로 계속해서 error function의 최소값을 찾아 나간다. 최소값을 찾기 위하여 미분값을 0으로 만드는 $$\mathbf{x}$$를 계산한다.

$$
\frac{\partial F(\mathbf{x}+\triangle \mathbf{x})}{\partial \triangle \mathbf{x}} = 2 \mathbf{b} + 2 \mathbf{H}\triangle \mathbf{x} = 0
$$

따라서 optimal한 state는 다음과 같다.

$$
\triangle \mathbf{x}^* = -\mathbf{H}^{-1}\mathbf{b}
$$

이때 $$\mathbf{H}$$는 크기가 큰 matrix이므로 inverse를 계산 시 많은 계산이 소요된다. 이러한 문제를 해결하기 위해서 Cholesky factorization, 혹은 QR decomposition과 같은 방법을 사용한다. Cholesky factorization은 matrix $$\mathbf{H}$$를 하 삼각행렬(lower triangular matrix)의 곱으로 표현하는 방법이다($$\mathbf{H}= \mathbf{L}\mathbf{L}^T$$). Triangular matrix는 일반 matrix에 비해 inverse의 계산이 간단하므로 이렇게 분해함으로써 inverse를 더 빠르게 계산할 수 있다.

$$
\begin{aligned}
\triangle \mathbf{x}^* &= -\mathbf{H}^{-1}\mathbf{b}\\
&= -\mathbf{L}^{-T}\mathbf{L}^{-1}\mathbf{b}
\end{aligned}
$$

#### 4. State update

3번 과정에서 global error function을 최소로 만드는 state인 $$\triangle \mathbf{x}$$를 계산하였다. 이제 이 결과를 이용하여 state를 update한다.

$$
\mathbf{x} = \mathbf{x} + \triangle \mathbf{x}
$$

업데이트 완료되면 업데이트된 state를 기준으로 다시 선형화를 하여 위 과정을 반복한다. 여러번 반복함으로써 계속 최적화된 state로 업데이트하고, state가 수렴하면 계산을 중지하고 optimization과정을 종료한다.

위와 같이 quadratic form을 계산함으로써 optimization 문제를 해결하는 방법을 *Gauss-Newtom optimization* 이라고 한다.


### Least square와 Gaussian의 관계

위에서 least square의 계산과정은 모두 설명하였다. Least square문제는 아래의 식을 품으로써 최적의 state를 계산하였다.

$$
\begin{aligned}
x*
&=\text{argmin}_{\mathbf{x}} \sum_i \mathbf{e}_i^T(\mathbf{x}) \mathbf{\Omega}_i \mathbf{e}_i(\mathbf{x})
\end{aligned}
$$

하지만 왜 이런 형태의 식을 만들어서 풀게 되었을까? 이것은 Gaussian 분포와 관계가 있다. Bayes filter의 posterior의 식은 다음과 같다.

$$
p(x_{0:t} \mid z_{1:t}, u_{1:t}) = \eta p(x_0) \prod_t [p(x_t \mid x_{t-1}, u_t)p(z_t \mid x_t)]
$$

여기서 prior와, motion model 그리고 observation model은 모두 gaussian 분포를 따른다. 따라서 log likelihood를 계산하면 다음과 같다.

$$
log p(x_{0:t} \mid z_{1:t}, u_{1:t}) = \text{const.} +  log p(x_0) +  \sum_t [log p(x_t \mid x_{t-1}, u_t)+ log p(z_t \mid x_t)]
$$

여기서 모든 확률은 Gaussian 분포를 따르는데, Gaussian 분포의 log는 다음과 같다.

$$
log \mathcal{N}(x,\mu,\Sigma) = \text{const.} -\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)
$$

계산된 Gaussian의 log형태가 매우 익숙하다. 위의 형태는 우리가 최적화에 사용하였던 함수와 같은 형태임을 알수 있다. 따라서 posterior를 error function으로 표현하면 다음과 같다.

$$
\begin{aligned}
log p(x_{0:t} \mid z_{1:t}, u_{1:t}) &= \text{const.} +  log p(x_0) +  \sum_t [log p(x_t \mid x_{t-1}, u_t)+ log p(z_t \mid x_t)]\\
&= \text{const.} - \frac{1}{2} e_p(\mathbf{x}) -\frac{1}{2} \sum_t [e_{u_t}(\mathbf{x})+e_{z_t}(\mathbf{x})]
\end{aligned}
$$

따라서, posterior의 최대값을 찾는 문제는 error function의 최소값을 찾는 문제가 된다.

$$
\begin{aligned}
argmax \ \ log p(x_{0:t} \mid z_{1:t}, u_{1:t})
&= argmin \ \ e_p(\mathbf{x})+ \sum_t [e_{u_t}(\mathbf{x})+e_{z_t}(\mathbf{x})]
\end{aligned}
$$

이러한 Gaussian과의 관계에 의해서 위에서와 같이 error function이 정의됨을 기억하자.



**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
