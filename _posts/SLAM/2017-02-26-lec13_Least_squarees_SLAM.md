---
layout: post
title: '[SLAM] Graph-based SLAM (Pose graph SLAM)'
tags: [SLAM]
description: >
  Graph SLAM에 대해서 설명한다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이 글에서는 Least square를 이용하여 실제로 어떻게 SLAM에 적용이 되는지 알아볼 것이다. 아직 least squre에 대해서 익숙하지 않다면 이전 글인 [least squre(최소자승법)](http://jinyongjeong.github.io/2017/02/26/lec12_Least_squarees/)을 참고하기 바란다. 이번 글에서의 표기법은 이전글의 표기법을 그대로 사용한다.

## Graph-based SLAM

Graph-based SLAM은 로봇의 위치와 움직임을 graph처럼 node와 edge로 표현함을 의미한다.

* Node: Graph의 node는 로봇의 pose를 의미한다.
* Edge: 두 node사이의 edge는 로봇의 위치 사이의 odometry정보이며 constraint라고 한다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/graph1.png" width="100%">

로봇이 움직이며 odometry 정보를 이용하여 node와 constraint를 생성한다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/graph2.png" width="100%">

로봇이 위 그림과 같이 이전에 방문했던 지역을 다시 방문할 경우 주변 환경에 대한 정보를 이용하여 같은 위치임을 인식하고 연속적이지 않은(non-successive) node사이에 constraint를 추가하고, graph를 최적화 함으로써 measurement에 최적화된 로봇의 위치를 계산할 수 있다.

아래의 youtube 동영상은 pose graph SLAM을 이용한 SLAM의 예이다.

<iframe width="640" height="480" src="https://www.youtube.com/embed/JLjFxDMovnc" frameborder="0" allowfullscreen> </iframe>

동영상에서 차량이 움직이는 경로의 실선은 constraint이며, 실선 중간에 생성되는 네모는 graph의 node이다. 위 application은 도로 상의 marking정보를 이용하여 장소를 인식하고 non-successive node사이의 constraint를 추가함으로써 odometry의 error를 보정한다. 동영상에서 이러한 constraint를 추가함으로써 계속적으로 graph가 최적화 되는 모습을 볼 수 있다.

## Overall SLAM system

Graph SLAM은 아래 그림과 같이 크게 Front-end와 Back-end로 나눌 수 있다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/SLAM_overall.png" width="100%">

위의 동영상을 예로 들면, Front-end는 camera로 부터 얻어지는 이미지로 부터 데이터 처리를 통해 sub-map을 만들고, 생성된 sub-map을 통해서 다시 방문한 장소를 인식함으로써 각 node사이의 constraint를 생성하는 과정을 말한다. 이렇게 생성된 constraint들과 node들의 정보를 이용하여 최적화된 node의 위치를 계산하는 과정은 graph SLAM의 back-end에서 수행한다. 최근 많이 사용되는 graph SLAM의 back-end로는 [iSAM](http://ieeexplore.ieee.org/abstract/document/4682731/)과 [g2o](http://ieeexplore.ieee.org/abstract/document/5979949/)가 있다.


## Graph

이 글에서 로봇이 이동하는 환경은 2-dimension 으로 가정하며, 로봇의 위치는 다음과 같이 3 DOF로 표현한다.

$$
\mathbf{x}_i = (x_i, y_i, \theta_i)
$$

Graph는 이러한 로봇의 위치를 표현하는 $$\mathbf{x}_i$$로 구성된 vector이다.

$$
\mathbf{x} = \mathbf{x}_{1:n}
$$

Graph를 구성하는 노드 $$\mathbf{x}_i$$와 $$\mathbf{x}_{i+1}$$ 사이에는 odometry 센서로부터 얻어진 measurement가 constraint/edge로 생성된다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/odometry.png" width="100%">

만약 로봇이 같은 장소를 다시 방문했을 때 두 로봇의 위치에서 얻은 measurement를 이용하여 두 노드 사이의 상대적인 위치(relative pose)를 계산할 수 있다. 아래 그림은 다른 위치에서 획득한 같은 물체의 센서 데이터를 보여준다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/virtual.png" width="100%">

두 센서데이터의 매칭을 통해서 두 node사이의 상대위치를 계산할 수 있고, 이 상대위치가 두 node사이의 constraint가 된다. 이때 직접적으로 node사이의 관계를 구한 것이 아닌 센서 measurement로부터 계산하였기 때문에 virtual measurement라고 부른다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/virtual2.png" width="100%">


## Pose graph

아래 그림은 Pose graph SLAM에서 measurement에 대한 error를 그림으로 표현하여 보여준다.  

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/pose_graph.png" width="100%">

$$\mathbf{x}_i$$와 $$\mathbf{x}_j$$는 현재 graph에서 두 node의 위치이다. $$<\mathbf{z}_{ij},\Omega_{ij}>$$는 $$\mathbf{x}_i$$의 위치에서 센서를 이용하여 측정한 $$\mathbf{x}_j$$의 위치이다. 센서로 측정한 node의 위치와 현재 graph상의 위치의 차이를 $$\mathbf{e}_{ij}$$로 표현한다. Graph optimization은 이러한 error를 최소화 시키는 graph를 계산하는 것이다. Graph optimization을 계산하는 과정은 이전 글인 [least square](http://jinyongjeong.github.io/2017/02/26/lec12_Least_squarees/)을 참고하기 바란다. 여기서 $$\Omega_{ij}$$가 센서 measurement의 information matrix임을 기억하자.

Error function ($$\mathbf{e}_{ij}(\mathbf{x}_i, \mathbf{x}_j)$$)을 homogeneous coordinate로 표현하면 다음과 같이 표현할 수 있다.

$$
\mathbf{e}_{ij}(\mathbf{x}_i, \mathbf{x}_j) = \text{t2v}(\mathbf{Z}_{ij}^{-1}(\mathbf{X}_i^{-1}\mathbf{X}_j))
$$

homogeneous coordinate은 로봇의 translation과 rotation을 하나의 matrix로 표현하는 방법이며, 이러한 표현방법에 대해서는 [SE(3) and SO(3)](http://jinyongjeong.github.io/2016/06/07/se3_so3_transformation/)에서 다루었다.

위 식에서 $$\mathbf{Z}_{ij}$$는 i에서 바라본 j의 measurement이며, $$\mathbf{X}_i^{-1}\mathbf{X}_j$$는 현재 graph에서 i를 기준으로 j의 위치를 의미한다. t2v함수는 homogeneous coordinate를 vector form으로 바꾸는 transform 함수이다.

Graph optimization식을 전체 state를 표현하는 $$\mathbf{x}$$를 이용하여 표현하면 다음과 같다.

$$
\begin{aligned}
x^* &=\text{argmin}_{\mathbf{x}} \sum_{ij} \mathbf{e}_{ij}^T(\mathbf{x}_i,\mathbf{x}_j) \mathbf{\Omega}_{ij} \mathbf{e}_{ij}(\mathbf{x}_i,\mathbf{x}_j)\\
&=\text{argmin}_{\mathbf{x}} \sum_k \mathbf{e}_k^T(\mathbf{x}) \mathbf{\Omega}_k \mathbf{e}_k(\mathbf{x})
\end{aligned}
$$

여기서 state vector $$\mathbf{x}$$는 graph의 node가 각각의 block을 구성하는 vector이다.

$$
\mathbf{x}^T = \begin{pmatrix}\mathbf{x}_1^T & \mathbf{x}_2^T & \cdots & \mathbf{x}_n^T
\end{pmatrix}
$$

[이전 글](http://jinyongjeong.github.io/2017/02/26/lec12_Least_squarees/)에서 설명한 것처럼 error function을 선형화하면 Jacobian으로 표현할 수 있다.

$$
\begin{aligned}
\mathbf{e}_{ij}(\mathbf{x}+\triangle\mathbf{x}) &\approx
\mathbf{e}_{ij}(\mathbf{x}) + \mathbf{J}_{ij}(\mathbf{x})\triangle \mathbf{x}
\end{aligned}
$$

선형화된 error function $$\mathbf{e}_{ij}$$는 state vector의 $$\mathbf{x}_i$$와 $$\mathbf{x}_j$$에만 관련이 있으므로 Jacobian은 sparse matrix가 된다.

$$
\frac{\partial \mathbf{e}_{ij}(\mathbf{x})}{\partial \mathbf{x}} = \begin{pmatrix}
0 & \cdots & \frac{\partial \mathbf{e}_{ij}(\mathbf{x}_i)}{\partial \mathbf{x}_i} & \cdots && \frac{\partial \mathbf{e}_{ij}(\mathbf{x}_j)}{\partial \mathbf{x}_j} & \cdots & 0
\end{pmatrix}
$$

$$
\mathbf{J}_{ij} = \begin{pmatrix} 0 & \cdots & \mathbf{A}_{ij} & \cdots & \mathbf{B}_{ij} & \cdots & 0)
\end{pmatrix}
$$

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/jacobian.png" width="100%">

[이전 글](http://jinyongjeong.github.io/2017/02/26/lec12_Least_squarees/)에서 optimization 과정을 통해서 최적화된 $$\mathbf{x}^* $$를 계산하였다. 최적화 식을 만족하는 state를 계산하기 위해서는 $$\mathbf{b}^T$$와 $$\mathbf{H}$$를 계산해야 한다.

$$
\begin{aligned}
\mathbf{b}^T &= \sum_{ij} \mathbf{e}_{ij}^T \mathbf{\Omega}_{ij} \mathbf{J}_{ij}\\
\mathbf{H} &= \sum_{ij} \mathbf{J}_{ij}^T  \mathbf{\Omega}_{ij} \mathbf{J}_{ij}
\end{aligned}
$$

Jacobian matrix $$\mathbf{J}_{ij}$$가 sparse matrix이기 때문에 information matrix인 $$\mathbf{H}$$도 sparse matrix가 된다. Jacobian matrix의 sparse함이 각 파라미터에 어떻게 영향을 미치는지 그림으로 보자.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/jacobian_effect.png" width="100%">


<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/jacobian_effect2.png" width="100%">

위 그림에서 볼 수 있듯이 Jacobian이 sparse하기 때문에 information matrix인 $$\mathbf{H}$$도 sparse해 짐을 알 수 있다. 식으로 표현하면 다음과 같다.

$$
\begin{aligned}
\mathbf{b}_{ij}^T &=  \mathbf{e}_{ij}^T \mathbf{\Omega}_{ij} \mathbf{J}_{i}\\
&=  \mathbf{e}_{ij}^T \mathbf{\Omega}_{ij} \begin{pmatrix} 0 & \cdots & \mathbf{A}_{ij} & \cdots & \mathbf{B}_{ij} & \cdots & 0
\end{pmatrix}\\
&=   \begin{pmatrix} 0 & \cdots & \mathbf{e}_{ij}^T \mathbf{\Omega}_{ij}\mathbf{A}_{ij} & \cdots & \mathbf{e}_{ij}^T \mathbf{\Omega}_{ij}\mathbf{B}_{ij} & \cdots & 0
\end{pmatrix}\\
\end{aligned}
$$

$$
\mathbf{b} = \sum_{ij} \mathbf{b}_{ij}^T
$$


$$
\begin{aligned}
\mathbf{H}_{ij} &= \mathbf{J}_{ij}^T  \mathbf{\Omega}_{ij} \mathbf{J}_{ij}\\
&= \begin{pmatrix} 0 & \cdots & \mathbf{A}_{ij} & \cdots & \mathbf{B}_{ij} & \cdots & 0
\end{pmatrix}^T \mathbf{\Omega}_{ij} \begin{pmatrix} 0 & \cdots & \mathbf{A}_{ij} & \cdots & \mathbf{B}_{ij} & \cdots & 0
\end{pmatrix}\\
&=
\begin{pmatrix}
0 & \cdots & \cdots & \cdots & \cdots &\cdots & 0\\
0 & \cdots & \mathbf{A}_{ij}^T \mathbf{\Omega}_{ij}\mathbf{A}_{ij} & \cdots &  \mathbf{A}_{ij}^T \mathbf{\Omega}_{ij}\mathbf{B}_{ij} & \cdots & 0\\
0 & \cdots & \cdots & \cdots & \cdots &\cdots & 0\\
0 & \cdots &  \mathbf{B}_{ij}^T \mathbf{\Omega}_{ij}\mathbf{A}_{ij} & \cdots & \mathbf{B}_{ij}^T \mathbf{\Omega}_{ij}\mathbf{B}_{ij} & \cdots & 0\\
 0 & \cdots & \cdots & \cdots & \cdots &\cdots & 0
\end{pmatrix}
\end{aligned}
$$

$$
\mathbf{H} = \sum_{ij} \mathbf{H}_{ij}
$$


위에서 설명한 과정을 통해서 최적의 $$\mathbf{x}$$를 계산하는 과정을 정리하면 다음과 같다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/algorithm.png" width="100%">

위에서 설명한 과정을 통해 모든 measurement에 대한 information matrix($$\mathbf{H}$$)와 $$\mathbf{b}$$를 계산하고, 두 값을 이용하여 state의 변화량인 $$\triangle \mathbf{x}$$를 계산한다($$\triangle \mathbf{x} = -\mathbf{H}^{-1}\mathbf{b}$$). $$\mathbf{H}$$는 크기가 큰 matrix이므로 inverse과정이 높은 계산량을 요구하므로, 이를 쉽게 계산하기 위해서 Cholesky factorization 방법을 사용한다([이전 글](http://jinyongjeong.github.io/2017/02/26/lec12_Least_squarees/) 참고). 이제 계산된 변화량을 이용하여 state를 업데이트하고($$\mathbf{x} = \mathbf{x}+\triangle \mathbf{x}$$) state가 수렴할 때 까지 반복하여 최적화된 state를 계산한다.

## Example of Pose graph

이해를 돕기위해 graph의 optimization과정을 통한 state update과정의 예를 통해 설명할 것이다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/example.png" width="100%">

위 그림과 같이 현재 2개의 state가 있으며, 센서를 통해 측정한 두 node 사이의 odometry정보는 1이다. 처음에는 모든 state의 값을 모르기 때문에 모두 0으로 설정한다.

$$
\begin{aligned}
\mathbf{x} &= \begin{pmatrix} 0 & 0 \end{pmatrix}^T\\
\mathbf{z}_{12} &= 1\\
\mathbf{\Omega} &= 2\\
\mathbf{e}_{12} &= \mathbf{z}_{12} -(x_2 - x_1) = 1-(0-0) = 1\\
\mathbf{J}_{12} &= \begin{pmatrix}1 -1\end{pmatrix}^T\\
\mathbf{b}_{12} &= \mathbf{e}_{12}^T \mathbf{\Omega}_{12} \mathbf{J}_{12} = \begin{pmatrix} 2 & -2 \end{pmatrix}\\
\mathbf{H}_{12} &= \mathbf{J}_{12}^T \mathbf{\Omega} \mathbf{J}_{12} = \begin{pmatrix} 2 & -2 \\ -2 & 2\end{pmatrix}\\
\triangle \mathbf{x} &= -\mathbf{H}_{12}^{-1} b_{12}
\end{aligned}
$$

센서 measurement의 information은 2로 설정하였다. state update를 위해 $$\mathbf{b}, \mathbf{H}$$를 계산하였다. 이때 $$\triangle\mathbf{x}$$를 계산하기 위해서는 $$\mathbf{H}$$의 inverse를 계산해야 하지만 현재 $$det(\mathbf{H})$$는 0이므로 계산할 수 없다. 이것은 모든 node의 uncertainty가 같아 graph가 floating상태이기 때문이다. 이를 해결하기 위해서 첫번째 node를 고정(fix)시켜 줘야한다. 이를 위해 첫번째 node의 uncertainty를 낮춰준다(즉, information을 더해서 크게 해준다).

$$
\begin{aligned}
\mathbf{H} &= \begin{pmatrix} 2 & -2 \\ -2 & 2 \end{pmatrix} + \begin{pmatrix} 1 & 0 \\ 0 & 0 \end{pmatrix}\\
\triangle \mathbf{x} &= -\mathbf{H}^{-1} b_{12}\\
\triangle \mathbf{x} &= \begin{pmatrix} 0 & 1 \end{pmatrix}^T
\end{aligned}
$$

따라서 error를 최소화 하는 state의 변화량은 $$\begin{pmatrix} 0 & 1 \end{pmatrix}^T$$가 되며 state를 업데이트하면 최종적으로 $$\mathbf{x} = \mathbf{x} + \triangle \mathbf{x} = \begin{pmatrix} 0 & 1 \end{pmatrix}^T$$가 된다.

## Hierarchical Approach to Least Square SLAM

Hierarchical Approach는 Graph SLAM의 속도를 향상시키기 위한 한가지 방법이다.

<img align="middle" src="/images/post/SLAM/lec13_least_square_SLAM/hierarchical.png" width="100%">

맨 왼쪽 그림은 원래의 pose graph를 보여주고 있다. Hierarchical Approach는 이름에서 알 수 있듯이 여러개의 계층구조로 구성되어 있으며 아래의 layer일수록 많은 node를, 위로 갈수록 적은 수의 node를 갖는 graph로 구성되어 있다. Optimization 문제는 node의 갯수가 많을 수록 계산시간이 오래 걸리기 때문에 이러한 계층구조를 이용하여, 적은 수의 node의 graph에서(sampling된 node로 이루어진 graph) 최적의 node위치를 계산 후, 계산된 node를 기준으로 주변 node의 위치를 다시 계산하는 방법을 이용한다. Hierarchical Approach에 대해서는 자세히 언급하지 않겠다. 이 부분에 대해서 자세히 알고 싶으면 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의 16강을 참고하기 바란다.


**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
