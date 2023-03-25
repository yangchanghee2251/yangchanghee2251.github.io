---
layout: single
title: "블로그 카테고리 & 카테고리 리스트 & toc 만들기"
author_profile: true
categories:
  - jekyll
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---

## 블로그 카테고리란?
아래 그림과 같이 전체 글 수, categories안에 있는 human, UVI study라든가 만들 수 있는 툴이다.
<img src="../post_images/카테고리.png" width="50%"  title="table1" alt=""/>  
만약 저 카테고리에서 jekyll를 선택하면 카테고리 리스트가 생기게 되는데 아래 그림과 같다.
<img src="../post_images/카테고리 리스트.png" width="50%"  title="table1" alt=""/>  
난 이 페이지를 얻기 위해서 약 2시간을 태웠다. 😃😃😃  

## navigation
우선은 minimal mistake에서 기본적으로 제공하고 있는 `_data/nagigation.yml`을 확인해야 하는데 default 값으로는 sidebar-category가 없을 것이다. 따라서 이걸 새로 만들어줘야 되는데 아래 코드처럼 만들면 된다.
```
main:
sidebar-category:
  - title: Research
    children:
      - title: "Human"
        url: "/human"
        category: "human"
      - title: "UVI Study"
        url: "/uvi study"
        category: "uvi study"
  - title: Server
    children:
      - title: "Ubuntu"
        url: "/ubuntu"
        category: "ubuntu"
  - title: blog
    children:
      - title: "Jekyll"
        url: "/jekyll"
        category: "jekyll"
```
이렇게 `data/navigation.yml`를 수정하면 각 페이지들을 알맞는 카테고리에 할당해줘야 한다. 또 새로 만들다 보면 `_posts`에 바로 파일을 만드는데 이제 카테고리로 할당해주면 각 카테고리에 맞는 폴더를 제작해주고 관리해주면 편하다.  
예를 들면 `_post/2023-03-22-blog_issue.md`를 `_post/jekyll/2023-03-22-blog_issue.md`에 넣어주면 편하다.  
이렇게 폴더를 생성하고 옮겨준 후에 각 posts의 header 부분을 바꿔줘야 하는데 나 같은 경우
```
---
layout: single
title: "jekyll 우분투 서버에서 돌리는 법 + Issue"
---
```
여기서
```
---
layout: single
title: "jekyll 우분투 서버에서 돌리는 법 + Issue"
author_profile: true
categories:
  - jekyll
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---
```
이렇게 바꿧다. 여기서 toc 같은 경우에는 아래 그림과 같이 오른쪽에 나오는 목차다.  
<img src="../post_images/doc.png" width="50%"  title="table1" alt=""/>  
아무튼 이렇게 바꾸면 되는데 여기서 `_config.yml`을 바꿔야한다.  
여기서
```
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
```
이렇게 바꾸면 된다.
```
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
      sidebar:                  
        nav: "sidebar-category" 
```

## _posts 만들기
이거 때문에 너무 오래 걸렸다...
우선 `_pages`를 만들어야 하는데 만약 없다면 바로 만들어주도록 하자. 폴더를 만들어주면 `category-{카테고리 이름}.md`을 만들어 줘야한다.  
```
| _page
| - category-human.md
| - category-jekyll.md
| - category-ubuntu.md
| - category-uvi study.md
```
이런식으로 만들어주면 된다. 각 파일내 코드는 다음과 같다.
```
---
title: "human"
layout: archive
permalink: /human
author_profile: true
sidebar:
    nav: "sidebar-category"
---
`{`% assign posts = site.categories.human %`}`
`{`% for post in posts %`}` `{`% include archive-single.html type=page.entries_layout %`}` `{`% endfor %`}`
```
추가적으로 2개의 코드 줄도 넣어야 한다.  
`{`, `}` 이것 양쪽 `는 제거해줘야한다.


## 후기
진짜 너무 힘들었다. 😃😃😃
