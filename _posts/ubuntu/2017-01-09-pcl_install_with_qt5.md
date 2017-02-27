---
layout: post
title: '원하는 QT 버전에 맞게 PCL 설치하기'
tags: [ubuntu]
description: >
  PCL을 설치할 때 원하는 QT버전으로 지정하여 설치하는 방법에 대해서 설명한다. 
---


원하는 버전의 QT5와 VTK, PCL을 설치하는 방법에 대해 설명한다. 

일반적인 방법으로 PCL을 설치할 경우 PCL의 기본 QT의 경로가 /usr/lib/x86_64-linux-gnu/ 로 잡히게 된다. 

그러나 일반적으로 QT5를 sudo로 설치할 경우 /opt 에 설치되기 때문에 경로를 설정해줘야 한다. 

VTK와 PCL를 제외한 나머지 dependency는 설치 되어 있다고 하자.(openni 등)

만약 dependecy가 설치되어 있지 않다면 이 [shell script](/images/post/ubuntu/pcl_install/essential_install.sh)를 다운받아 실행하자.


```
chmod 777 essential_install.sh
sudo ./essential_install.sh
```

위 shell script는 PCL의 dependency와 기타 기본 라이브러리를 설치해준다. 

PCL의 dependency설치에 대해서 자세히 보고 싶으면 다음 [이 블로그](https://github.com/hsean/Capstone-44-Object-Segmentation/wiki/PCL-1.8:-Ubuntu-14.04-Installation-Guide)를 참고하자.

이제 기본적인 dependency가 설치되어있는 환경에서 PCL의 dependency인 VTK와 PCL을 원하는 QT버전에 맞게 설치하는 방법에 대해서 설명한다. 이는 QT를 /opt/ 경로에 설치한 경우이다. 

# 1. VTK 설치

git clone으로 VTK를 다운받아 준다


```
git clone https://github.com/Kitware/VTK.git
```

그런 다음 build 폴더를 만들어서 build폴더로 이동한 후 다음 옵션으로 cmake해준다. 옵션으로 Qt5의 경로를 지정해준다. 


```
cd build

cmake -DVTK_QT_VERSION:STRING=5 -DQT_QMAKE_EXECUTABLE:PATH=/opt/Qt5.6.1/5.6/gcc_64/bin/qmake -DVTK_Group_Qt:BOOL=ON -DCMAKE_PREFIX_PATH:PATH=/opt/Qt5.6.1/5.6/gcc_64/lib/cmake -DBUILD_SHARED_LIBS:BOOL=ON ..
```

그다음 설치해준다.


```
make -j8
sudo make install
```

# 2. PCL 설치

git 에서 다운받는다


```
git clone https://github.com/PointCloudLibrary/pcl.git
```

PCL의 qt5 경로를 설정해주기 위해서 cmake file에서 prefix 를 설정한다. /pcl/cmake/pcl_find_qt5.cmake 를 열고 최 상단에 아래의 setting을 추가


```
set(CMAKE_PREFIX_PATH /opt/Qt5.6.1/5.6/gcc_64/lib/cmake)
```

이제 build 폴더를 만들고 빌드 후 설치한다.


```
mkdir build
cd build
cmake ..
make -j8
make install
```





