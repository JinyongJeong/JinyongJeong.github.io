---
layout: post
title: 'Ubuntu shell file make desktop application (ROS)'
tags: [ubuntu]
comments: true
description: >
  우분투 shell 파일 desktop 파일로 등록하기 (ROS 환경)
sitemap :
  changefreq: weekly
  priority : 1.0
---
우분투에서 Clion 과 같은 프로그램을 설치하면 프로그램을 시작하기 위해서는 shell로 구동을 시작해야 한다. 이런 경우 터미널을 열고 실행을 해야하기 때문에 실수로 터미널을 끄게 되면 프로그램이 꺼지게 된다. 상당히 불편하다. 이런경우 프로그램을 실행하는 shell파일을 desktop 프로그램으로 등록하고, 윈도우키를 누르고 실행하면, 터미널 실행없이 바로 실행시킬 수 있다.  

```bash
sudo gedit /usr/share/applications/clion.desktop
```

만약 위의 경로에 설정을 하면 모든 사용자에게 적용되며, local 사용자에게만 적용시키고 싶은 경우 다음의 경로에 추가한다. 

```bash
sudo gedit /home/user/.local/share/applications/clion.desktop
```

`/usr/share/applications` 폴더에는 다양한 desktop application들이 등록되어 있다. 위의 파일을 생성 후 다음과 같이 작성해 준다. 

```bash
[Desktop Entry]
Type=Application
Terminal=false
Name=clion
Icon=/path/to/icon/icon.svg
Exec=/home/jjy0923/CLion-2020.1.2/clion-2020.1.2/bin/clion.sh
```

`Terminal`은 프로그램 실행 시 terminal을 열지 말지를 설정할 수 있다. `Exec=`는 실행시키고자 하는 shell 파일을 지정해 주면 된다. `Icon=`에는 프로그램의 icon을 설정해 주면 되는데 따로 지정하고 싶지 않은 경우는 삭제해버리면 된다. `Nam=`은 프로그램을 시작하기 위한 application 이름이다. 

여기까지는 일반적인 등록 방법이다. 하지만 만약 clion에서 ROS를 사용한다면 조금 달라진다. 터미널을 열고 clion을 실행시킬 때는 `.bashrc`에 등록되어 있는대로 `source /opt/ros/melodic/setup.bash`가 실행되기 때문에 문제가 없지만, desktop으로 실행시키게 되면 ros 환경이 불러와지지 않는다. 따라서 ROS를 사용하는 환경이라면 다음과 같이 `Exec`를 수정해 준다. 

```bash
[Desktop Entry]
Type=Application
Terminal=false
Name=clion
Icon=/path/to/icon/icon.svg
Exec=bash -c "source /opt/ros/melodic/setup.bash && /home/jjy0923/CLion-2020.1.2/clion-2020.1.2/bin/clion.sh"
```

당연히 `source /opt/ros/melodic/setup.bash` 에서 `melodic`은 본인이 사용하고 있는 ROS 버전으로 변경해줘야 한다. 

위와 같이 desktop application을 등록하게 되면 윈도우키를 누르고 다음과 같이 프로그램을 검색해서 실행할 수 있다. 

![/images/post/ubuntu/2020-06-23-Ubuntu-shell-file-make-desktop-application/Screenshot_from_2020-06-23_11-39-53.png](/images/post/ubuntu/2020-06-23-Ubuntu-shell-file-make-desktop-application/Screenshot_from_2020-06-23_11-39-53.png)
