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

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

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

여기서 구별해야 할 점은 $$R_t$$는 input의 noise가 아닌 process의 noise이다. $$R_t$$는 control input에서 들어오는 Gaussian noise가 한번의 선형 변환을 거친 전체 state인 $$x_t$$의 noise 이므로 process noise라고 부른다. 이때 control input인 $$u_t$$의 covariance는 $$M_t$$라고 표기한다.

위의 선형 모델을 이용한 motion model과 observation model은 다음과 같다.

* Motion model

$$
p(x_t \mid u_t, x_{t-1}) = \frac{1}{\sqrt{det(2\pi R_t)}}e^{-\frac{1}{2}(x_t-A_t x_{t-1} - B_t u_t)^TR_t^{-1}(x_t-A_t x_{t-1} - B_t u_t)}
$$

* Observation model

$$
p(z_t \mid x_t) = \frac{1}{\sqrt{det(2\pi Q_t)}}e^{-\frac{1}{2}(z_t-C_t x_{t})^T Q_t^{-1}(z_t-C_t x_{t})}
$$

Motion model은 prediction step에서, observation model은 correction step에 적용된다.

* Prediction step

$$
\overline{bel}(x_t) = \int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t}) bel(x_{t-1}) dx_{t-1}
$$

* Correction step

$$
bel(x_t) = \eta p(z_t \mid x_t)\overline{bel}(x_t)
$$

t-1에서의 state의 확률은 motion model에 의해 t의 state의 확률이 결정되며(prediction step), prediction step에서 계산된 t에서의 state의 확률은 observation model에 의해서 보정된다. 이와같은 bayes filter식은 여러 확률 모델 분포를 모두 포함하고 있는 식이며, 그 중에서 모든 확률 분포를 Gaussian 확률 분포로 가정하는 모델이 Kalman filter이다. Gaussian으로 확률분포를 표현할 때, 간단히 평균(mean, $$\mu$$)와 분산(variance, $$\Sigma$$)으로 표현하기 때문에 두개의 파라미터만으로 분포를 표현할 수 있는 장점이 있다. Kalman filter 알고리즘은 다음과 같다.

$$
\begin{aligned}
1: & \text{Kalman filter}(\mu_{t-1}, \Sigma_{t-1}, u_t, z_t)\\
&[Prediction step]\\
2: & \ \ \bar{\mu}_t = A_t \mu_{t-1} + B_t u_t\\
3: &\ \ \bar{\Sigma_t} = A_t \Sigma_{t-1} A_t^T + R_t\\
&[Correction step]\\
4: &\ \ K_t = \bar{\Sigma_t}C_t^T(C_t \bar{\Sigma_t}C_t^T + Q_t)^{-1}\\
5: &\ \ \mu_t = \bar{\mu_t} + K_t(z_t - C_t \bar{\mu_t})\\
6: &\ \ \Sigma_t = (I - K_t C_t)\bar{\Sigma_t}\\
7: &\ \ \text{return} \ \ \mu_t, \Sigma_t\\
\end{aligned}
$$

위 식은 Kalman filter algorithm을 보여주고 있다. Kalman filter는 bayes filter이기 때문에 prediction과 correction의 두 단계로 이루어 지며, 다소 복잡해 보이지만 한단계씩 이해하면 어렵지 않다.

* Prediction step (Kalman filter)

첫번째 prediction 단계는 복잡하지 않다. 직관적으로 t-1의 평균은 motion model을 통해 t의 평균으로 계산되어 진다(2). 이때 $$\mu, \Sigma$$에 붙어있는 bar($$\bar{\mu}, \bar{\Sigma}$$)는 prediction step임을 의미한다. 그 다음으로 covariance는 위에서 설명한 Gaussian linear transformation의 의해 계산되어 진다(3). 이때 $$R_t$$는 process noise 이며, control input($$u_t$$)의 covariance가 $$M_t$$일 때 $$R_t = B_t M_t B_t^T$$이다. 일반적인 로봇시스템이나 자동차 시스템에서 control input은 wheel encoder로 부터 얻어지는 odometry 정보를 많이 이용하며, encoder 센서의 uncertainty가 $$M_t$$가 된다.

