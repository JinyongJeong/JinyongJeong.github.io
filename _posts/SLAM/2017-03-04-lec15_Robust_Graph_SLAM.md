---
layout: post
title: '[SLAM] Robust Graph SLAM'
tags: [SLAM]
description: >
  Outlier에 영향을 덜 받는 Robust Graph SLAM에 대해서 알아본다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

이번 글에서는 이전에 설명했던 [graph-based SLAM](http://jinyongjeong.github.io/2017/02/26/lec13_Least_square_SLAM/)이 outlier(잘못된 정보)에 Robust하게 만드는 방법에 대해서 설명한다.

Graph-based SLAM은 least square방법을 사용하여 로봇과 landmark의 위치를 최적화 시킨다. 즉, 현재의 graph상의 로봇의 위치에서 얻어질 것으로 예상되는 측정값과 실제 측정값과의 차이를 최소화 시키는 방향으로 graph가 최적화 된다.

만약 그래프가 최적화 되는 과정에서 아래 그림과 같이 잘못된 연결(edge)이 생성되면 어떻게 될까?

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/outlier.png" width="100%">

위 그림과 같이 전혀 다른 두 위치를 같은 위치로 인식하여 edge를 발생시켰을 경우 아래와 같이 그래프 전체에 왜곡이 발생하게 된다.

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/distortion.png" width="100%">

그리고 이렇게 잘못 발생된 edge가 많아질 수록 왜곡은 심해지게 된다. 이러한 문제는 다음과 같은 상황에서 주로 발생한다.

1. 특징이 없는 환경(아무런 특징이 없는 복도 등)
2. 같은 빌딩내의 비슷한 환경의 방
3. GPS의 multi-path(GPS 신호가 다른 건물에 반사된 후 수신되어 오차가 발생하는 현상)

따라서 실제 graph-based SLAM을 수행할 때 위의 상황 뿐만아니라 다양한 상황에서 왜곡이 발생하게 된다. 이러한 잘못된 정보를 outlier라고 하며, 이러한 outlier에 덜 영향을 받는 최적화 방법이 필요하다. 이번 글에서는 최적화 과정을 Robust하게 만드는 방법에 대해서 설명하도록 한다.

### Robust M-estimator

이전 글인 [Least square](http://jinyongjeong.github.io/2017/02/26/lec12_Least_square/)에서는 noise를 Gaussina 분포로 가정하고, least square방법으로 최적해를 구하는 방법에 대해서 설명하였다. 또한 least square와 Gaussian 분포의 관계에 대해서도 언급하였었다. Least square방법은 M-estimator의 한 종류이며, Gaussian noise를 가정한 model이다. M-estimator는 noise의 형태를 Gaussian으로 가정하지 않는다. M-estimator의 PDF(Probability density function)은 아래와 같이 정의된다.

$$
p(e)= exp(-\rho(e))
$$

그렇다면 least square에서 최적해를 계산하는 것과 같이 negative log likelihood form으로 표기하면 PDF의 최대값을 찾는 문제는 log likelihood의 최소값을 찾는 문제가 된다. 이 부분에 대해서 이해가 잘 되지 않으면 [Least square](http://jinyongjeong.github.io/2017/02/26/lec12_Least_square/)를 참고하자.

$$
\mathbf{x}^* = argmin_{\mathbf{x}} \sum_i \rho(e_i(\mathbf{x}))
$$

즉 위의 log likelihood식에서 $$\rho(e)$$가 $$\rho(e) = e^2$$ 처럼 제곱의 형태인 것이 Least square문제가 되는 것이다. 그렇다면 $$\rho$$ fucntion에는 어떤 종류들이 있는지 살펴보도록 하자.

* Gaussian: $$\rho(e) = e^2$$
* L1 norm: $$\rho(e) = \mid e \mid$$
* Huber M-estimator: $$ \rho(e) = \begin{cases} \frac{e^2}{2} & if \mid e \mid < c \\
c(\mid e \mid -\frac{c}{2}) & otherwise \end{cases}$$
* Others: Tukey, Cauchy ...

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/estimator.png" width="100%">

위 그림은 각 함수의 형태를 보여준다. 최적화를 하기 위한 cost function의 형태에 변화를 줌으로써 outlier에 강인한 특성을 주기 위한 노력들이다. 이렇게 cost function의 형태를 바꿈으로써 outlier에 Robust한 특성을 어떻게 갖게 되는 것인지 살펴보자. 최적화는 error들의 합을 최소화 하는 방향으로 입력을 변화시키며 해를 찾아가는 방법이다. 이러한 과정에서 error가 매우 큰(Outlier라고 생각 할 수 있는) term들이 최적화 과정에서 큰 weight로 작용하게 된다(영향을 크게 준다). 따라서 위의 함수들은 ourlier라고 생각 할 수 있는, 즉 error가 매우 큰 term에 대해서 weight를 다소 줄이는 방향으로 cost function의 형태를 구성한 것이다. 다양한 형태의 function중에서 가장 많이 사용되는 형태는 Huber M-estimator이다.

앞으로 설명할 Max-mixture와 dynamic covariance scaling 방법도 m-estimator와 비슷한 개념을 이용하여 최적화의 robust함을 높이려는 방법이다.

### Max-mixture

Max-mixture는 M-estimator와 마찬가지로 graph-based optimization을 outlier에 robust하게 만들기 위한 방법중에 하나이다. Max-mixture는 이름에서 알 수 있듯이 여러개의 Gaussian 분포를 합하는 과정에서 수학적인 이점을 얻기 위해서 각 Gaussian 분포의 최대값을 이용한다. 왜 이러한 방법을 사용하는지 아래 수식을 보도록 하자.

세상에 존재하는 다앙햔 분포를 1개의 Gaussian만을 이용하여 표현하기엔 부족한 경우가 많이 발생한다. 따라서 다양한 분포를 표현하기 위해서 여러개의 Gaussian을 더하는 형태로 분포를 표현할 수 있다.

$$
p(\mathbf{z}\mid\mathbf{x}) = \sum_k w_k \eta_k exp(-\frac{1}{2}\mathbf{e}_{ij}^T \Omega_{ij}\mathbf{e}_{ij})_k
$$

위의 분포를 최적화 시키기 위한 cost function을 얻기 위해 negative log likelihood을 취하면 다음과 같은 형태가 된다.

$$
-log p(\mathbf{z}\mid\mathbf{x}) = -log \sum_k w_k \eta_k exp(-\frac{1}{2}\mathbf{e}_{ij}^T \Omega_{ij}\mathbf{e}_{ij})_k
$$

1개의 Gaussian분포일 경우에는 sum($$\sum$$)이 없기 떄문에 log와 exponential이 계산되어 우리가 앞에서 계산한 cost function의 형태로 계산되어 최적화 과정을 진행할 수 있게 된다. 하지만 Gaussian mixture 모델의 경우 Sum의 형태이기 때문에 log가 sum의 안으로 들어갈 수 없다. 따라서 mixture 모델을 최적화 시키기 위해서 최대값을 이용하는 max-mixture approximation 방법을 이용한다.

$$
\begin{aligned}
p(\mathbf{z}\mid\mathbf{x}) &= \sum_k w_k \eta_k exp(-\frac{1}{2}\mathbf{e}_{ij}^T \Omega_{ij}\mathbf{e}_{ij})_k\\
&\approx max_k \ \ w_k \eta_k exp(-\frac{1}{2}\mathbf{e}_{ij}^T \Omega_{ij}\mathbf{e}_{ij})_k
\end{aligned}
$$

즉 위의 근사화과정을 그림으로 표현하면 아래와 같다.

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/max_mixture.png" width="100%">

즉 두개의 분포의 값을 더하는 것 대신 최대값만을 추출하여 분포를 표현하는 것이다. 이는 두 분포의 평균값이 어느정도 거리로 떨어져 있을 경우에는 오차가 작지만 두 분포의 평균값이 매우 가까운 경우에는 큰 오차를 발생시킨다. 근사화된 식의 negative log likelihood는 다음과 같다.

$$
\begin{aligned}
-log p(\mathbf{z}\mid\mathbf{x}) &= -log \ \ max_k \ \ w_k \eta_k exp(-\frac{1}{2}\mathbf{e}_{ij}^T \Omega_{ij}\mathbf{e}_{ij})_k\\
&= min_k (\frac{1}{2}\mathbf{e}_{ij}^T \Omega_{ij}\mathbf{e}_{ij})_k - log(w_k \eta_k)
\end{aligned}
$$

즉 max-mixture는 Gaussian 분포의 max값을 취하는 형태로 근사화 함으로써 우리가 풀 수 있는 최적화의 Cost function을 구할 수 있게 되었다.

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/mm_cost.png" width="100%">

위 그림에서 빨간색 그래프는 같은 mean값을 갖는(다른 variance) 두 gaussian의 합을 max-mixture방법을 통해 얻은 분포의 cost-function을 보여준다. 앞에서 설명한 M-estimator의 cost function들과 유사함을 알 수 있다. 따라서 여러개의 Gaussian 분포를 더함으로써 graph optimization과정을 Robust하게 만들 수 있다.

Max-mixture에 대해서 더 공부하고 싶으면 [이 논문](/images/post/SLAM/lec15_MM_robust_graph/Eolson_2013_IJRR.pdf)을 참고하기 바란다.

### Dynamic Covariance Scaling(DCS)

Graph의 최적화를 Robust하게 만드는 다른 방법은 DCS(Dynamic Covariance Scaling)방법이다. DCS는 각 Error값에 해당하는 information matrix의 크기를 조절함으로써 robust하게 만든다. 원래 graph optimization식은 다음과 같다.

$$
\mathbf{x}^* = argmin_{\mathbf{x}} \sum_{ij} \mathbf{e}_{ij}(\mathbf{x})^T \Omega_{ij}\mathbf{e}_{ij}(\mathbf{x})
$$

DCS는 cost function의 information matrix의 크기를 조절하는 scaling factor($$s_{ij}^2$$)를 추가한다.

$$
\mathbf{x}^* = argmin_{\mathbf{x}} \sum_{ij} \mathbf{e}_{ij}(\mathbf{x})^T (s_{ij}^2 \Omega_{ij})\mathbf{e}_{ij}(\mathbf{x})
$$

scaling factor인 $$s_{ij}^2$$는 다음과 같이 정의된다.

$$
s_{ij} = min(1, \frac{2\Phi}{\Phi+\chi_{ij}^2})
$$

즉 두개의 파라미터 $$\Phi$$와 $$\chi$$에 의해 scaling factor가 조절되며, 이 두 파라미터에 의해서 cost function이 변하게 된다.

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/DCS.png" width="100%">

위 그림은 DCS의 원래 cost function(검정색)과 scaling factor(파란색)그리고 최종 scaling이 된 cost function(빨간색)을 보여준다. Error가 적은 영역에서 scaling factor는 1이지만 error가 큰 영역으로 갈 수록 scaling factor가 작아지며, error가 큰 영역의 함수 출력값을 줄여준다. 이는 m-estimator가 error가 큰 영역의 값을 줄이는 것과 비슷한 효과로 볼 수 있다. 원래 DCS가 이러한 개념을 처음 사용한 것은 아니다. DCS가 나오기 전인 2012년 IROS에 error가 큰 term의 영향을 없애는 "Switchable Constraint for Robust Pose Graph SLAM"이 발표되었다. 이는 획기적으로 graph 최적화를 robust하게 만들었으나 과정이 매우 복잡한 단점이 있었다. 그 이후에 "Switchable Constraint" 논문을 위와같이 close form으로 정리하여 ICRA 2013년도에 발표한 논문이 "Robust Map Optimization using Dynamic Covariance Scaling"이다.

이 글에서 DCS에 대해서 너무 깊이 논하지는 않을 것이다. 조금 더 자세히 내용이 필요한 경우 위 논문을 찾아보기 바란다.

### 정리

이번 글에서는 graph-based SLAM을 outlier에 robust하게 만드는 방법에 대해서 살펴보았다. 이러한 목적으로 여러가지 방법이 사용되고 있으나 결과적으로는 최적화의 cost function에서 error가 큰 영역의 출력값을 줄여주는 방법이라는 점에서는 유사함을 알 수 있다. 

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
