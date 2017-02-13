---
layout: post
title: '그래픽 카드별 CUDA 및 Opencv 설정'
tags: [ubuntu]
description: >
  그래픽 카드별 CUDA 및 Opencv 설정
---

현재 github의 fast rcnn 을 사용하면서 부딛히는 문제점들을 정리해본다.

현재 사용하고 있는 PC는 2대로 한대는 GTX760, 다른 한대는 GTX1060을 사용하고 있는데

그래픽카드가 다르기 때문에 setting또한 달라서 복잡하다. 

# 1. GTX760 PC

GTX760은 CUDA7.5와 Oepncv 2.4조합으로 잘 세팅이 된다. 

# 2. GTX1060 PC

## CUDA 8.0

다른 gtx1080을 사용하는 친구는 CUDA7.5로 fast rcnn 및 caffe세팅이 된다는데 나는 계속 오류가 나서 CUDA8.0 RC버전으로 올려서 설치하니까 caffe 오류는 잡을 수 있었다. 하지만 CUDA8.0으로 올리면서 Opencv와의 호환에 문제가 발생한다.

## OPENCV3.0 혹은 3.1

CUDA8.0을 설치하고 Opencv3.0을 build하면 오류가 나는데, opencv에서 graphcut알고리즘이 cudalegacy 모듈을 사용하는데 CUDA8.0에서 이 legacy를 삭제한것 같다. 현재 이 문제는 github의 opencv에는 해결되어 있으므로 github의 opencv를 사용하면 해결된다. 

문제해결 포스팅 관련 URL

<https://github.com/opencv/opencv/pull/6510>
 
문제 해결된 opencv github

<https://github.com/opencv/opencv>


## 추가적인 문제

### 1. thrust

추가적으로 CUDA8.0RC에 thrust관련 문제가 있다. 

문제해결 포스팅 관련 URL

<https://github.com/thrust/thrust/issues/800>

thrust github URL

<https://github.com/thrust/thrust>

위의 포스팅에 글처럼 github에서 git clone을 받은 후 기존의 `/usr/local/cuda8.0/thrust` 를 `thrust_old`로 바꾸고 다운받은 `thrust`폴더의 링크를 넣어준다. 

### 2. Opencv3.0 features2D error

Opencv3.0 을 build하다보면 features2d.hpp에서 error가 발생한다. 

위의 문제와 관련된 post는 다음에 있다

<http://stackoverflow.com/questions/33050599/opencv-3-0-features2d-hpp-error-unknown-algorithminfo>

vector관련 에러는 `using std::vector`를 넣어주고

`AlgorithmInfo` 관련 에러는 `AlgorithmInfo`를 `Algorithm`으로 바꾸어 준다. 


기타 fast rcnn 설정을 위한 참고 URL

<http://m.blog.daum.net/goodgodgd/27>