* Correction step (Kalman filter)

Correction 단계에서는 새로운 변수인 K(Kalman gain)이 추가된다. K는 현재 관측 데이터($$z_t$$)의 정확도에 따라 predicted state와 관측된 state(observation model을 이용하여 관측값으로부터 추정된 state)의 보정 비율을 결정하는 역활을 한다. 이때 $$Q_t$$가 observation의 covariance이다. 5번 식에서 ($$z_t - C_t\bar{\mu_t}$$)는 현재 실제로 관측된 데이터($$z_t$$)와 현재 위치로 예상되는 위치($$\bar{\mu_t}$$)에서 기대되는 관측값($$C_t\bar{\mu_t}$$)과의 차이를 Kalman gain(K)의 크기만큼 보정함으로써, 최종 Gaussian의 평균을 계산한다. 쉬운 이해를 돕기 위해 예를 들어보자. 우리의 로봇은 로봇의 위치에서 부터 주변 lanemark까지의 거리를 측정할 수 있는 로봇이라고 하자. 만약 encoder data와 motion model에 의해서 예상되는 로봇의 위치를 알고 있고, 주변의 lanemark의 위치를 이미 알고 있을 때, 예상되는 lanemark까지의 거리를 계산할 수 있다. 만약 이 예상되는 거리가 10m인데, 실제 lanemark까지의 거리가 9m로 측정이 된다면, 두 값이 오차인 1m는 odometry sensor로 부터 발생한 것일 수 도 있고, 거리를 측정하는 센서로 부터 발생한 것일 수도 있다. 이때 Kalman gain은 10m와 9m중 어느 데이터를 더 신뢰할지를 결정하는 파라미터로 볼 수 있다.

조금 더 쉽게 이해하기 위해 observation의 covariance인 $$Q_t$$가 무한대라고 해보자. covariance가 무한대라는 의미는 거리측정 센서로부터 얻어진 데이터는 전혀 신뢰 할 수 없다는 것을 의미한다. 이때 K는 0이 되며, $$\mu_t = \bar{\mu_t}$$가 된다. 즉, 관측된 센서 데이터는 신뢰할 수 없으므로, 예측된 로봇의 위치를 전적으로 신뢰하겠다는 것이다. 반대로 $$Q_t$$가 0라고 해보자. Covariance가 0이라는 의미는 센서 데이터를 100% 신뢰할 수 있음을 의미한다. 따라서 $$Q_t$$가 0이라면 $$K = C_t^{-1}$$이 되며, 5번식은 $$\mu_t = C_t^{-1} z_t$$가 된다. 즉 로봇의 거리 측정센서로 부터 얻어진 데이터($$z_t$$)를 전적으로 신뢰하여, 이로부터 로봇의 state를 추정하겠다는 의미이다. 6번식 covariace를 계산하는 부분도 이와 마찬가지로 $$Q_t$$가 무한대 일때는 최종 covariace는 prediction의 covariance를 그대로 사용하며, $$Q_t$$가 0일때는 관측데이터가 100%신뢰할 수 있음을 의미하므로 covariace는 0이 된다.

<img align="middle" src="/images/post/SLAM/lec03_kalman_filter_and_EKF/kalman_fig.png" width="700">

위 그림은 Kalman filter의 과정을 그림으로 표현하였다. 빨간색은 그래프는 prediction step에서 계산한 state의 Gaussian, 초록색은 observation으로 추정한 state의 Gaussian 분포이다. Kalman filter algorithm의 계산에 의해 두 Gaussian분포는 파란색의 최종 Gaussian 분포로 state가 결정된다. 이때 초록색 Gaussian의 variance가 빨간색보다 작기 때문에, 최종 결과는 measurement에 더욱 dominant하다.

### Extended Kalman Filter (EKF)

* KF와 마찬가지로 노이즈는 평균(mean)이 0인 Gaussian 분포로 가정한다.
* KF와의 차이점은 motion 모델과 observation 모델을 선형으로 가정하지 않고 비선형 함수로 확장한 것이다.
* 거의 대부분의 실제 시스템은 비선형이다.

