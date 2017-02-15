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

이 글에서는 velocity-based model을 이용하여 설명한다. 아래 그림은 velocity-based model을 보여준다. $$x,y,\theta$$는 로봇의 x,y좌표 및 방향을 의미하며, 로봇의 선속도는 $$v$$, 각속도는 $$\omega$$이다.

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

따라서 위에서 계산한 Jacobian matrix을 이용하여 EKF의 prediction step을 다음과 같이 계산할 수 있다.

$$
\bar{\Sigma_t} = G_t \Sigma_{t-1} G_t^T + V_t M_t V_t^T = G_t \Sigma_{t-1} G_t^T + R_t
$$

### Observation model

비선형 observation model을 EKF에 적용해 보기 위해서 가상의 로봇을 이용한다. 이 로봇은 3개의 센서를 갖고 있다. 첫번째 센서는 로봇의 위치에서 부터 landmark까지의 euclidean distance를 측정할 수 있다. 두번째, 세번째 센서는 landmark까지의 x방향의 거리와 y방향의 거리를 각각 측정할 수 있다. 로봇의 state는 $$\mathbf{\bar{x}_t} = \begin{bmatrix} \bar{x}_t \\ \bar{y}_t \\ \bar{\theta}_t \end{bmatrix}$$로 표시하며, landmark의 위치는 $$ \mathbf{m} = \begin{bmatrix} m_x\\m_y \end{bmatrix}$$라고 하자. 이때의 각 센서의 데이터입력 $$\mathbf{z}$$는 다음과 같다.

$$
\mathbf{z_t} =
\begin{bmatrix}
z_1\\z_2\\z_3
\end{bmatrix}
=
\begin{bmatrix}
\sqrt{(m_x - x_t)^2+(m_y - y_t)^2}\\
m_x - x_t\\
m_y - y_t
\end{bmatrix}
$$

따라서 위 비선형 observation model을 선형화 하기 위해서 Jacobian을 구하면 다음과 같다.

$$
H_t = \frac{\partial \mathbf{z_t}}{\partial \mathbf{\bar{x}_t}} =
\begin{pmatrix}
\frac{-m_x+\bar{x}_t}{\sqrt{(m_x-\bar{x}_t)^2 + (m_y-\bar{y}_t)^2}} & \frac{-m_y+\bar{y}_t}{\sqrt{(m_x-\bar{x}_t)^2 + (m_y-\bar{y}_t)^2}} & 0 \\
-1 & 0 & 0 \\
0 & -1 & 0
\end{pmatrix}
$$

따라서 위에서 계산한 Jacobian $$H_t$$를 이용하여 EKF의 correction step을 수행할 수 있다.
Observation model에서 $$Q_t$$는 measurement 노이즈로, 데이터를 얻는 센서의 부정확성으로 인해 발생한다. 따라서 observation model에서 $$Q_t$$는 센서의 uncertainty자체를 의미한다. 추가적으로 Jacobian matrix는 선형화 포인트에서만 유효하기 때문에 매 step마다 다시 계산해 주어야 한다는 점을 기억해야 한다.

다음 글은 EKF를 이용한 SLAM에 대해서 설명한다.

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
