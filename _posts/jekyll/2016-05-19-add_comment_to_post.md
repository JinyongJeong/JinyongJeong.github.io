---
layout: post
title: 'Jekyll에 댓글 추가하기(Disqus)'
tags: [jekyll]
description: >
  Jekyll을 이용하여 만든 블로그의 포스팅에 댓글을 추가하는 방법 소개
---
Jekyll 블로그는 기본적인 layout만을 제공해 주기 때문에 블로그 포스팅에 댓글을 달 수 있도록 만들기 위해서는 추가적인 작업이 필요하다. 댓글창을 만들기 위해 [Disqus](https://disqus.com/)를 사용할 것이다. 

# 1. Disqus 계정 만들기
<img align="middle" src="/images/post/jekyll/add_comment_to_post/fig1.png" width="500" height="500px">

첫번째로 [Disqus](https://disqus.com/)에 접속하여 회원가입 후 등록을 한다. 등록 후 Main 페이지에서 우측 상단의 톱니바퀴를 누르면 나오는 메뉴 중 `Setting`으로 이동 후 다시 `Add Disqus To Site`로 이동한다. 페이지 이동 후 페이지 가장 아래쪽으로 이동하면 `GET STARTED`를 눌러 블로그 명 및 URL을 등록한다. 


# 2. Universal Code 복사하기
<img align="middle" src="/images/post/jekyll/add_comment_to_post/fig2.png" width="800" height="500px">


등록을 마친 후 Setting의 Installation page로 이동 후 platform을 선택한다. Jekyll에 추가하기 위해서는 Universal code를 선택한다. 이제 1번에 있는 코드를 복사하여 Jekyll의 `_layouts/post.html` 파일에 추가해준다. 


# 3. Layout/post.html 및 config 파일 수정


`_layouts/post.html` 파일을 열고 `<article>` 블락 사이에 복사한 내용을 아래 코드에서 `<insert your code>` 부분에 넣어준다. 

```html
{% raw %}
---
layout: default_post
---
<article class="post" itemscope itemtype="http://schema.org/BlogPosting">
  <header class="post-header single-post-header" style="background-image:url('{{ site.url }}{{ page.image }}')">
    <div>
      <h1 class="post-title single-post-title" itemprop="name headline">{{ page.title }}</h1>
      <p class="post-meta single-post-meta">
        <time datetime="{{ page.date | date_to_xmlschema }}" itemprop="datePublished">{{ page.date | date_to_long_string }}</time>
        •
        {% assign words = page.content | number_of_words %}
        {% if words < 360 %}
          1 min
        {% else %}
          {{ words | divided_by:180 }} mins
        {% endif %}
        read
      </p>
    </div>
  </header>
  <div class="wrapper">
    <div class="single-post-summary">
     <code> {{ page.summary }} </code>
    </div>
    <div class="post-content single-post-content" itemprop="articleBody">
     <code> {{ content }} </code>
    </div>
    {% if page.comments %}
        <-- add your code here -->
    {% endif %}
  </div>
</article>
{% endraw %}

```


포스팅에서 comment활성화 여부를 설정하기 위해서 위의 예의 주석으로 되어 있는 `{% raw %}{% if page.comments %}{% endraw %}` `{% raw %}{% endif %}{% endraw %}` 이 부분을 추가해준다. 추가할 때 앞 뒤로 중괄호({})를 넣어준다. 다음으로 `_config.yml` 파일을 열어 마지막에 다음의 코드를 추가한다. 

```
disqus_shortname: myblog(등록한 본인 블로그 이름)
```

# 4. Post에 명령 추가

마지막으로 포스팅을 할 때 포스팅 `.md` file의 맨 위에 `comments: true` 를 추가해 주면 포스팅에 댓글쓰기가 다음과 같이 활성화 된다. 

<img align="middle" src="/images/post/jekyll/add_comment_to_post/fig3.png" width="800" height="200px">



