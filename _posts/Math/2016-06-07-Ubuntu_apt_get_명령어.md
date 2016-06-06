---
layout: post
title:  "SE(3) and SO(3) transformation"
date:   2016-06-07 13:33:49 +0900
comments: true
summary: "SE(3) and SO(3) transformation"
categories: [Math]
---

## SE(3), SO(3) and GL(3,R)

SE(3), SO(3), GL(3,R) 와 관련된 내용을 정리해 보려고 한다.

만약 3차원 공간상에서 f를 x1의 점을 x2의 점으로 변환시키는 matrix R이라고 할 때 다음과 같이 표현 할 수 있다.

$$ f : \mathbb{R}^3 \to \mathbb{R}^3 $$

$$
\begin{bmatrix}
X_2\\
Y_2\\
Z_2  
\end{bmatrix}
= R \begin{bmatrix}
                  X_1\\
                  Y_1\\
                  Z_1  
                  \end{bmatrix}
$$

이때 역행렬이 가능한 $$3X3$$ 매트릭스의 set은 general linear group $$GL(3, \mathbb{R})$$ 이다.
