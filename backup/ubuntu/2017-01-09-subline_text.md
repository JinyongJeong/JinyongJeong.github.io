---
layout: post
title: 'Ubuntu Sublime text 설치하기'
tags: [ubuntu, IRAP]
description: >
  Sublime text 설치 후 셋팅하는 방법을 설명한다. 
---


# SublimeText3

### 1. sublime text 설치하기

##### [sublimetext](http://www.sublimetext.com/) 에서 다운로드 후 설치

##### ppa 이용한 설치 방법


```
sudo add-apt-repository ppa:webupd8team/sublime-text-3
sudo apt-get update
sudo apt-get install sublime-text-installer
```

### 2. Project 를 위한 패키지 구성

##### Package Control 설치

메뉴 View -> Show Consol (Ctrl+`) 실행후 console 창을 열어서 아래의 명령어 입력후 에디터 재시작
Sublime text 버전에 따라 다르게 입력

Sublime Text 2

```
import urllib2,os; pf='Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); open( os.path.join( ipp, pf), 'wb' ).write( urllib2.urlopen( 'http://sublime.wbond.net/' +pf.replace( ' ','%20' )).read()); print( 'Please restart Sublime Text to finish installation')
```

Sublime Text 3

```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

#### YcmdCompletion 패키지

##### 1. YcmdCompletion 설치

* Sublime Text에서 Command Palette (Ctrl+Shift+P)창을 열어서 'Package Control : Install Pacakge' 실행 후 'YcmdCompletion' 찾아서 설치

* YcmdCompletion은 기본적으로 Ycmd server를 이용해서 code-completion을 수행. ycmd server가 설치 되어있지 않으면 다음 단계로 ycmd 를 설치해야함.

##### 2. Ycmd server 설치

* Ycmd의 github 원격 저장소를 복제후 submodule 업데이트

```
git clone https://github.com/Valloric/ycmd.git
cd ycmd
git submodule update --init --recursive
```

* Ycmd 빌드

```
./build.py --all
```

##### 3. YcmdCompletion 설정

###### HMAC key 생성

Command Pallete (Ctrl+Shift+p) -> Ycmd: Create HMAC keys

###### Sublime Text의 Ycmd Completion 설정

Preferences -> Package Settings -> YcmdCompletion -> Settings - Default 클릭 후 YcmdCompletion.sublime-settings 파일의 주석 처리된 부분을 풀어서 다음과 같이 입력

```
{
    "ycmd_server": "http://127.0.0.1",
    "ycmd_port": 8080,
    "HMAC": "위에서 생성한 HMAC key",
    "use_auto_start_localserver": 1,
    "ycmd_path": "/home/USERNAME/ycmd/ycmd",
    "languages": ["cpp", "python"],
}
```

###### Sublime Text의 Syntax Specific 추가

Preferences -> Settings - More -> Syntax Specific - User 클릭 후 C++.sublime-settings에 다음 내용 추가(c++ 프로젝트 혹은 파일을 열었을 때 C++.sublime-settings을 열 수 있다.

```
{
    "auto_complete_selector": "source - (comment, string.quoted)",
    "auto_complete_triggers": [ 
        {"selector": "source.c++", "characters": "."},
        {"selector": "source.c++", "characters": "::"},
        {"selector": "source.c++", "characters": "->"} 
    ]
}
```

###### Sublime project 생성하기
* CMake를 이용하는 방법

```
cd build
cmake . -G "Sublime Text 2 - Unix Makefiles"
```

* CMakeLists.txt 를 이용하는 방법

project의 root의 CMakeLists.txt에 다음 항목 추가

```
set(CMAKE_EXPORT_COMPILE_COMMANDS "ON")
set(CMAKE_GENERATOR "Unix Makefiles" CACHE INTERNAL "" FORCE)
set(CMAKE_EXTRA_GENERATOR "Sublime Text 2" CACHE INTERNAL "" FORCE)
```

###### Ycmd default_settings.json 파일 설정

위에서 복제한 ycmd server directory에서 default_settings.json 파일 내용 중 아래 부분 변경


```
  "global_ycm_extra_conf": "/home/ycmd/ycmd/.ycm_extra_conf.py",
  "confirm_extra_conf": 0,
  "hmac_secret": "위의 HMAC key 입력",
```

###### [.ycm_extra_conf.py](https://github.com/Valloric/ycmd/blob/master/cpp/ycm/.ycm_extra_conf.py) 파일 설정하기

* gobal ycm_conf 설정 : ycmd server의 default_settings.json 파일의 "global_ycm_extra_conf" 설정 경로에 위 링크 파일 복사
* local ycm_conf 설정 : 해당 프로젝트의 root에 .ycmd_extra_conf.py복사 후 compilation_database_folder 값을 아래와 같이 변경

```
compilation_database_folder = os.path.expanduser("~/projects/naver/build")
```

###### YCM-Generator 이용해서 .ycm_extra_conf.py 만들기

YCM-Generator를 github repository로부터 복제


```
git clone https://github.com/rdnetto/YCM-Generator.git
```

프로젝트 디렉토리에 .ycm_extra_conf.py 만들기

```
cd YCM-Generator
./config_gen.py 프로젝트디렉토리
```

위의 프로그램을 수행하면 프로젝트디렉토리의 root에 .ycm_extra_conf.py파일이 생성됨




