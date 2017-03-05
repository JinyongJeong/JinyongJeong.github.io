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

이번 글에서는 이전에 설명했던 [graph-based SLAM](http://jinyongjeong.github.io/2017/02/26/lec13_Least_square_SLAM/)이 Outlier(잘못된 정보)에 Robust하게 만드는 방법에 대해서 설명한다.

Graph-based SLAM은 least square방법을 사용하여 로봇과 landmark의 위치를 최적화 시킨다. 즉, 현재의 graph상의 로봇의 위치에서 얻어질 것으로 예상되는 측정값과 실제 측정값과의 차이를 최소화 시키는 방향으로 graph가 최적화 된다.

만약 그래프가 최적화 되는 과정에서 아래 그림과 같이 잘못된 연결(edge)이 생성되면 어떻게 될까?

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/outlier.png" width="100%">

위 그림과 같이 전혀 다른 두 위치를 같은 위치로 인식하여 edge를 발생시켰을 경우 아래와 같이 전체 graph가 왜곡되는 현상이 발생한다.

<img align="middle" src="/images/post/SLAM/lec15_MM_robust_graph/distortion.png" width="100%">

그리고 이렇게 잘못 발생된 edge가 많아질 수록 왜곡은 심해지게 된다. 이러한 문제는 다음과 같은 상황에서 주로 발생한다.

1. 특별한 특징이 없는 환경(아무런 특징이 없는 복도 등)
2. 같은 빌딩내의 비슷한 방의 환경
3. GPS의 multi-path(GPS 신호가 다른 건물에 반사된 후 수신되어 오차가 발생하는 현상)

따라서 실제 graph-based SLAM을 수행할 때 위와 같은 상황 뿐만아니라 다양한 상황에 따라서 왜곡이 발생하게 된다.
따라서 이러한 잘못된 정보를 outlier라고 하며, 이러한 outlier에 robust한 최적화 방법이 필요하다.



**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
