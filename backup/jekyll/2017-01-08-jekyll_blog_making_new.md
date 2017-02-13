---
layout: post
title: 'Jekyll을 이용하여 Github Blog 만들기'
tags: [jekyll]
description: >
  Jekyll을 이용한 간편하게 Github에 blog를 만드는 방법 소개
---

## Ruby와 Jekyll 설치
Jekyll을 이용하여 github의 Blog를 만들어보도록 하자. 우선 Jekyll을 설치하기 위해서는 ruby를 설치해야 한다. 아래명령어를 이용하여 ruby를 설치한 후 Jekyll을 설치한다. 

```
sudo apt-get update
sudo apt-get install ruby-full
sudo gem install jekyll
```

Jekyll이 설치되었으면 설치가 잘 되었는지 확인하기 위해 버전을 확인한다.

```
jekyll -v
```

각 blog의 종류에 따라 다르지만 아래의 명령어를 통해 추가 패키지를 설치해주자.

```
sudo gem install jekyll-sitemap
sudo gem install jekyll-paginate
```

## github의 repository를 생성

[Github](http://www.github.com)으로 접속 후 새로운 repository 생성. <yourname>.github.io 로 repository 생성 후 로컬에서 home folder로 이동하여 repository 복사.

````
git clone https://github.com/<yourname>/<yourname>.github.io.git
````

폴더를  복사 후  그 폴더로 이동한 후 git 을 초기화시킨다. 초기화 시킨 후 git의 폴더에 jekyll의 기본 테마 설치

````
git init
jekyll new . --force 
````

Jekyll로 생성된 모든 파일을 git에 add해 준 후 commit, push 를 해서 github에 올려준다.

````
git add *
git commit -m "first commit"
git push -u origin master
````

첫 commit 시에 config 파일에 user.email과 user.name을 등록하라고 나오면 

````
git config user.email "이메일주소"
git config user.name "이름"
````

을 입력하여 이메일과 이름을 등록해 준다.이로써 github에 블로그가 생성되었다!

git을 clone 받은 후 처음에는 `git push -u origin master`를 모두 입력해야 하지만 한번 한 이후에는 `git push`만 입력해도 된다.

## Blog 작성

블로그 작성 시 입력한 내용이 예상한대로 동작을 하는지 확인 하기 위해서 매번 github에 push하는 것은 매우 힘든 일이다. 작성을 하는 동시에 바로 확인을 하기 위해서 위에서 언급한 것과 같이 `jekyll serve`를 이용하여 서버를 동작 시킨 후 `localhost:4000`으로 접속하면 바로 수정된 내용을 확인할 수 있다. 이렇게 수정이 완료된 후 github로 push하면 편하게 포스팅을 수정할 수 있다.
