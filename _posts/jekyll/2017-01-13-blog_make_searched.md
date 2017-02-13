---
layout: post
title: 'github blog를 google에서 검색되도록 설정하기'
tags: [jekyll]
description: >
  github io와 같은 개인 블로그를 google과 naver, 그리고 daum과 같은 포탈사이트에서 검색 가능하도록 만드는 방법
sitemap :
  changefreq : weekly
  priority : 1.0
---

이 글은 다음 [blog](http://dveamer.github.io/homepage/SubmitSitemap.html)의 글을 참고하였습니다.

네이버 블로그와 같은 포탈의 블로그 서비스를 사용할 경우 자동으로 검색이 가능하지만, github와 같은 플렛폼을 사용할 경우에는 직접 각 포탈에 검색이 가능하도록 등록을 해 주어야 한다. 이 글에서는 블로그의 글이 google, daum, naver에서 검색 가능하도록 등록하는 방법에 대해서 설명한다. 

## 1. sitemap 생성

sitemap을 google에 등록해 두면 주기적으로 크롤링을 통해 url을 연결시킨다. 우선 sitemap을 생성하는 방법에 대해서 설명한다. ruby를 통해 jekyll 홈페이지를 만든 경우에는 `sudo gem install jekyll-sitemap` 의 명령어를 이용해 플러그인을 사용할 수 있다. 하지만 여기서는 플러그인을 사용하지 않는 방법을 설명한다. 

블로그의 `/root` 경로에 `/sitemap.xml` 파일을 만들고 아래의 내용을 복사해 넣는다. 반드시 root 디렉토리에 넣어야 한다. 

```
{%raw%}
---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9 http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd" xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  {% for post in site.posts %}
    <url>
      <loc>{{ site.url }}{{ post.url }}</loc>
      {% if post.lastmod == null %}
        <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>
      {% else %}
        <lastmod>{{ post.lastmod | date_to_xmlschema }}</lastmod>
      {% endif %}

      {% if post.sitemap.changefreq == null %}
        <changefreq>weekly</changefreq>
      {% else %}
        <changefreq>{{ post.sitemap.changefreq }}</changefreq>
      {% endif %}

      {% if post.sitemap.priority == null %}
          <priority>0.5</priority>
      {% else %}
        <priority>{{ post.sitemap.priority }}</priority>
      {% endif %}

    </url>
  {% endfor %}
</urlset>
{%endraw%}
```

git과 commit으로 블로그를 업데이트 후 `blog주소/sitemap.xml`로 접속했을 때 아래와 같은 화면이 나와야 정상적으로 sitemap이 등록된 것이다. 

<img align="middle" src="/images/post/jekyll/google_search/sitemap.png" width="700">

sitemap에는 각 해당 글의 lastmod, sitemap.changefreq, sitemap.prioritye 등의 정보가 설정되는데, 이것은 각 글의 맨 위에 다음과 같이 sitemap의 옵션을 추가해 줌으로써 추가적으로 설정 가능하다. 설정이 없을 때의 default 설정은 `sitemap.xml`에 정의되어 있다. 

```
{%raw%}
---
layout: post
title:  "제목"
date:   2016-03-14 12:00:00 
lastmod : 2016-03-15 12:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---
{%endraw%}
```

changefreq를 너무 짧게 하면 빈번한 접속으로 안좋은 영향을 미칠 수도 있다고 하니 적당히 하루 혹은 일주일로 하면 좋을 것 같다. 추가적으로 `blog주소/sitemap.xml`을 실행했을 때 위와 같이 나오지 않는 경우는 아마 주소링크에 &와 같은 특수기호가 있는 경우가 있을 수 있다. 예를들어 파일의 이름이 URL의 링크 주소가 되는데, 만약 파일이름이 한글일 경우 url의 주소에 %의 기호가 들어가 있다. 이럴경우 xml이 정상적으로 해석하지 못한다. 따라서 최대한 URL의 링크가 되는 파일이름은 영문으로 만들고, 특수기호는 최대한 사용하지 않는 것이 좋다. 

## 2. RSS feed 생성

Rss feed는 naver와 daum에 등록하기 위함이다. `sitemap.xml`과 마찬가지로 root 디렉토리에 `/feed.xml`파일을 생성하고 아래의 코드를 복사한다. 

```
{%raw%}
---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ site.title | xml_escape }}</title>
    <description>{{ site.description | xml_escape }}</description>
    <link>{{ site.url }}{{ site.baseurl }}/</link>
    <atom:link href="{{ "/feed.xml" | prepend: site.baseurl | prepend: site.url }}" rel="self" type="application/rss+xml"/>
    <pubDate>{{ site.time | date_to_rfc822 }}</pubDate>
    <lastBuildDate>{{ site.time | date_to_rfc822 }}</lastBuildDate>
    <generator>Jekyll v{{ jekyll.version }}</generator>
    {% for post in site.posts limit:30 %}
      <item>
        <title>{{ post.title | xml_escape }}</title>
        <description>{{ post.content | xml_escape }}</description>
        <pubDate>{{ post.date | date_to_rfc822 }}</pubDate>
        <link>{{ post.url | prepend: site.baseurl | prepend: site.url }}</link>
        <guid isPermaLink="true">{{ post.url | prepend: site.baseurl | prepend: site.url }}</guid>
        {% for tag in post.tags %}
        <category>{{ tag | xml_escape }}</category>
        {% endfor %}
        {% for cat in post.categories %}
        <category>{{ cat | xml_escape }}</category>
        {% endfor %}
      </item>
    {% endfor %}
  </channel>
</rss>
{%endraw%}
```

## 3. robots.txt 생성

`robots.txt`파일에 `sitemap.xml`파일의 위치를 등록해 두면 검색엔진의 크롤러들이 홈페이지를 크롤링하는데 도움을 주게 된다고 한다. root 디렉토리에 `/robots.txt` 파일을 만들고 아래와 같이 입력한다. 

```
User-agent: *
Allow: /

Sitemap: http://jinyongjeong.github.io/sitemap.xml
```

`User-agent`는 허용할 검색엔진 명을 넣게 된다. 따로 설정하지 않으면(*) 모든 검색엔진을 허용하게 된다. 자세한 내용은 <http://dveamer.github.io/homepage/SubmitSitemap.html>를 참고한다. 


## 4. 사이트 등록

### google (google search console등록)

[Google Search Console](https://www.google.com/webmasters/#?modal_active=none)를 접속한다. 

이 사이트에서 본인의 블로그를 등록해야 google에서 검색이 가능하다. 
`속성추가` 버튼을 눌러 본인의 blog 주소를 입력하여 사이트를 등록한다. 
`크롤링 > sitemaps` 메뉴를 열어 `sitemap 추가` 버튼을 눌러 만들어 두었던 `sitemap.xml`파일을 제출한다. 
제출이 완료되면 `sitemap.xml`파일이 등록된 것을 확인할 수 있으며 색인이 접수 중임을 알 수 있다. 


### naver

[네이버 웹마스터 도구](http://webmastertool.naver.com/)에 접속한다. 

로그인하여 구글과 비슷하게 블로그 주소를 등록하는 과정을 거친다. 그 후 "사이트 소유 확인"이라는 과정을 거치게 되는데 HTML 파일을 다운받아 블로그의 root에 업로드 하여 확인하는 과정을 거치게 된다. 이 과정을 거치면 google의 analystics와 유사한 기능을 사용할 수 있는 것 같다. 그 다음에 RSS를 등록하는 과정이 필요하다. 왼쪽 메뉴에서 `요청 > RSS제출` 을 클릭해서 URL을 포함한 주소인 `블로그URL/feed.xml`을 입력한다. 
추가적으로 `요청 > 사이트맵제출`로 들어가서 google과 마찬가지로 `블로그URL/sitemap.xml`을 입력해서 sitemap을 등록시켜 준다. 

### daum

[DAUM 검색등록](https://register.search.daum.net/index.daum)에 접속 후 로그인한다. 

`등록` 탭에서 `블로그 RSS등록`을 선택하고 `블로그URL`에 본인의 URL을 입력하고 확인버튼을 누른다. 
이전 블로그에서 보면 `feed.xml`파일을 올리는 RSS 부분이 있었는데 지금 확인해보니 URL을 입력하는 창만 뜬다. 
우선 URL만 등록해보도록 한다. 

위의 과정을 거치고 최종으로 반영되기 까지 어느정도의 시간이 걸린다. 일주일정도 시간이 흐른뒤에 확인해보도록 하자. 

