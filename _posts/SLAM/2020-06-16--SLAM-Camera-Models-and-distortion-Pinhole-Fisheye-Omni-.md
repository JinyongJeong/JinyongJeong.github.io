---
layout: post
title: '[SLAM] Camera Models and distortion (Pinhole, Fisheye, Omni)'
tags: [SLAM]
comments: true
description: >
  [CCamera Models and Distortion
sitemap :
  changefreq: weekly
  priority : 1.0
---
Visual SLAM을 공부하는 단계에서 작성한 글입니다. 혹시 틀린점이나 설명이 부족한 부분이 있으면 알려주세요. 최대한 빠르게 확인 후 업데이트 하도록 하겠습니다.  

Visual SLAM, SfM 등 카메라를 이용한 연구를 하기 위해서 가장 기본적으로 알아야 할 부분이 바로 카메라 모델이다. 일반적인 카메라는 대부분 핀홀(Pin-hole) 카메라 형태로 모델링하여 사용하고 있으며, 처음 카메라에 대해서 배울 때 대부분의 설명 자료들이 핀홀 모델을 기반으로 설명을 하고 있다. 이러한 핀홀 모델은 렌즈를 통해 들어오는 빛이 굴절되지 않고 바로 이미지 센서로 들어오는 perspective projection을 기반으로 하고 있다. 하지만 실제 연구를 진행하다 보면 넓은 화각이 이점이 되는 어플리케이션 (ex. Visual SLAM)이 있기 때문에 FOV (Field of View)가 170도 이상인 어안렌즈 (Fisheye lens)를 사용하는 경우도 많다. 이러한 화각이 매우 넓은 렌즈의 경우에는 일반적인 모델로 표현하기 어렵다. 따라서 이러한 경우에는 perspective projection이 아닌 equidistance projection 모델을 사용한다.

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled.png)

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_1.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_1.png)

카메라를 통해 이미지를 얻는다는 것은 3차원 공간에 있는 물체에 반사되어 오는 빛이 이미지 센서에 들어오는 신호가 변형되는 것이다. 따라서 카메라를 모델링 한다는 것은 실제 3차원에 있는 점을 카메라를 통해 보았을 때 실제 이미지상에 어떤 위치에 놓이게 되는지 추정하는 것이다. 이러한 카메라 모델이 정확히 정의 되어야 3차원 공간의 정보와 2D 이미지 상의 매칭을 수학적으로 표현 가능하다. VIsual SLAM, Visual odometry, SfM (Structure from Motion)등이 이러한 카메라 모델이 필요한 대표적인 분야라고 할 수 있다.

# Camera models

카메라 모델은 물체에 반사되는 빛이 이미지 센서까지 도달하는 방법에 대한 모델이다. 이러한 과정에서 빛의 왜곡에 의한 영향은 distortion 파라미터로 보정하게 된다. 먼저 다양한 카메라 모델을 살펴보자.  

여기서 $$f$$ 는 빛이 모이는 중심점에서 image plane 까지의 거리인 Focal length, $$\theta$$ 는 principal axis인 $$Z_c$$ 와 빛이 들어오는 각도, 그리고 $$r$$은 이미지의 중심인 principal point와 image point의 거리를 의미한다. 

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_2.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_2.png)

- 다양한 카메라 모델들
    - Perspective projection

    $$r = f tan \theta$$

    - Stereographic projection

    $$r = 2ftan(\theta/2)$$

    - Equidistance projection

    $$r=f\theta$$

    - Equisolid angle projection

    $$r = 2fsin(\theta/2)$$

    - Orthogonal projection

    $$r = f sin(\theta)$$

빛이 이미지로 변환되는 과정은 다양한 모델로 표현이 되지만, 주로 사용되는 모델은 perspective projection, Equidistance projection 모델이다. 이 글에서는 이 두 모델, 그리고 추가적으로 omni camera model까지 다뤄본다. 

## Perspective projection (Pinhole model)

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Screenshot_from_2020-06-09_21-42-24.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Screenshot_from_2020-06-09_21-42-24.png)

Perspective projection은 기본적인 pin-hole 카메라 모델이며, Principal axis (노란색 line) 기준으로 빛이 들어오는 각도와 image plane으로 나아가는 빛의 각도가 동일한 모델이다. 즉 위 그림 기준으로 $$\alpha = \beta$$ 이다. 다양한 카메라 모델을 설명할 때의 식으로 표현하면 다음과 같다. 

$$r = f tan \beta$$

