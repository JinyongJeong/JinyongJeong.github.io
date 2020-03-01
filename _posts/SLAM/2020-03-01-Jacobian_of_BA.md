---
layout: post
title: '[SLAM] Bundle Adjustment의 Jacobian 계산'
tags: [SLAM]
description: >
    Bundle Adjustment의 Jacobian 계산
sitemap :
  changefreq : weekly
  priority : 1.0
---

## Jacobian

일반적으로 최적화를 수행할 때 가장 쉬운 방법은 함수의 미분을 계산하고, 현재의 값에서 미분값이 작아지는 방향으로 값을 변경해 가면서 최적화를 수행하는 방법이다. 이러한 방법을 gradient descent 방법이라고 하는데, Jacobian은 이러한 함수가 multi-variable 일때의 미분을 의미한다. 
즉 Jacobian은 multi-variable 문제에서 내가 최적화 하고 싶은 parameter들에 대한 편미분을 matrix로 표현한 것이다.
Graph-based SLAM 뿐만 아니라 BA(Bundle Adjustment), Visual SLAM쪽을 공부하다 보면 계속 이러한 Jacobian을 만나게 되는데 정확히 어떻게 계산이 되는지 알고 넘어가는게 좋을 것 같아서 수식을 정리해 보았다. 

수식이 너무 많아서 손필기로 정리하였으며, Error function은 일반적인 BA에서 사용하는 reprojection error에 대한 Jacobian을 구하는 방법이다. 
(혹시 수식적으로 틀린 부분이 있거나, 더 간단하게 풀수있는 방법이 있으면 댓글로 알려주세요)

<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-2.jpg" width="100%">
<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-3.jpg" width="100%">
<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-4.jpg" width="100%">
<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-5.jpg" width="100%">
<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-6.jpg" width="100%">
<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-7.jpg" width="100%">
<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-8.jpg" width="100%">
<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-9.jpg" width="100%">


