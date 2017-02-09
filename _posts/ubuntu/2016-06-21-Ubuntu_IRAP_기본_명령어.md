---
layout: post
title: 'Ubuntu SVN 및 기타 명령어 정리 (IRAP)'
tags: [ubuntu,IRAP]
description: >
  IRAP에서 주로 사용하는 svn 및 기타 ubuntu 명령어 정리
---
본 포스팅은 한국과학기술원(KAIST)의 [IRAP](http://irap.kaist.ac.kr)(Intelligent Robotic Autonomy and Perception) 연구실에서 주로 사용하는 Ubuntu 명령어들을 정리하였다.

## 1. ssh 서버 접속

```
ssh 계정이름@irap.kaist.ac.kr
```

## 2. 데이터 복사

```
rsync -avPh 계정이름@irap.kaist.ac.kr/경로 복사할local폴더
```

## 3. SVN Check out

```
svn co svn+ssh://계정이름@irap.kaist.ac.kr/경로/trunk 복사할local폴더
```

## 4. SVN commit

```
svn ci
```

## 5. SVN 서버의 계정명과 컴퓨터 local 이름이 다른경우

`.ssh` 폴더로 이동한다.

```
cd ~/.ssh
```

`config` 파일을 만들고 내용을 추가해준다.

```
touch config
vi config
```

아래의 내용 추가

```
Host irap.kaist.ac.kr
User each_user_name
```

## 6. SVN 버전관리 예외 처리

SVN 폴더내에서 특정 확장자 파일 혹은 특정 파일에 대한 버전관리 예외처리를 할 수 있다.

SVN home 폴더 위치에서 다음의 명령어 실행

```
svn propedit svn:ignore .
```

다음을 실행하면 nano editor가 열리며 ignore에 넣을 파일명을 입력할 수 있다. Latex svn의 경우

```
*.aux
*.log
*.synctex
*.bbl
*.blg
*.gz
paper.pdf
```

위와 같이 입력해 놓으면 각 확장자 파일과 `paper.pdf` 파일은 svn 버전관리에 대해서 예외처리 된다.

## 7. SVN 로컬 파일 변경 취소

```
svn revert filename
```

위와 같이 입력하면 local에서의 파일 변경이 취소된다.

## 8. SVN commit시 PDF 파일 binary로 인식 차단

svn commit을 하게 되면 각 파일의 수정사항이 svn server에 등록되어 있는 메일로 전달이 되는데 이 과정에서 pdf 파일의 경우 binary로 인식되어 사람이 알아 볼 수 없는 형태로 전달된다. 따라서 commit을 할 때 pdf파일은 binary로 인식되지 않도록 다음 명령어를 이용한다.

```
svn propset svn:mime-type application/octet-stream <filename>
```

## 9. SVN tag 하기

SVN의 특정 버전을 tag하고 싶을 때는 다음의 명령어를 이용해서 tag를 만든다. 일반 copy를 사용하지 않고 `svn copy`를 사용 해야한다.

```
svn copy svn+ssh://계정명@irap.kaist.ac.kr/경로/trunk svn+ssh://계정명@irap.kaist.ac.kr/경로/tags/tag명

```

주의해야 할 점은 내 컴퓨터에 있는 폴더가 아닌, 서버에 있는 폴더를 직접 복사한다는 점이다.

## 10. LCM 환경설정

### LCM ttl 설정

```
sudo vi /etc/environment
```

/etc 경로의 environment 파일을 열어 다음줄을 추가한다.

```
export LCM_DEFAULT_URL=udpm://239.255.76.67:7667?ttl=0
```

ttl을 0으로 설정하는 것이며, ttl이 0일 때는 publish된 lcm message가 local computer 내에서만 공유된다. ttl을 1로 설정했을 경우는 외부로 까지 cast 된다. 

### Buffer size 수정

LCM을 이용하여 대용량의 파일을 전송하기 위해서 MAX와 Default buffer 사이즈를 수정한다.

```
sudo vi /etc/sysctl.conf
```

위 파일을 열고 다음을 맨 아래에 추가한다.

```
net.core.rmem_max=20971520
net.core.rmem_default=20971520
```

각각 buffer 사이즈를 조절하는 것인데, 컴퓨터마다 최대값이 다르기 떄문에 더 높일 수도 있다. 

## 11. MTU 설정

MTU는 이더넷의 전송 packet의 사이즈이다. MTU 값이 작으면 작은 packet 사이즈로 빠른 속도로 packet을 전송하고, 크면 큰 packet사이즈로 느리게 전송한다. 일반적으로 MTU의 default값은 1500이다.

현재 연구실에서 센서 시스템에서 많은 데이터를 한번에 받아오는 경우가 많기 때문에 이런 경우에는 MTU값을 늘려주어야 한다. 다음의 명령어를 이용하여 MTU를 늘릴 수 있다.

```
ifconfig eth0 mtu 8000
```

`ifconfig`를 실행하면 현재 컴퓨터의 이더넷 연결 상태를 확인할 수 있으며 현재 설정되어 있는 MTU값도 확인 가능하다. 하지만 무조건 mtu값이 높은 것이 좋은 것은 아니다. 실제 데이터의 packet 사이즈가 작은데 mtu가 너무 높은 경우 속도가 느려질 수도 있다. 


## 12. 폴더 및 파일 권한 설정

```
chmod 777 filename
```

777은 각각 (소유자,그룹,방문자)의 권한을 설정한다. 순서는 rwx(read,write,execute) 로 7은 111(4+2+1)을 의미한다. 관리자 권한이 필요한 경우에는 앞에 `sudo` 를 붙여준다.

## 13. 소유자, 그룹 설정

### 소유자 변경

```
chown 소유자명 파일명
```

### 그룹변경

```
chgrp 그룹명 파일명
```

관리자 권한이 필요한 경우에는 앞에 `sudo`를 붙인다.

## 14. '/usr/bin/ld: cannot find -IGL' 이라는 에러가 뜰 경우

이런 경우는 libGL.so 파일을 재대로 찾지 못하는 경우이다 이런경우는 아래의 명령어로 링크를 다시 걸어준다.

```
sudo rm /usr/lib/x86_64-linux-gnu/libGL.so 
sudo ln -s /usr/lib/libGL.so.1 /usr/lib/x86_64-linux-gnu/libGL.so 
```



