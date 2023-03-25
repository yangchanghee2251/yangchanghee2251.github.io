---
layout: single
title: "Github 블로그 naver에 노출시키기"
author_profile: true
categories:
  - jekyll
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---

## naver 노출 시키기
google search consol을 이용해서 google에 노출시키는 방법은 [이전](https://yangchanghee2251.github.io/jekyll/google_consol/) post에서 소개했다. 이번에는 naver를 노출시켜보자!  
1. 우선은 [Naver 서치어드바이저](https://searchadvisor.naver.com/)로 접속을 해야한다.
2. 그 후에 아래 그림과 같이 웹마스터 도구를 들어간다.  

<img src="../post_images/naver_search.png" width="1000px"  title="table1" alt=""/>  

약관에 동의해주도록 하자 ㅎㅎ  
<img src="../post_images/동의.png" width="1000px"  title="table1" alt=""/>  

그 다음 내 blog의 URL를 넣어주도록 하자  

<img src="../post_images/naver_URL.png" width="1000px"  title="table1" alt=""/>  

그 후에 아래 그림에 HTML 파일 업로드에서 html 파일을 받아 준 다음 나의 레포토지에 저장한 후에 `git push`를 진행하자.

<img src="../post_images/HTML_파일_받기.png" width="1000px"  title="table1" alt=""/>  

## Sitemap 등록
우선 등록된 Site를 클릭해서 들어가보자  

<img src="../post_images/요약.png" width="1000px"  title="table1" alt=""/>  

이곳에서 좌측에 요청 => 사이트맵 제출을 들어가서 site map을 제출해주자!
구글 search에서 등록했던 Sitemap을 똑같이 naver에도 적용해주도록 하자. [이전](https://yangchanghee2251.github.io/jekyll/google_consol/)를 보면 참고 가능하다.

<img src="../post_images/naver_sitemap.png" width="1000px"  title="table1" alt=""/>  


## 후기
생각보다 간단했고 만들기 쉬웠다 ㅎㅎ