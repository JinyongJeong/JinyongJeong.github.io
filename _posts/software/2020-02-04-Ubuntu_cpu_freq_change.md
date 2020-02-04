---
layout: post
title: 'Ubuntu CPU Frequency 최적화 하기'
tags: [software]
description: >
    Ubuntu CPU Frequency 최적화 하기
---


## Ubuntu CPU Frequency

일반적으로 전력 관리를 위해 CPU의 frequency는 가변적으로 설정되어 있다. 일반적으로 800MHz-4200MHz정도로 설정되어 있다. 이런 CPU의 Frequency는 OS가 자동적으로 load을 보고 조절하는 것으로 알고 있으나, 실제 Ubuntu에서 구현된 code를 돌려보면 CPU의 frequency의 변화에 의해서 함수 구동 속도가 변하는 현상을 볼 수 있다. 일반적으로 Code가 최대 performance로 돌아야 하는 경우는 CPU frequency가 일정하게 최대 속도인것이 좋다. 

현재 frequency 및 min/max frequency는 아래 명령어로 확인 가능하다

```
watch -n 0.1 "lscpu | grep MHz"
```


## Ubuntu CPU Frequency 조절

다음 package를 설치한다. 


```
sudo apt-get install cpufrequtils
```

CPU min 설정

```
sudo cpufreq-set -d 0.8Ghz
```

CPU min frequency 설정

```
sudo cpufreq-set -u 0.8Ghz
```

아래 명령어를 통해 현재 CPU의 max frequency를 확인 후 약간 아래값으로 minimum 값을 설정해주면 일정한 속도의 code 성능을 확인할 수 있다. 

```
watch -n 0.1 "lscpu | grep MHz"
```

이 방법을 통해 minimum frequency를 높이게 되면 노트북의 경우 배터리 성능에 영향이 있을 수도 있으니, 적절히 사용하면 좋을 것 같다.


