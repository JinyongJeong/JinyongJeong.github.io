---
layout: post
title: 'SE(3) and SO(3) transformation'
tags: [math]
description: >
  SE(3) and SO(3) transformation
---
## SE(3), SO(3) and GL(3,R)

SE(3), SO(3), GL(3,R) 와 관련된 내용을 정리해 보려고 한다.

만약 3차원 공간상에서 f를 x1의 점을 x2의 점으로 변환시키는 matrix R이라고 할 때 다음과 같이 표현 할 수 있다.

$$ f : \mathbb{R}^3 \to \mathbb{R}^3 $$

$$
\begin{bmatrix}
x_2\\
y_2\\
z_2  
\end{bmatrix}
= R \begin{bmatrix}
                  x_1\\
                  y_1\\
                  z_1  
                  \end{bmatrix}
$$

이때 역행렬이 가능한 $$3X3$$ 매트릭스의 set은 general linear group $$GL(3, \mathbb{R})$$ 이다. 이런 무한개의 가능성을 갖고 있는 R 중에서 determinant 가 $$\pm1$$인 orthogonal matrices 들을 orthogonal group 이라고 한다.($$O(3)$$ $$\subset$$ $$GL(3, \mathbb{R})$$)

이러한 변환 matrix중에서 두 점의 쌍의 거리가 변하지 않는 transformation을 isometries이라고 하며, 그 중에서 determinant가 +1인 matrix을 proper isometries라고 한다. 이러한 special orthogonal group을 $$SO(3)$$ 라고 한다. ($$SO(3)\subset O(3)$$)

이런 $$SO(3)$$ group은 순수한 rotation만 표현 가능하다. Translation을 표현하기 위해서는 $$4X4$$ matrix를 고려해야 하며, 3D point들은 homogeneous coordinate으로 확장해야 한다.($$GL(4, \mathbb{R})$$)

$$
\begin{bmatrix}
X_2\\
1
\end{bmatrix}
= T \begin{bmatrix}
                  X_1\\
                  1
                  \end{bmatrix}
$$

$$
\begin{bmatrix}
x_2\\
y_2\\
z_2\\
1
\end{bmatrix}
=\begin{bmatrix}
                  R_{11}&R_{12}&R_{13}&T_x\\
                  R_{21}&R_{22}&R_{23}&T_y\\
                  R_{31}&R_{32}&R_{33}&T_z\\
                  0&0&0&1
                  \end{bmatrix}
                  \begin{bmatrix}
                  x_1\\
                  y_1\\
                  z_1 \\
                  1
                  \end{bmatrix}
$$

이러한 T중에서 R의 determinant가 +1을 만족하며, affine rigid motion을 이루는 group을 special Euclidean group($$SE(3)$$)라고 부른다.(affine: 평행선의 관계는 유지되는 변환, rigid(isometry): 각 쌍의 점들의 거리가 변하지 않는 변환)

즉 $$SE(3) \subset GL(4, \mathbb{R})$$의 관계이다.

더 자세한 내용은 [A tutorial on SE(3) transformation parameterizations and on-manifold optimization](/Download/SE3/jlblanco2010geometry3d_techrep.pdf) 를 참고하기 바란다.
