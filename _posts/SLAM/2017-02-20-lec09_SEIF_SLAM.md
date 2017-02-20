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

EKF는 correction step의 계산량이 많지만, EIF는 prediction step의 계산량이 많다. 따라서 SEIF도 correction의 계산량은 많지 않다. SEIF의 measurement update과정은 다음과 같다.

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

SEIF의 이전 두 단계에서 information matrix와 vector를 계산하였기 때문에 이를 이용하여 Gaussian 분포를 표현할 수 있다. 이 Gaussian의 분포에서 mean값은 함수의 최대값을 갖는 위치 이므로 위와 같이 optimization 방법으로 mean값을 계산한다. 최대값을 계산하기 위해서 함수를 미분하여 미분값이 0이 되는 $$\mu$$를 구하면 된다.

따라서 위의 방법으로 mean값을 구하는 과정에서 로봇의 위치와 active landmark의 mean값만을 구함으로써 더 효율적으로 계산 가능하고, constant computation을 보장할 수 있다.

## SEIF sparsification















SEIF에 대해서 이해가 잘 되지 않거나 조금 더 깊이 보고 싶으시면 Thrun교수님의 논문인
[Simultaneous Localization and Mapping With Sparse Extended Information Filters](/images/post/SLAM/lec09_SEIF_SLAM/SEIF.pdf)를 참고하기 바란다.

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
