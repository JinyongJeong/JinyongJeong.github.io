---
layout: post
title: 'Windows Partition Auto Mount in Ubuntu 14.04'
tags: [ubuntu]
description: >
  우분투 14.04에서 NTFS형식으로 되어있는 윈도우 파티션을 자동 마운트 하는 방법 소개
---
윈도우 파티션의 파일시스템은 NTFS(New Technology File System)으로 보통 구성되어 있다. 예전에는 Ubuntu에서 NTFS 시스템을 읽거나 사용하기 어려웠는데 근래에는 Reverse Engineering을 통해 Ubuntu에서도 사용 가능할 수 있게 되었다. 기본적으로 가장 오래된 시스템은 FAT(File Allocate Table)이며 안정성 면에서는 FAT32가 가장 좋다. 하지만 FAT32는 32bit 시스템을 사용하기 떄문에 4Gbyte 이상의 파일을 저장할 수 없다. 왜냐하면 32bit 방식은 표현할 수 있는 주소의 갯수가 2의 32승 = 4.2950e+09 이고  한개의 주소값으로 1Byte를 표현 할 수 있으므로 32bit 시스템으로 표현할 수 있는 최대 메모리는 4.2950e+09 Byte, 약 4GByte이다. 마찬가지로 32bit 운영체제에서 최대 사용가능한 메모리 용량이 4GByte인것도 같은 이유이다. 32bit 시스템에서 4GByte이상의 하드디스크를 사용할 수 있는 것은 하드시스크의 data를 RAM으로 이동 후 처리하기 때문에 가능한 것이다. 이러한 한계 때문에 개발 된 것이 exFAT시스템이다. exFAT시스템은 용량의 제한이 매우 커서 따로 신경쓸 필요가 없다. 하지만 exFAT 시스템으로 설정된 USB 및 디스크를 안전하게 제거하지 않는다거나 여러 경우에 불안한 모습을 보일 때가 있다고 한다. 마지막으로 NTFS 방식은 Windows에서 개발된 방식이기 때문에 블랙박스라던지 기타 디바이스에서 인식이 안되는 문제가 있다고 한다. 

다시한번 정리하면

FAT32 : 안전성에서는 가장 좋음, 저장 용량의 한계가 있음 

exFAT : 안정성이 떨어짐, 저장용량의 한계는 없음

NTFS: 확장성이 떨어짐

따라서 디스크 포멧 시 본인의 맞는 방식에 맞춰서 사용하면 된다. 기본적으로 Windows가 설치된 디스크는 NTFS 방식으로 되어 있기 때문에, 윈도우가 설치된 디스크를 Ubuntu에 자동으로 mount 시키기 위해서는 다음과 같은 절차가 필요하다.

## 1. directory 생성

우선 원하는 위치에 directory를 설정한다. 

```
sudo mkdir /media/win
```

## 2. fstab file수정

`/etc` 경로의 fstab file을 수정한다. 

```
sudo gedit /etc/fstab
```

우선 fstab file을 열고 다음을 추가한다. 

```
UUID=16D219F9D219DE35 /media/win ntfs auto,defaults,rw 0 2

```

또는

```
/dev/sda1 /media/win ntfs-3g auto,defaults,rw 0 2
```

위의 두가지 방식으로 설정 가능하다(device 이름 혹은 UUID)

`/media/win` : mount 위치

`ntfs-3g` : file type

`rw` : read and write

fstab까지 설정이 완료 후 재부팅하면 windows의 파티션이 자동으로 mount되는 것을 확인 가능하다.

## 추가

어느순간 잘 되다가 윈도우 파티션을 mount할 수 없다고 뜨는 경우가 있다. 이런경우는 종종 윈도우가 정상적으로 종료되지 않았을 경우이기 때문에 윈도우로 갔다가 돌아오면 정상적으로 mount가 되는데, 한번씩 윈도우 파티션이 hibernated 파티션이라고 뜨면서 안되는 경우가 있다. 이런경우에는 윈도우를 켰다가 끄고 재 접속을 해도 auto mount가 되지 않는다. 이런경우는 위의 설정은 변경하지 않고 아래의 명령어를 이용하여 hibernated를 풀어주면 된다. 

```
sudo mount -t ntfs-3g -o remove_hiberfile /dev/sda1 /media/win
```

그래도 안되는 경우에는 Window에서 hibernate 옵션을 강제로 꺼준다. 윈도우를 키고 cmd 창을 관리자 모드로 실행 시킨 후 다음 명령어를 실행시킨다.

```
powercfg -h off
```


