---
layout: post
title: '[Ubuntu] Ubuntu 노트북 사용시 배터리 최적화 하기'
tags: [ubuntu]
description: >
  Ubuntu 노트북 사용시 배터리 최적화 하기
---


## Ubuntu 노트북 배터리 최적화 하기

노트북에 윈도우와 ubuntu를 같이 사용하다보면 윈도우는 배터리 관리가 잘 되는 반면, Ubuntu는 배터리 관리와 관련된 옵션이 별로 없음을 알 수 있다. 이러한 문제를 쉽게 해결할 수 있는 방법이 TLP라는 패키지를 사용하는 것이다. 

TLP는 한번 설치해 두면 알아서 전원을 관리하며, 재부팅 시에도 자동으로 적용되기 때문에 한번 설치해두고 잊어버리면 됩니다.

실제 TLP가 하는 역할은 다음과 같다고 합니다 (참고 포스팅 및 공식 홈페이지 참고)

#### 전원 공급원에 따른 설정

* "터보 부스트"/ "터보 코어"를 포함한 프로세서 속도 조절
* 멀티 코어/하이퍼 스레딩을 위한 전원 인식 프로세서 스케쥴러
* 하드 디스크 고급 전원 관리 레벨, 수준과 스핀다운 동작시간
* SATA 적극적 연결 관리 설정
* PCI express 활성 상태 전원 관리 - 리눅스 커널 2.6.35버전이나 그이상
* 라데온 KMS 전원관리
* 라데온 동적 전원 관리
* WIFI 전원 절약 모드
* 오디오 전원 절약 모드

#### 추가기능

* I/O 스케쥴러
* 블랙리스트 기능을 포함한 USB 자동 중지 기능
* 시스템 시작이나 종료 시 메인보드 통합 wifi, 블루투스, WWAN장치 활성화/비활성화
* Wake On Lan 기능 끄기
* 인텔 프로세서 전압 낮추기

### 설치방법

```
1. 저장소 추가
sudo add-apt-repository ppa:linrunner/tlp
2. apt update
sudo apt-get update
3. tlp 설치
sudo apt-get install tlp tlp-rdw
4. 싱크패드 사용자만! (싱크패드를 사용하는 경우 배터리 성능 효과를 위해 추가 설치)
sudo apt-get install tp-smapi-dkms acpi-call-tools
5. TLP 시작
sudo tlp start
```

한번 시작해 두면 OS를 재시작 해도 자동으로 적용됩니다.

### 추가 디테일 설정

일반적으로 잘 모르는 경우에는 기본 설정대로 사용하는걸 추천하지만, 혹시라도 개인적으로 전압이라던지 기타 설정을 변경하고 싶은 경우에는 다음의 파일을 수정하면 된다.

```
sudo vi /etc/default/tlp
```




[참고 사이트](https://sergeswin.com/954)