Extended Kalman Filter는 아래와 같이 기존 KF의 선형 모델을 비선형 함수인 $$g(u_t,x_{t-1})$$와 $$h(x_t)$$로 바꿈으로써 비선형으로 확장한 모델이다.

$$
\begin{aligned}
x_t &= g(u_t, x_{t-1}) + \epsilon_t &\leftarrow &x_t = A_t x_{t-1} + B_t u_t + \epsilon_t\\
z_t &= h(x_t) + \delta_t            &\leftarrow &z_t = C_t x_t + \delta_t
\end{aligned}
$$

하지만 motion 모델과 observation 모델을 비선형으로 확장한 경우 문제가 발생한다. 다음 그림은 이러한 문제를 보여준다.

<div style="width:43%; float:left; margin-right:3px;">
<img align="left" src="/images/post/SLAM/lec03_kalman_filter_and_EKF/linear.png">
</div>
<div style="width:54%; float:left;">
<img align="left" src="/images/post/SLAM/lec03_kalman_filter_and_EKF/non_linear.png">
</div><div style="clear:both;"></div>

많은 문제에서 Gaussian 분포를 사용하는 이유는 평균(mean)과 분산(variance) 두개의 파라미터로 분포를 표현함과 동시에 데이터들의 분포를 정확히 반영할 수 있기 때문이다. 따라서 반복적인 계산을 통해 state를 추정하는 문제에서 입력이 Gaussian 분포일 때 출력 또한 Gaussian 분포이여야 한다. 왼쪽 그림은 선형 시스템에서의 입력과 출력을 보여준다. 선형 시스템이기 때문에 입력이 Gaussian 분포일 때 출력 또한 Gaussian 분포가 된다. 하지만 오른쪽 그림과 같이 비선형 시스템의 경우, 입력은 Gaussian 분포이지만 시스템의 비선형성에 의해 출력은 Gaussian 분포가 아니다. 따라서 이런 경우 출력을 평균과 분산으로 표현 할 수 없다. 이러한 문제를 풀기 위해서는 비선형함수를 선형화(Linearization) 시키는 과정이 필요하다.

##### 선형화(Linearization)

EKF에서 비선형 함수를 선형화 시키기 위해서는 1차 Taylor 근사법(First order Talyer Expansion)을 사용한다. 선형 근사화된 model은 다음과 같다.

* Motion model

$$
g(u_t, x_{t-1}) \approx g(u_t,\mu_{t-1}) + \frac{\partial g(u_t, \mu_{t-1})}{\partial x_{t-1}}(x_{t-1} - \mu_{t-1})
$$

* Observation model

$$
h(x_t) \approx h(\bar{\mu_t}) + \frac{\partial h(\bar{\mu_t})}{\partial x_t} (x_t - \bar{\mu_t})
$$

이떄 비선형 함수들을 state로 편미분하여 matrix를 생성하는데 이 matrix를 **Jacobian** 이라고 부르며, 두 matrix는 $$G_t = \frac{\partial g(u_t, \mu_{t-1})}{\partial x_{t-1}}$$, $$H_t = \frac{\partial h(\bar{\mu_t})}{\partial x_t}$$로 표기한다.

##### Jacobian matrix

* Jacobian matrix는 non-square matrix이다.
* 비선형 함수 vector가 $$g(x)$$일 때 Jacobian $$G_x$$는 다음과 같이 계산된다.

$$
g(x) =
\begin{bmatrix}
g_1(x)\\
g_2(x)\\
\vdots\\
g_m(x)
\end{bmatrix}
$$

$$
G_x =
\begin{bmatrix}
\frac{\partial g_1}{\partial x_1} & \frac{\partial g_1}{\partial x_2} & \cdots & \frac{\partial g_1}{\partial x_n}\\
\frac{\partial g_2}{\partial x_1} & \frac{\partial g_2}{\partial x_2} & \cdots & \frac{\partial g_2}{\partial x_n}\\
\vdots & \vdots & & \vdots\\
\frac{\partial g_m}{\partial x_1} & \frac{\partial g_m}{\partial x_2} & \cdots & \frac{\partial g_m}{\partial x_n}
\end{bmatrix}
$$

