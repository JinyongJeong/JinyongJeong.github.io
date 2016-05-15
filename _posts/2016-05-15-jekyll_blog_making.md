---
layout: post
title:  "Jekyll을 이용하여 Github Blog 만들기"
date:   2016-05-15 13:33:49 +0900
categories: jekyll update
---
# 1. curl/rvm(ruby version manager) 설치

````
sudo apt-get install curl
curl -sSL https://get.rvm.io | bash -s stable
````

위 코드의 설치 과정에서 만약 다음과같은 error가 뜬다면
{% highlight ruby %}
curl: (77) error setting certificate verify locations:
CAfile: /etc/pki/tls/certs/ca-bundle.crt
CApath: none
{% endhighlight %}

certificate 경로가 맞지 않는 것이다. 다음의 코드를 입력하여 경로 생성 후 `.crt` 파일을 복사해준다. 만약 `.crt`파일이 없다면 `sudo apt-get install ca-certificates`를 통해 설치한다.

````
sudo mkdir -p /etc/pki/tls/certs
sudo cp /etc/ssl/certs/ca-certificates.crt /etc/pki/tls/certs/ca-bundle.crt
````

에러가 또 뜬다면 다음을 입력

````
command curl -sSL https://rvm.io/mpapis.asc | gpg --import -
````

rvm 스크립트를 현재 쉘에 로드 및 현재 쉘에 로드

````
source ~/.rvm/scripts/rvm
rvm requirements
````

만약약 다음과 같은 error가 발생한다면
{% highlight ruby %}
E: Failed to fetch http://kr.archive.ubuntu.com/ubuntu/pool/main/g/gawk/gawk_4.0.1+dfsg-2.1ubuntu2_amd64.deb Could not connect to kr.archive.ubuntu.com:80 (103.22.220.133). - connect (111: Connection refused)
{% endhighlight %}

setting의 software updates에서 Download from 을 main server 로 바꿔준 후 다시 `rvm requirements` 실행

# 1. Ruby 설치

ruby 2.1.2 설치 후 디폴트로 설정

````
rvm install 2.1.2
rvm use 2.1.2 --default
````

# 1. nodejs 설치

````
sudo apt-get install nodejs
````

# 1. Jekyll 설치 및 버전 확인

````
gem install jekyll
jekyll -v
````

여기까지 Jekyll 설치 완료

# 1. jekyll의 설치를 확인

Jekyll을 이용하여 기본 테마를 생성 후 서버를 동작시킨다.

````
jekyll new myblog
cd myblog
jekyll serve
````

`http://localhost:4000`

으로 들어가면 local 블로그를 확인할 수 있다.

# 1. github의 repository를 생성

www.github.com 으로 들어간 후 새로운 repository 생성. yourname.github.io 로 repository 생성 후 로컬에서 home folder로 이동하여 repository 복사

````
git clone https://github.com/yourname/yourname.github.io.git
````

을 하여 폴더를  복사 후  그 폴더로 이동한 후 git 을 초기화시킨다. 초기화 시킨 후 git의 폴더에 jekyll의 기본 테마 설치

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

