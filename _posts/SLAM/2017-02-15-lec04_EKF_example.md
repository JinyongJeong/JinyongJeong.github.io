---
layout: post
title: '[SLAM] Extended Kalman Filter(EKF) 예제'
tags: [SLAM]
description: >
  Robot의 실제 모델을 이용하여 EKF를 설명.

sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이번 글에서는 이전 글에서 설명한 Extended Kalman Filter(EKF)를 실제 모델을 이용해서 설명한다. 이전 글을 통해 EKF의 선형화 과정과 bayes filter 과정을 이해했지만 실제 어떻게 적용을 하는지에 대해서는 이해가 잘 되지 않을 수 있다. 이 글에서는 velocity motion model과 observation model을 이용하여 EKF과정을 설명한다.

### Motion model

Robot의 Motion model은 크게 두가지로 나뉜다.

* Odometry-based model
* Velocity-based model

이 글에서는 velocity-based model을 이용하여 설명한다. 아래 그림은 velocity-based model을 의미한다. $$x,y,\theta$$는 로봇의 x,y좌표 및 방향을 의미하며, 로봇의 선속도는 $$v$$, 각속도는 $$\omega$$이다.

<img align="middle" src="/images/post/SLAM/lec04_EKF_example/velocity_model.png" width="100%">

이때 로봇의 상태(state)는 $$\mathbf{x_t} = \begin{bmatrix} x \\ y \\ \theta \end{bmatrix}$$이며, control input은 $$u_t = \begin{bmatrix} v \\ \omega \end{bmatrix}$$이다. 실제 로봇을 이동시키기 위해서는 모터를 구동시켜야 하기 때문에 로봇에 들어가는 실제 입력은 모터를 구동하기 위한 제어값이다. 하지만 여기서 control input은 로봇이 얼마나 이동했는지를 측정하기 위한 센서값을 의미한다. Velocity-based model에서 motion model은 다음과 같다.

$$
\begin{bmatrix}
x_{t}\\y_{t}\\ \theta_{t}
\end{bmatrix}
=
\begin{bmatrix}
x_{t-1} - \frac{\hat{v_t}}{\hat{\omega_t}} \sin \theta_{t-1} + \frac{\hat{v_t}}{\hat{\omega_t}}\sin(\theta_{t-1} + \hat{\omega_t}\vartriangle t)\\
y_{t-1} - \frac{\hat{v_t}}{\hat{\omega_t}} \cos \theta_{t-1} - \frac{\hat{v_t}}{\hat{\omega_t}}\cos(\theta_{t-1} + \hat{\omega_t}\vartriangle t)\\
\theta_{t-1} + \hat{\omega_t} \vartriangle t
\end{bmatrix}
$$

즉 velocity-based model은 비선형 함수로 정의된다. 위 식에서 $$\hat{}$$은 노이즈를 포함한 control input을 의미한다. control input의 covariance가 $$M_t$$일 때 다음과 같이 노이즈를 분리할 수 있다.

$$
\begin{bmatrix} \hat{v} \\ \hat{\omega} \end{bmatrix}
=
\begin{bmatrix} v \\ \omega \end{bmatrix}+\mathcal{N}(0,M_t)
$$

따라서 노이즈 항을 따로 분리하면 다음과 같이 다시 쓸 수 있다.

$$
\begin{bmatrix}
x_{t}\\y_{t}\\ \theta_{t}
\end{bmatrix}
=
\begin{bmatrix}
x_{t-1} - \frac{v_t}{\omega_t} \sin \theta_{t-1} + \frac{v_t}{\omega_t}\sin(\theta_{t-1} + \omega_t\vartriangle t)\\
y_{t-1} + \frac{v_t}{\omega_t} \cos \theta_{t-1} - \frac{v_t}{\omega_t}\cos(\theta_{t-1} + \omega_t\vartriangle t)\\
\theta_{t-1} + \omega_t \vartriangle t
\end{bmatrix} + \mathcal{N}(0,R_t)
$$

여기서 $$R_t$$는 process noise로 control input의 uncertainty에 의해 발생하며, $$R_t$$에 대해서는 뒤에서 다시 자세히 설명한다.

이제 motion model을 알고 있으므로 앞에서 설명한 Jacobian matrix를 이용하여 선형화된 model을 계산할 수 있다.

$$
\mathbf{x_t} = G_t \mathbf{x_{t-1}} + V_t \mathbf{u_t}
$$

위 식에서 $$G_t, V_t$$는 각각 $$\mathbf{x_{t-1}}$$와 $$\mathbf{u_t}$$로 편미분을 통해 계산한 Jacobian matrix이다.

$$
G_t = \frac{\partial g(u_t,\mu_{t-1})}{\partial \mathbf{x_{t-1}}} =
\begin{pmatrix}
1 & 0 & -\frac{v_t}{\omega_t}\cos \theta_{t-1} + \frac{v_t}{\omega_t}\cos(\theta_{t-1}+\omega_t \vartriangle t)\\
0 & 1 & -\frac{v_t}{\omega_t}\sin \theta_{t-1} + \frac{v_t}{\omega_t}\sin(\theta_{t-1}+\omega_t \vartriangle t)\\
0 & 0 & 1
\end{pmatrix}
$$

$$
V_t = \frac{\partial g(u_t,\mu_{t-1})}{\partial \mathbf{u_{t}}} =
\begin{pmatrix}
\frac{-\sin \theta_{t-1} + \sin (\theta_{t-1}+\omega_t \vartriangle t)}{\omega_t} & \frac{v_t(\sin \theta_{t-1} - \sin (\theta_{t-1}+\omega_t \vartriangle t))}{\omega_t^2} + \frac{v_t \vartriangle t \cos (\theta_{t-1} + \omega_t \vartriangle t)}{\omega_t}\\
\frac{\cos \theta_{t-1} - \cos (\theta_{t-1}+\omega_t \vartriangle t)}{\omega_t} &
\frac{v_t(-\cos \theta_{t-1} + \cos (\theta_{t-1}+\omega_t \vartriangle t))}{\omega_t^2} + \frac{v_t \vartriangle t \sin (\theta_{t-1} + \omega_t \vartriangle t)}{\omega_t}
\end{pmatrix}
$$




### Observation model


**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
