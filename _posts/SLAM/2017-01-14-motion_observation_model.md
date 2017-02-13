---
layout: post
title: '[SLAM] Motion & Observation model'
tags: [SLAM]
description: >
  Motion model과 observation model 설명.

  본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 부분을 지적해주시면 확인 후 수정토록 하겠습니다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

이번 글에서는 SLAM의 framework에서 중요한 Motion model과 Observation model에 대해서 설명한다. 이전 post([bayes filter](http://jinyongjeong.github.io/2017/01/13/SLAM_bayes_filter/))에서 설명한 것 처럼 SLAM은 다음과 같이 recursive bayes filter의 식으로 표현할 수 있다.

$$
\begin{aligned}
bel(x_t)  &= p(x_t \mid z_{1:t},u_{1:t}) \\
          &= \eta p(z_t \mid x_t)\int_{x_{t-1}}p(x_t \mid x_{t-1}, u_{t}) bel(x_{t-1}) dx_{t-1}\\
\end{aligned}
$$

**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**
