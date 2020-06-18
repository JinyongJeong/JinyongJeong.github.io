---
layout: post
title: '[SLAM] Opencv Camera model 정리'
tags: [SLAM]
comments: true
description: >
  [SLAM} Opencv Camera model 정리
sitemap :
  changefreq: weekly
  priority : 1.0
---
Opencv에는 다양한 카메라 모델을 다루기 위한 기본 Class들이 정의되어 있다. 그 중에서 Pinhole model과 Fish-eye model이 어떻게 모델되어 있는지 식으로 살펴보려고 한다. 카메라 모델에는 이 두가지 모델 이외에도 다양한 모델이 있으며, 이러한 다양한 카메라 모델에 대해서는 다음의 page를 참고하자.

[[SLAM] Camera Models and distortion (Perspective, Fisheye, Omni)](http://jinyongjeong.github.io/2020/06/15/Camera_and_distortion_model/)

## Pinhole Camera model


<p align="center">
  <img src="/images/post/SLAM/2020-06-19--SLAM-Opencv-Camera-model-정리/Untitled.png" width="50%">
</p>

Pin-hole 카메라 모델은 일반적으로 많이 사용되는 카메라 모델이며, 물체로부터 오는 lay를 focal length 가 1인 normalized image plane으로 projection하고, image coordinate으로 변환시키는 모델이다. 이때 일반적으로 distortion은 radial distortion과 tangential distortion을 고려한다. 

먼저 distortion을 고려하지 않는 경우 다음과 같다. 

$$\begin{array}{l}
{\left[\begin{array}{l}
x \\
y \\
z
\end{array}\right]=R\left[\begin{array}{l}
X \\
Y \\
Z
\end{array}\right]+t} 
\end{array}$$

Distortion을 고려하지 않은 모델은 매우 단순하다. $$P=(X,Y,Z)$$ 는 global coordinate에서의 물체의 위치이다. 이때 global coordinate에서 카메라의 rotation과 translation 이 각각 $$R$$, $$t$$ 라면 첫번째 수식을 통해 P는 Camera coordinate에서의 좌표인 $$(x,y,z$$) 로 변환된다.

$$\begin{array}{l}

x^{\prime}=x / z \\
y^{\prime}=y / z 
\end{array}$$


<p align="center">
  <img src="/images/post/SLAM/2020-06-19--SLAM-Opencv-Camera-model-정리/Screenshot_from_2020-06-19_06-23-04.png" width="50%">
</p>

그 다음은 Camera coordinate으로 변환된 좌표를 normalized image plane으로 변환합니다. normalized image plane은 원점으로 부터 거리가 1인 평면을 의미하며, 이때의 x, y값에 f를 곱함으로써 초첨거리가 f인 평면으로 변환할 수 있다. 따라서 첫번째로 camera coordinate으로 변환된 좌표를 각각 $$z$$ 로 나눠줌으로써 normalized image plane으로 변환한다. 

$$\begin{array}{l}

u=f_{x} * x^{\prime}+c_{x} \\
v=f_{y} * y^{\prime}+c_{y}
\end{array}$$

그리고 마지막으로는 $$x', y'$$ 각각에 focal length ($$f$$)를 곱하고 principal point ($$c$$) 를 더해줌으로써 image coordinate으로 변환시킨다. focal length를 곱해주는것은 위에서 설명한 것 처럼 normalized image plane에서 초점거리가 f인 image plane으로 변환하는 과정이다. 그리고  camera coordinate의 경우 이미지의 중심이 $$(0,0)$$ 이지만 이미지의 경우 일반적으로 좌측상단이 $$(0,0)$$ 이다. 따라서 이러한 좌표의 변환을 해주는 것이 principal point를 더해주는 것이다. 이 계산은 일반적으로 3X3의 Camera Intrinsic matrix ($$K$$)를 곱해주는 과정으로 표현하기도 한다. 

이러한 과정을 통해 3차원 포인트를 카메라로 바라보았을 때 해당하는 물체가 어느 픽셀에 도달할지를 계산할 수 있게 되었다. 하지만 실제 카메라 모델은 이렇게 이상적이지 않다. 렌즈의 물리적인 이유, 센서와 렌즈의 misalignment와 같은 이유로 발생하는 distortion을 보정해 주어야 한다. 일반적인 카메라 모델의 경우 보통 Radial distortion과 Tangential distortion 모델을 활용한다. 두가지 distortion model에 대해서 익숙하지 않다면, 위 링크의 글을 살펴보길 바란다. 

일반적인 Radial distortion은 다음과 같은 식으로 표현된다. 

$$x_{\text {distorted }}=x\left(1+k_{1}^{*} r^{2}+k_{2}^{*} r^{4}+k_{3}^{*} r^{6}\right)$$

$$y_{\text {distorted }}=y\left(1+k_{1}^{*} r^{2}+k_{2}^{*} r^{4}+k_{3}^{*} r^{6}\right)$$

그리고 Tangential distortion은 다음과 같이 표현된다. 

$$x_{\text {distorted }}=x+\left[2^{*} p_{1}^{*} x^{*} y+p_{2}^{*}\left(r^{2}+2^{*} x^{2}\right)\right]$$

$$y_{\text {distorted }}=y+\left[p_{1}^{*}\left(r^{2}+2^{*} y^{2}\right)+2^{*} p_{2}^{*} x^{*} y\right]$$

각 Distortion 식에서 $$x,y$$는 이상적인 (undistorted) pixel이며, $$k_1, k_2, k_3$$는 radial distortion parameter, 그리고 $$p_1, p_2$$는 tangential distortion parameter이다. 이러한 Distortion model을 적용한 모델은 다음과 같다.

$$\begin{array}{l}{\left[\begin{array}{l}x \\y \\z\end{array}\right]=R\left[\begin{array}{l}X \\Y \\z\end{array}\right]+t} \\ x'=x / z \\y^{\prime}=y / z \end{array}$$

Normalized image plane으로 변환하는 과정은 동일하다. 

$$\begin{array}{l}x^{\prime \prime}=x^{\prime} \frac{1+k_{1} r^{2}+k_{2 }r^4+k_{3} r^{6}}{1+k_{4} r^{2}+k_{5} r^{4}+k_{6} r^{6}}+2 p_{1} x^{\prime} y^{\prime}+p_{2}\left(r^{2}+2 x^{\prime 2}\right) \\y^{\prime \prime}=y^{\prime} \frac{1+k_{1} r^{2}+k_{2} r^4+ k_{3} r^{6}}{1+k_{4} r^{2}+k_{5} r^{4}+k_{6} r^{6}}+p_{1}\left(r^{2}+2 y^{\prime 2}\right)+2 p_{2} x^{\prime} y^{\prime} \\\text { where } r^{2}=x^{\prime 2}+y^{\prime 2} \end{array}$$

Normalized image plane으로 변환된 값 $$x', y'$$ 은 radial, tangential distortion 모델에 의해서 distortion이 발생한다. 위 식에서 우항의 첫번째 항은 radial distortion, 그리고 두번째 세번째 항은 tangential distortion이다. 여기서 $$r$$은 normalized image plane에서 각 점의 principal axis로 부터의 거리를 의미한다. 

$$\begin{array}{l}u=f_{x} * x^{\prime \prime}+c_{x} \\v=f_{y} * y^{\prime \prime}+c_{y}\end{array}$$

Distortion model을 적용한 이후의 normalized image plane 의 pixel ($$x'', y''$$)은 Camera intrinsic matrix를 곱해줌으로써 image plane으로 변환된다. 위와 같이 distortion이 포함된 카메라 모델을 통해 3차원 좌표상에 있는 물체가 실제 핀홀 카메라 모델에서 어느 위치의 픽셀에 나타날지 계산할 수 있게 되었다. 반대로 이미지 plane에 있는 픽셀이 실제 3차원 좌표에서 어디에 위치하는지 계산할 수 있다. 하지만 이런 경우에 위 모델을 통해 알 수 것은 normalized image plane에서의 좌표인 $$x',y'$$ 이기 때문에 실제 3차원 좌표로 복원되기 위해서는 카메라로 부터 떨어진 거리인 depth ($$z$$)을 알고 있어야 한다. 

## Fish-eye Camera model

<p align="center">
  <img src="/images/post/SLAM/2020-06-19--SLAM-Opencv-Camera-model-정리/Untitled_1.png" width="50%">
</p>


Fish-eye camera는 물고기의 눈처럼 렌즈가 볼록하여 넓은 화각을 갖는 카메라 모델을 의미한다. 일반적인 핀홀 카메라와는 다르게 Object point의 빛이 렌즈를 통과하여 직진으로 image plane에 도달하는 것이 아닌 렌즈의 왜곡에 의해 빛이 꺽여서 image plane에 도달하게 된다. Fish-eye camera를 모델링하기 위해 가장 많이 활용되는 모델은 Equidistant model로 빛의 입시각 $$\theta$$와 image plane에서 principal point와의 거리인 $$r$$ 가 비례적인 관계를 갖는 모델이다. 즉, 

$$\frac{r_1}{\theta_1} = \frac{r_2}{\theta_2}$$

이다. OpenCV의 fish-eye 카메라 모델도 이와 같은 equidistant model을 이용하고 있다. 보통 equidistant model의 경우에는 radial distortion 만을 고려한다. 지금부터 3차원 point인 $$X$$가 실제 fish-eye 모델을 통해 image plane으로 변환되는 과정을 살펴보자.

$$X_c = RX + T$$

$$x = X_{c1}, y = X_{c2}, z= X_{c3}$$

핀홀카메라와 동일하게 3차원 공간의 점 $$X$$는 카메라의 rotation, translation인 $$R, T$$를 이용하여 camera coordinate인 $$X_c$$로 변환된다. 

$$a = \frac{x}{z}, b=\frac{y}{z}$$

Camera coordinate으로 변환된 점은 depth인 $$z$$로 normalize를 함으로써 normalized image plane으로 변환된다. 

$$r^2 = a^2 + b^2$$

$$\theta=atan(r)$$

이때 중요한것은 여기서의 $$r$$은 일반적은 핀홀 카메라의 normalized image plane에서 중심점으로 부터 해당 픽셀까지의 euclidean distance를 의미한다. $$\theta$$는 빛의 입사각을 의미한다. 

$$\theta_d = \theta(1+k_1 \theta^2 + k_2 \theta^4 + k_3 \theta^6 + k_4 \theta^8)$$

위 식은 radial distortion을 적용하는 식을 나타낸다. 일반적인 핀홀 카메라와 가장 큰 차이점의 있는 부분 중에 하나이다. 핀홀 카메라의 경우에는 pixel의 x, y좌표인 $$x', y'$$에 distortion 모델이 적용되었으나, equidistant model에서는 입사각인 $$\theta$$에 distortion 모델이 적용된다. 이때 일반적으로 4개의 radial distortion parameter가 사용된다. 

$$x' =\frac{\theta_d}{r} * a$$

$$y' = \frac{\theta_d}{r}*b$$

위 식은 equidistant model의 특징인 $$\frac{r_1}{\theta_1} = \frac{r_2}{\theta_2}$$의 특징을 이용하여 pixel의 값을 scaling 하는 식이다. normalized image plane에서의 pixel 값 ($$a, b$$)는 $$r$$과 $$\theta_d$$의 비율로 scaling된다. 위에서는 distortion model이 적용되었기 때문에 $$\theta_d$$가 사용되었다. 만약 distortion model을 적용하지 않고 이상적인 모델이라면 $$\theta_d$$ 대신 $$\theta$$를 사용하면 된다. 

$$u= f_x(x' + ay') + c_x$$

$$v=f_yy'+c_y$$

마지막으로 normalized image plane에서 scaling된 pixel값에 Intrinsic matrix를 곱해 줌으로써 image coordinate으로 변환하게 된다. 이러한 계산을 통해 실제 3차원에 있는 물체가 fish-eye 카메라 모델에서 어느 pixel에 위치할지 계산할 수 있다. 보통 이러한 Fish-eye 카메라의 이미지는 많은 경우 undistortion과정 (실제 직선은 직선으로 보이도록 변환) 을 통해 이미지를 변환하여 사용하게 된다. Undistortion 과정은 위의 과정과 반대로 실제 fish-eye model로 변환된 픽셀값 $$u, v$$ 로부터 normalized image plane의 점인 $$a, b$$를 계산하는 과정으로 생각할 수 있다. 이때 depth 값은 모르기 때문에 $$z$$를 복원할 수는 없다. 

### Reference

[Camera Calibration and 3D Reconstruction - OpenCV 2.4.13.7 documentation](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html)

[OpenCV: Fisheye camera model](https://docs.opencv.org/3.4/db/d58/group_calib3d_fisheye.html)
