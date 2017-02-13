---
layout: post
title: 'Jekyll Markdown에서 HTML소스코드 그대로 포스팅하기'
tags: [jekyll]
description: >
  Markdown을 이용하여 포스팅 시 HTML 코드를 그대로 포스팅하는 방법
---

Jekyll을 이용하여 포스팅을 하면서 코드를 올리는 경우가 종종 있다. C 나 python 혹은 기타 코드를 code block에 넣으면 코드가 그대로 보이지만 HTML을 넣을 경우 실제 데이터로 convert가 되어 표기가 되어 원하는 코드를 재대로 보여줄 수 없다. 이때는 code block에 `{ % raw % }` 와 `{ % endraw % }`를 넣어줌으로써 code의 raw를 변환없이 아래와 같이 포스팅 할 수 있다. 실제 입력 시 `{` 와 `%` 사이에는 공백이 없어야 한다. 



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