여기서 $$f$$ 는 focal length 로 image plane과 빨간색 선의 거리를 의미하며, $$r$$은 principal axis (노란색 line)에서 부터 image plane상의 빛까지의 거리를 의미한다. 즉 빛이 들어오는 각도와 focal length에 의해 이미지에서의 위치가 결정된다. 이러한 perspective model은 가장 기본적인 카메라 모델로, 대부분의 카메라 모델에서 활용된다. 

## Equidistance projection (Fisheye projection)

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Screenshot_from_2020-06-09_21-42-29.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Screenshot_from_2020-06-09_21-42-29.png)

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_3.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_3.png)

최근에는 Visual odometry 분야에서는 빠른 움직임에서 tracking loss를 최소화 하기 위하여 화각이 넓은 Fish-eye 카메라를 많이 활용한다. 이러한 Fish-Eye camera는 렌즈 설계부터 넓은 화각을 목적으로 설계되었기 때문에 일반적인 Perspective model로 모델링하기 어렵다. 이런 Fish-Eye 카메라에서 가장 많이 활용되는 모델이 Equidistance projection 모델이다. Perspective projection 모델은 들어오는 빛과 나가는 빛의 각도가 동일하였다면, Equidistance projection 모델은 들어오는 빛의 각도와  principal axis (노란색 선)으로 부터 떨어진 거리가 선형적인 모델이다.  즉 각도가 거리와 선형적이기 때문에 이름이 Equidistance 인 것 같다. 식으로 표현하면 위의 그림처럼

$$\frac{\alpha_1}{d_1} = \frac{\alpha_2}{d_2}$$

와 같이 표현할 수 있다. 

이러한 렌즈 설계는 오른쪽 그림과 같이 넓은 화각의 데이터를 얻을 수 있지만, 상당한 왜곡 (Distortion)을 발생시킴을 알 수 있다. 

## Omni directional 카메라 모델 (Catadioptric Camera)

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_4.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_4.png)

Omni-direction 카메라는 주변, 즉 360도를 전부 바라보는 카메라를 의미한다. 위  그림은 대표적인 omni-directional camera들의 종류를 보여준다. 첫번째 그림은 앞에서 설명한 Fish-eye lens로 180 도 이상의 화각을 갖기도 한다. 두번째는 일반적인 카메라에 거울을 붙여 수평 360도를 바라보는 카메라이다. 세번째는 여러개의 카메라를 사방으로 부착하여 사용하는 카메라이다.  여기서는 두번째 모델인 Catadioptic camera model을 설명한다. 

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_5.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_5.png)

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_6.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_6.png)

위 그림은 Catadioptric camera model을 설명하기 위한 그림이다. 이 모델은 central catadioptric camera를 위한 unified model로 다양한 곡선의 거울과, perspective model을 동시에 고려가 가능한 모델이다.  

모델을 이해하기 위한 단계는 크게 4단계로 나눠진다. 

1. Scene point인 $$P$$ 를 unit sphere로 projection 한다. 

    $$P_s = \frac{P}{||P||} = (x_s, y_s, z_s)$$

2. 중심점이 $$C_{\epsilon} = (0,0,\epsilon)$$ 인 새로운 reference frame으로 $$P_s$$를 옮긴다.

    ![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_7.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_7.png)

    여기서 $$\epsilon$$은 conic의 foci인 $$d$$와 latus rectum인 $$l$$에 의해서 결정된다.

    $$P_{\epsilon} = (x_s, y_s, z_s + \epsilon)$$

3. $$C_{\epsilon}$$ 으로 부터 거리 1떨어진 normalized image plane으로 $$P_{\epsilon}$$을 projection 한다.

    $$\tilde{m}=\left(x_{m}, y_{m}, 1\right)=\left(\frac{x_{s}}{z_{s}+\epsilon}, \frac{y_{s}}{z_{s}+\epsilon}, 1\right)=g^{-1}\left(P_{s}\right)$$

4. 마지막으로 일반적인 intrinsic matrix $$K$$를 곱해서 image coordinate으로 변환한다.

    $$\tilde{p} = K \tilde{m}$$

위 모델을 다시 간단히 살펴보면, $$\epsilon$$ 값에 따라 다양한 모델을 표현할 수 있다. 즉 일반적인 perspective model은 곡선이 없는, 즉 $$\epsilon = 0$$ 인 경우이며,  parabolic 형태의 거울일 경우에는 $$\epsilon=1$$ 인 경우이다. 더욱 자세한 내용은 아래 링크를 참고바란다. 

