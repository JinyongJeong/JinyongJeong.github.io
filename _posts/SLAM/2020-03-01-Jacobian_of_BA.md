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

## Jacobian with respect to Lie algebra

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

최종 rotation의 Jacobian을 보면 상당히 복잡하다. 
이 식을 조금 단순화 시키기 위해서 roation matrix를 단순화 시킨다.
Multiple view geometry 책의 Appendix 6의 A6.9.1에 이런 말이 있다.
Lie algebra t가 작으면, rotation matrix는 다음과 같이 근사 가능하다.

$$ R = I + [t]_{\times}$$

따라서 근사화된 R에 대해서 jacobian을 구해보면 다음과 같다. 

<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/Bundle Adjustment-11.jpg" width="100%">

즉 근사화된 Rotation matrix를 이용해서 Jacobian을 구해보면 입력 measurement의 skew symmetric matrix가 됨을 알 수 있다. 

대부분의 tracking 문제에서는 계산하고자 하는 state는 두 frame 사이의 relative pose이고, 이 relative는 크지 않다. 

따라서 R을 위와같이 근사화 해서 풀 수 있다. 

## Jacobian with respect to Quaternion

그렇다면 state가 quaternion으로 rotation을 표현한다면 어떻게 될까?
visual SLAM 코드를 공부하기 좋은 Pro-SLAM의 코드는 quaternion으로 rotation state를 표현하고 있으며, w를 제외한 3개로 state를 표현한다.
w는 3개의 state로 recovery가 가능하다. 
Point를 Quaternion으로 회전 시키는 식을 미분하면 다음과 같다.

<img align="middle" src="/images/post/SLAM/Jacobian_of_BA/quaternion.png" width="100%">

state를 3개만으로 표현하였기 때문에 최종 matrix에서 뒤의 3 column 즉

$$2 (v^TaI+va^T-av^T-w[a]_{\times})$$

만 해당이 된다. 

위에서 설명한 것 처럼 대부분의 tracking 문제에서는 두 frame 사이의 relative를 계산하는 것이며, 이 relative는 크지 않다.

따라서 Jacobian은 회전이 0 (identity) 에서 미분한 값으로 단순화 할 수 있다. (찾고자 하는 relative rotation이 작기 때문에)

회전이 0, identity일 때 quaternion의 w 는 1, 그리고 i,j,k term (위 식에서는 v vector)은 0이 된다. 

따라서 quaternion의 jacobian은 다음이 된다.

$$-2[a]_{\times}$$

Pro-SLAM 코드에서 rotation state는 quaternion으로 표현되기 때문에 위의 Jacobian을 사용한다. 

Rotation의 Jacobian을 계산하는 코드를 보면 일치함을 알 수 있다. 


    //ds update total error
    _total_error += _errors[u];
    
    //ds compute the jacobian of the transformation
    Matrix3_6 jacobian_transform;
    
    //ds translation contribution (will be scaled with omega)
    // proj(M_i)를 p_cam 으로 미분한 부분에서 translation부분
    jacobian_transform.block<3,3>(0,0) = _weights_translation[u]*Matrix3::Identity();
    
    //ds rotation contribution - compensate for inverse depth (far points should have an equally strong contribution as close ones)
    // rotation 부분에 대한 jacobian 계산, 여기서 사용한 state는 quaternion form이다.
    jacobian_transform.block<3,3>(0,3) = -2*srrg_core::skew(sampled_point_in_camera_left);
    
    //ds precompute
    // Intrinsic matrix를 곱해주는 부분
    const Matrix3_6 camera_matrix_per_jacobian_transform(_camera_calibration_matrix*jacobian_transform);
    
    //ds precompute
    const real inverse_sampled_c_left  = 1/sampled_c_left;
    const real inverse_sampled_c_right = 1/sampled_c_right;
    const real inverse_sampled_c_squared_left  = inverse_sampled_c_left*inverse_sampled_c_left;
    const real inverse_sampled_c_squared_right = inverse_sampled_c_right*inverse_sampled_c_right;
    
    //ds jacobian parts of the homogeneous division: left
    Matrix2_3 jacobian_left;
    // 각 카메라의 coordinate로 이동한 measurement로 만든 matrix (미분 term에서 가장 앞에 곱해지는 matrix)
    jacobian_left << inverse_sampled_c_left, 0, -sampled_abc_in_camera_left.x()*inverse_sampled_c_squared_left,
                   0, inverse_sampled_c_left, -sampled_abc_in_camera_left.y()*inverse_sampled_c_squared_left;
    
    //ds we compute only the contribution for the horizontal error: right
    Matrix2_3 jacobian_right;
    jacobian_right << inverse_sampled_c_right, 0, -sampled_abc_in_camera_right.x()*inverse_sampled_c_squared_right,
                    0, inverse_sampled_c_right, -sampled_abc_in_camera_right.y()*inverse_sampled_c_squared_right;
    
    //ds assemble final jacobian
    _jacobian.setZero();
    
    //ds we have to compute the full block
    // 최종 p_cam으로 미분 한 matrix, 여기에선 left, right의 reporjection error를 concat해서 사용
    _jacobian.block<2,6>(0,0) = jacobian_left*camera_matrix_per_jacobian_transform;
    
    //ds we only have to compute the horizontal block
    _jacobian.block<2,6>(2,0) = jacobian_right*camera_matrix_per_jacobian_transform;
    
    //ds precompute transposed
    const Matrix6_4 jacobian_transposed(_jacobian.transpose());
    
    //ds update H and b
    // 최적화 문제를 풀기 위해 계산된 jacobian으로 hessian 계산 (이 부분은 이전 Graph SLAM post 참고)
    _H += jacobian_transposed*_omega*_jacobian;
    _b += jacobian_transposed*_omega*error;