아래 그림은 Talyer 근사화를 통해 선형화를 하였을 때의 특징을 보여준다.

<div style="width:48%; float:left; margin-right:3px;">
<img align="left" src="/images/post/SLAM/lec03_kalman_filter_and_EKF/large_variance.png">
</div>
<div style="width:48%; float:left;">
<img align="left" src="/images/post/SLAM/lec03_kalman_filter_and_EKF/small_variance.png">
</div><div style="clear:both;"></div>

왼쪽그림은 입력의 분산(variance)가 큰 경우를 보여주며, 오른쪽 그림은 분산이 작은 경우를 보여준다. 분산이 큰 경우 실제 비선형 함수 출력의 평균값과 선형화를 통해 계산된 평균값의 차이가 큰 것을 알 수 있다. 반면 분산이 작은 경우는 선형화를 통해 계산된 평균값이 실제 평균값과 유사함을 알 수 있다. 따라서 선형화 시 선형화 지점으로 부터 멀수록(분산이 클수록) 실제 함수를 반영하지 못한다.

##### EKF algorithm

선형화된 motion 모델과 observation 모델을 이용한 bayes filter는 다음과 같다.

* linearized prediction model

$$
p(x_t \mid u_t, x_{t-1}) \approx \frac{1}{\sqrt{det(2\pi R_t)}}e^{-\frac{1}{2}(x_t-g(u_t, \mu_{t-1}) - G_t(x_{t-1} - \mu_{t-1}))^TR_t^{-1}(x_t-g(u_t, \mu_{t-1}) - G_t(x_{t-1} - \mu_{t-1}))}
$$

* linearized correction model

$$
p(z_t \mid x_t) \approx \frac{1}{\sqrt{det(2\pi Q_t)}}e^{-\frac{1}{2}(z_t-h(\bar{\mu_t})-H_t (x_t-\bar{\mu_t}))^T Q_t^{-1}(z_t-h(\bar{\mu_t})-H_t (x_t-\bar{\mu_t}))}
$$

KF와 마찬가지로 $$R_t, Q_t$$는 process noise와 measurement noise이다. EKF 알고리즘은 다음과 같다.

$$
\begin{aligned}
1: & \text{Extended Kalman filter}(\mu_{t-1}, \Sigma_{t-1}, u_t, z_t)\\
&[Prediction step]\\
2: & \ \ \bar{\mu}_t = g(u_t, \mu_{t-1})\\
3: &\ \ \bar{\Sigma_t} = G_t \Sigma_{t-1} G_t^T + R_t\\
&[Correction step]\\
4: &\ \ K_t = \bar{\Sigma_t}H_t^T(H_t \bar{\Sigma_t}H_t^T + Q_t)^{-1}\\
5: &\ \ \mu_t = \bar{\mu_t} + K_t(z_t - h(\bar{\mu_t}))\\
6: &\ \ \Sigma_t = (I - K_t H_t)\bar{\Sigma_t}\\
7: &\ \ \text{return} \ \ \mu_t, \Sigma_t\\
\end{aligned}
$$

EKF 알고리즘과 KF 알고리즘의 차이는 KF에서 선형함수를 통해 평균($$\mu$$)를 구하는 2,5번 식에서 선형함수 대신 비선형 함수가 사용되었다. 그리고 3,4번 식에서 선형함수의 $$A_t, C_t$$ Matrix는 Jacobian matrix인 $$G_t, H_t$$로 수정되었다. 여기서 $$R_t$$는 process noise이며, control input의 covariance matrix가 $$M_t$$일 때 $$R_t = V_t M_t V_t^T$$이다. 여기서 $$V_t$$는 $$g(u_t,\mu_{t-1})$$를 control input인 $$u_t$$로 편미분한 Jacobian이다.

여기까지 EKF에 대한 설명을 마친다. 다음 글에서는 실제 robot의 모델을 통해 EKF를 이해해보자.

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
