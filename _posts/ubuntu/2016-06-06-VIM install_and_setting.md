---
layout: post
title: 'vim install and setting'
tags: [ubuntu]
description: >
  vim install and setting
---
vim에 관련하여 [다음](http://www.joinc.co.kr/w/Site/Vim/Documents/UsedVim#AEN18) 블로그에 가면 자세한 정보를 얻을 수 있다. 

vim은 우분투에서 사용되는 vi 에디터에 설정을 더하여 보다 편리하게 사용할 수 있도록 해주는 에디터입니다.


우선 vim을 설치해주셔야 합니다.

```
$ sudo apt-get install vim
```

예전에 제가 겪었던 오류사항중에 우분투를 처음 설치하고 바로 vim을 설치하려는 경우, 우분투 저장소 업데이트가 되지 않아서 vim 패키지를 찾지 못하는 오류가 있었던 적이 있습니다.

만약 vim 패키지를 찾을 수 없다. 라는 오류가 뜨신다면 

```
$ sudo apt-get update
```

위의 명령어를 먼저 수행해 주시길 바랍니다.
설치가 완료되었으면 vim 설정을 해줘야합니다.
vim에는 정말 다양한 설정들이 많지만, 우선 제가 사용하는 것들만 소개해 드리겠습니다.

```
$ vi ~/.vimrc
```

위의 명령어를 입력하여 vim 설정파일을 열어줍니다. 아마 처음에는 아무것도 없을 텐데, i 키를 누르고 아래 보이는 설정파일들을 입력하도록 합시다.

```
set number            " line 표시를 해줍니다.
set ai                    " auto indent
set si                    " smart indent
set cindent            " c style indent
set shiftwidth=4      " shift를 4칸으로 ( >, >>, <, << 등의 명령어)
set tabstop=4         " tab을 4칸으로
set ignorecase      " 검색시 대소문자 구별하지않음
set hlsearch         " 검색시 하이라이트(색상 강조)
set expandtab       " tab 대신 띄어쓰기로
set background=dark  " 검정배경을 사용할 때, (이 색상에 맞춰 문법 하이라이트 색상이 달라집니다.)
set nocompatible   " 방향키로 이동가능
set fileencodings=utf-8,euc-kr    " 파일인코딩 형식 지정
set bs=indent,eol,start    " backspace 키 사용 가능
set history=1000    " 명령어에 대한 히스토리를 1000개까지
set ruler              " 상태표시줄에 커서의 위치 표시
set nobackup      " 백업파일을 만들지 않음
set title               " 제목을 표시
set showmatch    " 매칭되는 괄호를 보여줌
set nowrap         " 자동 줄바꿈 하지 않음
set wmnu           " tab 자동완성시 가능한 목록을 보여줌
syntax on        " 문법 하이라이트 킴"
```

참조: "는 주석의 의미입니다.

입력이 끝나고 저장하고 나오시면 이제 코드가 바뀌어있는걸 볼 수 있습니다.

