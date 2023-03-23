---
layout: single
title: "minimal mistake Utterances 리눅스 서버에서 댓글 생성 안될 때"
---

## Utterances란?
github blog를 만들면서 가장 먼저 블로그 꾸미기가 중요할 것 같아서 댓글 및 카테고리를 만들려고 한다.  
보통 github blog 하면서 댓글하는 걸로 Utterances와 Disqus가 있다고 하는데 대부분 Utterances를 추천하는 듯 하다.  
**그 이유는 Disqus의 경우 너무 무겁고 무료 라이센스를 사용하면 광고가 붙는다 하니 다들 껴러하는 눈치다.**  
또한 github에서 지원하는 app인듯 하여 라이센스를 부여하는 일은 없을 듯 하다?

## 어떻게 적용하는가?
생각보다 간단한데 Jekyll를 사용하면서 Utterances를 사용하면 보통 _layouts에 있는 posts.html에 적용시키는데 minimal mistake는 아무래도 많이 쓰는 추세다 보니까 이미 사용하기 편하도록 구성이 미리 되어있는 듯 하다.  

하지만 minimal mistake를 언젠간 나도 바꿀 날이 올 수도 있으니 두가지 전부 알아보도록 하자

## minimal mistake를 안쓰는 경우

[Utterances](https://github.com/apps/utterances)에 접속을 하면 아래와 같은 github url을 들어가게 된다.  

<img src="../post_images/utterances.png" width="1000px"  title="table1"/>  