[](http://rpg.ifi.uzh.ch/docs/omnidirectional_camera.pdf)

# Distortion models

위에서는 주로 많이 사용되는 카메라 모델에 대해서 알아보았다. 위의 카메라 모델들은 렌즈의 형태에 의해 빛이 굴절되는 모델을 표현하는 것이며, 이상적인 경우를 의미한다. 하지만 실제로는 다양한 이유로 (렌즈의 왜곡 등) 정확히 해당 모델로 표현이 되지 않는다. 일반적으로 이러한 왜곡은 두가지 distortion 모델로 정의된다. 

### Radial Distortion

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_8.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_8.png)

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_9.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_9.png)

Radial distortion은 렌즈의 중앙부와 바깥영역의 굴절률이 달라짐에 따라서 발생한다. Distortion이 없는 경우 (No distortion)에는 정면으로 바로보았을 때 모든 직선이 직선으로 보인다. 하지만 Barrel distortion 혹은 Pincushion distortion이 발생하였을 때는 실제 환경의 직선이 휘어져 보이게 된다.  Barrel, 그리고 Pincusion이라는 이름은 실제 Barrel (통)의 형태와 Cusion(쿠션)을 눌렀을때의 형태에서 따온 것이다. 

$$x_{\text {distorted }}=x\left(1+k_{1}^{*} r^{2}+k_{2}^{*} r^{4}+k_{3}^{*} r^{6}\right)$$

$$y_{\text {distorted }}=y\left(1+k_{1}^{*} r^{2}+k_{2}^{*} r^{4}+k_{3}^{*} r^{6}\right)$$

Radial distortion은 일반적으로 위와 같이 모델링된다. $$x, y$$는 normalized image coordinate에서의 undistorted된 pixel의 위치이다. 즉 distortion이 발생하지 않았을 때의 pixel 위치를 의미한다. $$r$$은 중심축에서의 각 픽셀의 거리로 $$r^2 = x^2 + y^2$$ 이다. 그리고 나머지 $$k_1, k_2, k_3$$는 radial distortion을 표현하는 parameter이다.

### Tangential Distortion

![/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_10.png](/images/post/SLAM/2020-06-16--SLAM-Camera-Models-and-distortion-Pinhole-Fisheye-Omni-/Untitled_10.png)

Tangential distortion은 실제 물리적으로 카메라 렌즈와 이미지 센서의 mis-alignment로 인해 발생하는 distortion이다. 최근에는 기술의 발달로 하드웨어 적인 mis-alignment가 매우 적기 때문에 크게 신경쓰지는 않는 추세이긴 하나 상황에 따라서 필요하기도 하다. 

$$x_{\text {distorted }}=x+\left[2^{*} p_{1}^{*} x^{*} y+p_{2}^{*}\left(r^{2}+2^{*} x^{2}\right)\right]$$

$$y_{\text {distorted }}=y+\left[p_{1}^{*}\left(r^{2}+2^{*} y^{2}\right)+2^{*} p_{2}^{*} x^{*} y\right]$$

Tangential distortion은 위와 같이 모델링된다. Radial distortion과 마찬가지로 $$x,y$$는 undistorted pixel 위치이다. Tangential distortion은 $$p_1, p_2$$ coefficient로 표현된다. 

# 정리

이 글에서는 가장 기본적으로 많이 사용되는 카메라 모델과 Distortion 모델에 대해서 설명하였다. 대부분의 카메라 calibration 을 위한 tool 들 (Matlab, Kalibr 등)은 대부분 위에서 설명한 camera model들과 distortion model들을 지원한다. Kalibr의 경우 Pinhole camera 모델의 경우 radial distortion과 tangential distortion을 모두 추정하지만, equidistance model의 경우에는 radial distortion만 추정하여 사용한다. 

### Reference

[](https://www.isprs.org/proceedings/XXXVI/5-W8/Paper/PanoWS_Berlin2005_Schwalbe.pdf)

[A generic camera model and calibration method for conventional, wide-angle, and fish-eye lenses - IEEE Journals & Magazine](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=1642666&casa_token=oJyQMwYes2EAAAAA:ND5Nt9U4Nnbc-I1zaC4X334O9nq9LTRaQjVvhhKij0i90ylqfq5MGmq1I8h6Q9MhCQ5GiPxypcQ&tag=1)

[What Is Camera Calibration? - MATLAB & Simulink - MathWorks 한국](https://kr.mathworks.com/help/vision/ug/camera-calibration.html)

[ethz-asl/image_undistort](https://github.com/ethz-asl/image_undistort)

[](http://rpg.ifi.uzh.ch/docs/omnidirectional_camera.pdf)