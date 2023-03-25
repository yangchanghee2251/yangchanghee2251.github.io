---
layout: single
title: "google에 내 URL 노출 시키기 (Google Search Consol) github page, jekyll"
author_profile: true
categories:
  - jekyll
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---

## Google Search Consol?
나도 이런게 있는지 몰랐는데 내 github page를 구글에 검색해봤는데 검색이 안나와서 띄우고자 찾게 되었다.  
[Google search consol](https://search.google.com/search-console/about)은 구글에 내 블로그를 노출 및 검색하면 나올 수 있도록 만들어주는 TOOL이라고 생각하면 된다.

## 어떻게 노출하지?
[yammong blog](https://yammong.github.io/blog/Githubio%EA%B5%AC%EA%B8%80%EA%B2%80%EC%83%89%EB%85%B8%EC%B6%9C%EC%8B%9C%ED%82%A4%EA%B8%B0)나는 이 블로그를 참고해서 진행했으며 google sarch Console 등록은 이 블로그를 참고하길 바란다.

## sitemap.xml 생성
우선 내 블로그를 따라온 사람이라면 jekyll에 minimal mistake를 사용할 것이기 때문에 똑같이 진행하면 문제없이 진행 될거라고 생각이 든다.
우선 `_config.yml` 파일에 url 속성에 블로그 이름을 넣었는지 가장 먼저 확인하고 레포토지에 sitemap.xml 파일을 만들어서 아래와 같은 내용을 추가한다.
```
    ---
    layout: null
    sitemap:
      exclude: 'yes'
    ---
    <?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      {% for post in site.posts %}
        {% unless post.published == false %}
        <url>
          <loc>{{ site.url }}{{ post.url }}</loc>
          {% if post.sitemap.lastmod %}
            <lastmod>{{ post.sitemap.lastmod | date: "%Y-%m-%d" }}</lastmod>
          {% elsif post.date %}
            <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>
          {% else %}
            <lastmod>{{ site.time | date_to_xmlschema }}</lastmod>
          {% endif %}
          {% if post.sitemap.changefreq %}
            <changefreq>{{ post.sitemap.changefreq }}</changefreq>
          {% else %}
            <changefreq>monthly</changefreq>
          {% endif %}
          {% if post.sitemap.priority %}
            <priority>{{ post.sitemap.priority }}</priority>
          {% else %}
            <priority>0.5</priority>
          {% endif %}
        </url>
        {% endunless %}
      {% endfor %}
    </urlset>
```
그리고 root.txt를 만들어야 한다.
```
 User-agent: *
 Allow: /
 Sitemap: {{ '/sitemap.xml' | relative_url | prepend: site.url }}
```
이렇게 저장을 한 후에 google site map을 다시 들어가서 sitemap.xml을 추가해야한다.


<img src="../post_images/사이트맵_추가.png" width="1000px"  title="table1" alt=""/>  
<img src="../post_images/제출된_사이트맵.png" width="1000px"  title="table1" alt=""/>  
위 그림이 나온다면 끝났다고 생각하면 된다.