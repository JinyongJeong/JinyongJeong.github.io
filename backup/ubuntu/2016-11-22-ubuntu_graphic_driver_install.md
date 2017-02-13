---
layout: post
title: 'Ubuntu NVIDIA Graphic Driver 설치(nouveau kernel 문제 등)'
tags: [ubuntu]
description: >
  Ubuntu NVIDIA Graphic Driver 설치(nouveau kernel 문제 등)
---
# 그래픽 드라이버 설치하기 
우분투를 처음 설치하고 나서 NVIDIA 그래픽 드라이버를 설치하는 과정에서 많은 사람들이 다양한 문제를 경험한다.

우선 NVIDA 홈페이지에 들어가서 본인의 시스템에 맞는 그래픽 드라이버를 최근 3가지 종류를 미리 다운받아 놓는것을 추천한다.

# 1. 시스템 재부팅


```
sudo reboot
```

명령어를 이용하여 시스템을 재 부팅해준다.

# 2. 터미널 모드로 진입

로그인 화면이 나타나게 되면 `ctrl+alt+F1` 버튼을 눌러 터미널 모드로 진입한다.

# 3. lightdm 끄기

NVIDA 드라이버 설치에 앞서 화면출력을 제어하는 Lightdm을 꺼줘야 한다.

```
sudo /etc/init.d/lightdm stop
```

위의 명령어로 lightdm 을 꺼준다

참고 페이지
<http://askubuntu.com/questions/295772/sudo-service-lightdm-stop-will-go-into-a-blank-screen>

# 4. 드라이버 설치

Download 폴더에 드라이버가 저장되어 있는 경우 해당 폴더로 이동하여 설치한다.

```
sudo ./NVIDA*.run
```

그래픽 드라이버 실행 시 관리자 권한으로 실행해야 설치가 된다.

만약 다운받은 드라이버가 실행파일로 되어 있지 않은 경우 

```
chmod 777 ./NVIDIA*.run
```

을 실행하여 실행가능한 파일로 바꿔준다.

# 5. nouveau 끄기

경우에 따라서 nouveau kernel이 실행중이라서 설치를 할수 없다는 경고 메세지가 나오는 경우가 있다.

이런 경우는 아래의 사이트를 참고하자
<https://altinukshini.wordpress.com/2011/07/28/how-to-install-nvidia-drivers-in-ubuntu-linux/>

우선 `/etc/modprobe.d/blacklist.conf` 파일을 열어

```
blacklist nouveau
```

를 가장 아래에 추가해준다. 

그다음 다음의 명령어로 nouveau를 비활성화 시킨다.

```
echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
update-initramfs -u
```

여기까지 진행한 후 재부팅하고 다시 sudo 권한으로 그래픽 드라이버를 설치하면 설치가 될 것이다.

위의 사이트에서는 위에 언급한 과정 말고 다른 부가적인 과정이 서술되어 있으나

본인은 이정도로도 설치가 되었다. 만약 안된다면 위의 사이트를 좀더 자세히 참고하길 바란다. 


