---
layout: post
title: '[SLAM] Sparse Extended Information Filter(SEIF) SLAM 계산'
tags: [SLAM]
description: >
  Filter의 연산량 개선을 위한 Sparse Extended Information Filter(SEIF)를 이용한 SLAM의 계산과정을 설명한다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이번 글에서는 이전 글에서 설명한 Sparse Extended Information Filter(SEIF) SLAM의 과정을 차근차근 한단계씩 이해해 보려고 한다. SEIF에 대해서 설명한 [이전 글](http://jinyongjeong.github.io/2017/02/19/lec08_SEIF/)에서 언급했듯이 SEIF는 적은 갯수의 큰 값을 갖는 information matrix의 sparsification과정을 통해 filter의 일정한 계산속도(constant computation time)를 보장하는 방법이다. SEIF의 과정은 전체적으로 개념은 EIF와 유사하나, 각 단계마다 계산량을 줄이기위한, 즉 일정한 계산속도를 보장하기 위한 테크닉이 들어있다. 이는 large scale의 환경에서 SLAM을 구현하기 위한 꼭 필요한 테크닉이므로 정확히 이해하고 넘어가는 것이 좋다.

SEIF는 크게 4개의 과정으로 나뉘어 진다.

$$
\begin{aligned}
& \text{SEIF SLAM}(\xi_{t-1}, \Omega_{t-1}, \mu_{t-1}, u_t, z_t)\\
1: \ \ & \bar{\xi}_t, \bar{\Omega}_t, \bar{\mu}_t = \text{SEIF motion update}(\xi_{t-1}, \Omega_{t-1},\mu_{t-1}, u_t)\\
2: \ \ & \xi_t, \Omega_t = \text{SEIF measurement update}(\bar{\xi}_t, \bar{\Omega}_t, \bar{\mu}_t, z_t)\\
3: \ \ & \mu_t = \text{SEIF updated state estimate}(\xi_t, \Omega_t, \bar{\mu}_t)\\
4: \ \ & \tilde{\xi}_t, \tilde{\Omega}_t = \text{SEIF sparsification}(\xi_t, \Omega_t, \mu_t)\\
5: \ \ & \text{return} \ \ \tilde{\xi}_t, \tilde{\Omega}_t, \mu_t
\end{aligned}
$$

앞으로 각 단계를 하나씩 유도해 볼 것이며, 각 단계에서 constant한 computation time을 위해 어떤 테크닉을 사용하였는지 설명하도록 하겠다.

##  SEIF motion update

### matrix inversion lemma

sparse한 matrix의 inverse를 빠르게 계산하기 위해서 matrix inversion lemma를 이용한다. 이 lemma는 많은 부분에서 사용되니 기억해 두면 좋다.

$$
(R+PQP^T)^{-1} = R^{-1} - R^{-1}P(Q^{-1}+P^T R^{-1}P)^{-1}P^TR^{-1}
$$

위 식에서 R은 크기가 큰 matrix, Q는 크기가 작은 matrix이며 P가 low dimension matrix를 high dimension으로 변형하는 직사각 행렬이다. 이때 만약 큰 matrix인 R의 inverse를 간단하게 계산할 수 있으면 전체 matrix의 inverse도 쉽게 구할 수 있다.

### motion update step(prediction step)

* 목적: 이전에 iteration에서 추정된 $$\xi_{t-1}, \Omega_{t-1}, \mu_{t-1}$$와 control input을 이용하여 $$\bar{\xi}_{t-1}, \bar{\Omega}_{t-1}, \bar{\mu}_{t-1}$$를 계산한다.
* 방법: Information matrix의 sparseness를 활용하여 효율적으로 계산한다. Information matrix가 제한된 수의 non-zero off-diagonal을 갖는 sparse matrix로 가정한다.

<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/EKF.png" width="100%">

우선 EKF SLAM에서 prediction 단계를 다시 보도록 하자. velocity-based model의 경우 위와같이 정의할 수 있다.

<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/EKF2.png" width="100%">

편의를 위하여 vector와 matrix를 $$\delta, \triangle$$로 표현한다.

Prediction step에서의 information matrix를 계산하기 위해 EKF의 5번 식에서 부터 계산을 시작한다.

$$
\begin{aligned}
\bar{\Omega}_t  &= \bar{\Sigma}_t^{-1}\\
                &= [G_t\Omega_{t-1}^{-1}G_t^T + R_t]^{-1}\\
                &= [\Phi_t^{-1} + R_t]^{-1}
\end{aligned}
$$

우리가 최종적으로 계산하고자 하는 information matrix는 위와 같이 계산할 수 있다. prediction step에서 가장 큰 계산량을 차지하는 부분은 $$\Omega_{t-1}^{-1}$$을 계산하여 $$\bar{\Omega}_t$$를 계산하는 과정이다. 여기서 $$\Phi_t$$는 다음같이 정의된다.

$$
\begin{aligned}
\Phi_t &= [G_t \Omega_{t-1}^{-1} G_t^T]^{-1}\\
       &= [G_t^T]^{-1} \Omega_{t-1} G_t^{-1}
\end{aligned}
$$

이제 위에서 설명한 matrix inversion lemma를 이용하여 전개하면 다음과 같다.

$$
\begin{aligned}
\bar{\Omega}_t  &= [\Phi_t^{-1} + R_t]^{-1}\\
                &= [\Phi_t^{-1} + F_x^TR_t^x F_x]^{-1}\\
                &= \Phi_t - \Phi_t F_x^T({R_t^{x}}^{-1}+F_x \Phi_t F_x^T)^{-1} F_x \Phi_t\\
                &= \Phi_t - \kappa_t
\end{aligned}
$$

이제 우리가 prediction step에서 구하고자 하는 information matrix를 계산하는 식을 자세히 들여다 보도록 하자. $$F_x$$는 $$3\times3$$ block을 제외하고는 전부 0인 matrix이다. 그리고 inverse 계산을 해야하는 $${R_t^{x}}^{-1}+F_x \Phi_t F_x^T$$는 $$3\times3$$ matrix이므로 연산량이 많지 않다. 따라서 만약
$$\Phi_t$$의 computation time이 constant하다면 전체 process의 computation time도 constant해 진다.

그러면 이제 $$\Phi_t$$의 계산과정을 들여다 보도록 하자.

$$
\begin{aligned}
\Phi_t &= [G_t \Omega_{t-1}^{-1} G_t^T]^{-1}\\
       &= [G_t^T]^{-1} \Omega_{t-1} G_t^{-1}
\end{aligned}
$$

$$\Phi_t$$를 계산하는 과정에서 가장 계산량이 많은 부분은 $$G_t^{-1}$$을 계산하는 부분이다. 이 부분을 자세히 살펴보도록 하자.

$$
\begin{aligned}
G_t^{-1} &= (I+F_x^T \triangle F_x)^{-1}\\
         &= \begin{pmatrix} \triangle + I_3 & 0 \\ 0 & I_{2N}
         \end{pmatrix}^{-1}\\
         &= \begin{pmatrix} (\triangle + I_3)^{-1} & 0 \\ 0 & I_{2N}
         \end{pmatrix}
\end{aligned}
$$

즉 위 계산결과를 보면 $$G_t^{-1}$$의 계산속도에 영향을 주는 부분은 $$(\triangle + I_3)^{-1}$$이다. 이 부분은 전체 matrix의 크기에 상관없이 항상 $$3\times3$$ matrix이기 때문에 $$G_t^{-1}$$의 계산속도는 일정하다는 것을 알 수 있다. 위 식을 조금 더 정리하면 다음과 같다.

$$
\begin{aligned}
G_t^{-1} &= (I+F_x^T \triangle F_x)^{-1}\\
         &= \begin{pmatrix} \triangle + I_3 & 0 \\ 0 & I_{2N}
         \end{pmatrix}^{-1}\\
         &= \begin{pmatrix} (\triangle + I_3)^{-1} & 0 \\ 0 & I_{2N}
         \end{pmatrix}\\
         &= I_{3+2N} + \begin{pmatrix} (\triangle + I_3)^{-1} - I_3 & 0 \\ 0 & 0
         \end{pmatrix}\\
         &= I+ F_x^T[(I+\triangle)^{-1} - I]F_x\\
         &= I + \Psi_t
\end{aligned}
$$

위의 정의를 이용하여 $$\Phi_t$$를 정리해 보면

$$
\begin{aligned}
\Phi_t &= [G_t^T]^{-1} \Omega_{t-1} G_t^{-1}\\
       &= (I+\Psi_t^T) \Omega_{t-1} (I+\Psi_t)\\
       &= \Omega_{t-1} +\Psi_t^T \Omega_{t-1} +\Omega_{t-1}\Psi_t + \Psi_t^T \Omega_{t-1} \Psi_t\\
       &= \Omega_{t-1} + \lambda_t
\end{aligned}
$$

여기서 $$\lambda_t$$는 일정개수의 요소들만 non-zero의 값을 갖는 matrix이다.

따라서 $$G_t^{-1}$$의 계산속도가 constant하고 information matrix가 sparse하므로 $$\Phi_t$$의 계산속도는 constant하며, $$\Phi_t$$의 계산속도가 constant하므로 최종적으로 prediction step의 information matrix 계산량도 constant하다.

위에서 설명한 대로 일정한 계산속도를 유지하며 information matrix를 계산하는 과정을 정리하면 다음과 같다.

##### 1. $$\Psi_t$$ 계산
##### 2. $$\Psi_t$$를 이용하여 $$\lambda_t$$ 계산
##### 3. $$\lambda_t$$를 이용하여 $$\Phi_t$$ 계산
##### 4. $$\Phi_t$$를 이용하여 $$\kappa_t$$ 계산
##### 5. 마지막으로 $$\kappa_t$$와 $$\Phi_t$$를 이용하여 $$\bar{\Omega}_t$$ 계산

위의 계산 방법을 통해 matrix크기에 상관없이 constant한 계산속도로 information matrix를 계산할 수 있다.

이제 information matrix를 구했으니 information vector를 구해야 한다. information vector는 아래의 유도과정을 통해 계산할 수 있다.

<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/prediction_mean.png" width="100%">

Information vector를 계산하기 위해 사용되는 값들의 대부분이 Information matrix를 구할 때 계산된 값들이며, inverse의 계산이 없기 때문에 계산량이 크지 않다.

따라서 SEIF의 prediction step에서 information matrix가 sparse 할 때, constant computation time으로 information vector와 information matrix를 계산할 수 있음을 증명하였다.

## SEIF measurement update(correction step)

SEIF의 correction step은 EIF와 마찬가지로 계산량이 크지않다. SEIF의 measurement update과정은 다음과 같다.

<div>
<img align="left" src="/images/post/SLAM/lec09_SEIF_SLAM/measurement1.png" width="100%">
<img align="left" src="/images/post/SLAM/lec09_SEIF_SLAM/measurement2.png" width="100%">
<img align="left" src="/images/post/SLAM/lec09_SEIF_SLAM/measurement3.png" width="85%"></div><div style="clear:both;"></div>

SEIF의 measurement update과정은 EKF와 대부분 동일하다. SEIF에서 바뀐 부분은 information vector와 matrix를 계산하는 12, 13번 과정이다. 큰 matrix의 inverse와 같은 많은 계산량을 요구하는 부분이 없기 때문에 constant computation time을 보장한다($$Q_t^{-1}$$는 observation의 noise로 matrix의 크기가 작으며 미리 계산할 수 있는 matrix이다). 위의 계산과정이 이해가 되지 않는다면 이전 글인 [EKF SLAM](http://jinyongjeong.github.io/2017/02/16/lec05_EKF_SLAM/)을 참고하기 바란다.

## SEIF updated state estimate

SEIF의 세번째 단계는 state의 평균값(mean, $$\mu_{t}$$)을 계산하는 과정이다. 계산된 평균값은 다음의 과정에서 사용된다.

* Motion model의 선형화
* Measurement model의 선형화
* Sparsification 단계

이전 두 단계에서 information vector와 matrix를 계산하였기 때문에 mean값을 쉽게 계산할 수 있다.

$$
\mu = \Omega^{-1} \xi
$$

하지만 이와 같이 단순한 정의에 의해서 계산하게 되면 한가지 문제가 발생한다. $$\Omega$$는 매우 큰 matrix이므로 이 matrix의 inverse과정은 매우 많은 계산량을 요구한다.

따라서 mean값을 계산하기 위해 optimization 방법으로 접근한다.

$$
\begin{aligned}
\hat{\mu} &= \text{argmax} \ \ p(\mu)\\
          &= \text{argmax}_{\mu} \ \ exp(-\frac{1}{2} \mu^T\Omega\mu + \xi^T\mu)
\end{aligned}
$$

SEIF의 이전 두 단계에서 information matrix와 vector를 계산하였기 때문에 이를 이용하여 Gaussian 분포를 표현할 수 있다. 이 Gaussian의 분포에서 mean값은 함수의 최대값을 갖는 위치 이므로 위와 같이 optimization 방법으로 mean값을 계산하는데, 함수를 미분하여 미분값이 0이 되는 $$\mu$$를 구하면 된다.

Optimization 방법으로 mean값을 구하는 과정에서, 로봇의 위치와 active landmark의 mean값만을 구함으로써 더 효율적으로 계산 가능하고, constant computation을 보장할 수 있다.

## SEIF sparsification

마지막은 Information matrix의 sparsification 과정이다. sparsification은 matrix의 non-zero off-diagonal의 수가 일정하게 유지되도록, 즉 sparse matrix가 유지되도록 만드는 과정이다. SEIF의 이전 step에서 constant computation을 보장하기 위한 조건이 information matrix의 sparseness이었으므로 이 과정이 꼭 필요하다.

Sparsification은 [SEIF](http://jinyongjeong.github.io/2017/02/19/lec08_SEIF/)에서 언급했던것 처럼 로봇과 landmark간의 link을 제거함으로써(conditional independence하다고 가정함으로써) non-zero off-diagonal의 수를 유지하는 것이다.

[SEIF](http://jinyongjeong.github.io/2017/02/19/lec08_SEIF/)에 설명한 것 처럼 landmark는 2가지 종류로 분류할 수 있다. 현재 관측하거나 관측하고 있다고 생각하는 landmark(즉, direct link가 있는)는 active landmark이며, direct link가 없는 landmark를 passive landmark라고 한다.

sparsification 과정을 위해서 landmark를 다시 3가지로 분류한다. $$m^{+}$$는 현재 active landmark이며
계속 active landmark이다. $$m^0$$는 현재는 active landmark이지만 passive landmark로 만들 대상이다. $$m^-$$는 현재도 passive landmark이며 계속 passive landmark이다.

$$
\begin{aligned}
p(x_t,m \mid z^t, u^t) &= p(x_t, m^+, m^0, m^- \mid z^t, u^t)\\
                       &= p(x_t \mid m^+, m^0, m^-, z^t, u^t)p(m^+, m^0, m^- \mid z^t, u^t)\\
                       &= p(x_t \mid m^+, m^0, m^- = 0, z^t, u^t)p(m^+, m^0, m^- \mid z^t, u^t)
\end{aligned}
$$

위 식에서 마지막 단계에서 우리가 만약에 $$m^+$$과 $$m^0$$에 대해서 알고있다면 $$x_t$$는 $$m^-$$에 independent하다는 사실을 이용하였다. 따라서 $$m^-$$의 값을 임의의 값으로 설정할 수 있으며, 여기서는 단순하게 0으로 설정하였다. 이제 위 식으로 부터 $$m^0$$를 제거함으로써 근사화 시킬 수 있다. passive landmark가 되는 $$x^0$$는 이제 로봇의 위치인 $$x_t$$와 independent하기 때문에 제거 가능하다.

$$
\begin{aligned}
\tilde{p}(x_t,m \mid z^t, u^t) &= p(x_t \mid m^+, m^- = 0, z^t, u^t)p(m^0, m^+, m^- \mid z^t, u^t) \\
                              &= \frac{p(x_t, m^+ \mid m^- = 0, z^t, u^t)}{p(m^+ \mid m^- = 0, z^t, u^t)}p(m^0, m^+, m^- \mid z^t, u^t)
\end{aligned}
$$

$$\tilde{}$$는 sparsification을 통해 근사화를 했음을 의미한다. 이제 $$\tilde{p}(x_t,m \mid z^t, u^t)$$의 information matrix를 계산하는 과정이 sparsification 과정을 의미한다.

$$
\begin{aligned}
\Omega_t &= \text{information matrix of } p(x_t, m^+, m^0, m^- \mid z^t, u^t) \\
\Omega_t' &=  \text{information matrix of } p(x_t, m^+, m^0, \mid m^- = 0, z^t, u^t)
\end{aligned}
$$

맨 먼저 $$\Omega_t'$$를 계산하는 것으로 시작한다. bayes rule을 이용하여 전개하면 다음과 같다.

$$
p(x_t, m^+, m^0, \mid m^- = 0, z^t, u^t) = \frac{p(x_t, m^+, m^0, m^- \mid z^t, u^t)}{p(m^- = 0 \mid z^t, u^t)}
$$

따라서 $$\Omega_t'$$는 $$p(x_t, m^+, m^0, m^- \mid z^t, u^t)$$의 information matrix에서 $$m^-$$에 해당하는 information 항을 0으로 만든 information matrix이다(information form을 이용한 Gaussian 분포 식을 생각하자). 이 matrix를 projection matrix $$S$$를 이용하여 표현하면 다음과 같다.

$$
\Omega_t' = S_{x,m^+,m^0}S_{x,m^+,m^0}^T \Omega_t S_{x,m^+,m^0} S_{x,m^+,m^0}^T
$$

여기서 Projection marix는 남기고자 하는 부분이 identity로 구성된 matrix이다. 예를들어 로봇의 포즈와 첫번째 landmark만 남기고 모두 0으로 만드는 projection matrix는 다음과 같다.

$$
S_{x,m1} = \begin{bmatrix} I_3 & 0 & 0 & \cdots & 0\\0 & I_2 & 0 & \cdots & 0
\end{bmatrix}^T
$$

따라서 $$\Omega_t'$$을 계산함으로써 $$m^-$$에 해당하는 row와 column은 0이 된다. 이 부분이 잘 이해가 되지 않는다면 matlab을 이용해서 계산해보면 쉽게 이해할 수 있다.

이제 $$\Omega_t$$와 $$\Omega_t'$$을 이용하여
$$
\begin{aligned}
\tilde{p}(x_t,m \mid z^t, u^t) &= \frac{p(x_t, m^+ \mid m^- = 0, z^t, u^t)}{p(m^+ \mid m^- = 0, z^t, u^t)}p(m^0, m^+, m^- \mid z^t, u^t)
\end{aligned}
$$
을 구성하는 각 항의 information matrix를 구해본다.

$$
\begin{aligned}
\Omega_t^1 &= \text{information matrix of } p(x_t, m^+ \mid m^- = 0, z^t, u^t) \\
\Omega_t^2 &=  \text{information matrix of } p(m^+ \mid m^- = 0, z^t, u^t) \\
\Omega_t^3 &=  \text{information matrix of } p(m^0, m^+, m^- \mid z^t, u^t)
\end{aligned}
$$

각 information matrix는 information form의 marginalization을 이용하여 계산한다. information form의 marginalization은 [EIF](http://jinyongjeong.github.io/2017/02/19/lec07_EIF/)에 정리되어 있다. 이를 통해 각각의 information matrix를 계산하면 다음과 같다.

$$
\begin{aligned}
\Omega_t^1 & = \Omega_t' - \Omega_t' S_{m^0}(S_{m^0}^T \Omega_t' S_{m^0})^{-1}S_{m^0}^T \Omega_t'\\
\Omega_t^2 & = \Omega_t' - \Omega_t' S_{x_t, m^0}(S_{x_t, m^0}^T \Omega_t' S_{x_t, m^0})^{-1}S_{x_t, m^0}^T \Omega_t'\\
\Omega_t^3 & = \Omega_t - \Omega_t S_{x_t}(S_{x_t}^T \Omega_t S_{x_t})^{-1}S_{x_t}^T \Omega_t\\
\end{aligned}
$$

이제 계산된 information matrix를 이용하여 sparsification 된 information matrix를 계산할 수 있다.

$$
\begin{aligned}
\tilde{\Omega}_t &= \Omega_t^1 - \Omega_t^2 + \Omega_t^3\\
                 &=\Omega_t - \Omega_t' S_{m^0}(S_{m^0}^T \Omega_t' S_{m^0})^{-1}S_{m^0}^T \Omega_t' \\
                 & \ \ + \Omega_t' S_{x_t, m^0}(S_{x_t, m^0}^T \Omega_t' S_{x_t, m^0})^{-1}S_{x_t, m^0}^T \Omega_t' \\
                 & \ \ - \Omega_t S_{x_t}(S_{x_t}^T \Omega_t S_{x_t})^{-1}S_{x_t}^T \Omega_t
\end{aligned}
$$

이제 sparsification을 통해 근사화된 information matrix가 계산되었다. 이제 information vector를 계산해야 되는데 다음과 같이 간단하게 계산할 수 있다.

$$
\begin{aligned}
\tilde{\xi}_t &= \mu_t^T \tilde{\Omega}_t\\
              &= \mu_t^T(\Omega_t - \Omega_t +  \tilde{\Omega}_t)\\
              &= \mu_t^T\Omega_t + \mu_t^T(\tilde{\Omega}_t - \Omega_t )\\
              &= \xi_t + \mu_t^T(\tilde{\Omega}_t - \Omega_t )
\end{aligned}
$$

## SEIF SLAM vs EKF SLAM

* SEIF는 roughly constant time complexity, EKF는 quadratic complexity
* SEIF는 linear memory complexity, EKF는 quadratic memory complexity
* SEIF는 계산량의 이점을 위해 근사화를 하였기 때문에 EKF에 비해 덜 정확하다.

<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/comp_time_landmark.png" width="100%">
landmark의 갯수가 증가(matrix 크기 증가)함에 따른 계산시간의 변화를 보여준다. SEIF는 크게 변하지 않지만 EKF는 계산시간이 크게 증가한다.
<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/comp_byte_landmark.png" width="100%">
landmark의 갯수 증가에 따른 메모리 크기변화를 보여준다. SEIF는 선형적으로 증가하지만 EKF는 quadratic하게 증가한다.
<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/comp_error_landmark.png" width="100%">
landmark의 갯수 증가에 따른 error를 보여준다. 근사화를 하였기 때문에 SEIF가 EKF에 비해 error는 높다.
<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/update_time_feature.png" width="100%">
Active landmark의 갯수에 따른 update 시간을 비교하였다. Active landmark의 갯수가 적을수록 계산시간은 빨라진다.
<img align="middle" src="/images/post/SLAM/lec09_SEIF_SLAM/error_feature.png" width="100%">
Active landmark의 갯수에 따른 error의 변화를 보여준다. Active landmark의 갯수가 적을수록 error는 증가한다.


SEIF에 대해서 이해가 잘 되지 않거나 조금 더 깊이 보고 싶으시면 Thrun교수님의 논문인
[Simultaneous Localization and Mapping With Sparse Extended Information Filters](/images/post/SLAM/lec09_SEIF_SLAM/SEIF.pdf)를 참고하기 바란다.

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
