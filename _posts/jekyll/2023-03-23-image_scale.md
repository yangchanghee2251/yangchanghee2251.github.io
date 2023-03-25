---
layout: single
title: "minimal mistake 이미지 확대 & 확장 법"
permalink: /blog
author_profile: true
categories:
  - jekyll
toc_label: "목차"
toc: true
toc_sticky: true
sidebar:
    nav: "sidebar-category"
---

## 참고 블로그

[Terry (임연우)`s blog](https://terry1213.github.io/blog/jekyll-magnific-popup/)에서 참고했으며 정리가 잘되어 참고하기 편했다. 좋은 글이니 자주 방문하도록 하자.

## Magnific Popup
블로그를 꾸미면서 작은 이미지가 보기 불편하다는 걸 확인했다. 그래서 이미지 확대 하는 방법을 찾아봤고 위 블로그에서 해결할 수 있었다!  
나는 Minimal mistake에서 진행했으며 다른 theme는 위 블로그를 참조하도록 하자. 우선 블로그 말대로 [Magnific-Popup](https://github.com/dimsemenov/Magnific-Popup)에서 파일을 다운 받아 `dist` 디렉토리의 `jquery.magnific-pop.js` 파일과 `libs/jquery` 디렉토리의 `jquery.js` 파일을 내 블로그 파일 assets/js에 넣었다.

## magnific-popup-setting.js
`assets/js` 디렉토리에 `magnific-popup-setting.js`파일을 만들고 넣는다.
```
$(document).ready(function() {

    // 2.1. id가 magnific인 경우에만 magnific-popup 적용.
    //$('.page__content img[id="magnific"]').wrap( function(){
    $('.page__content img').wrap( function(){
		
        // 2.2. magnific-popup 옵션 설정.
        $(this).magnificPopup({
            type: 'image',
            closeOnContentClick: true,
            showCloseBtn: false,
            items: {
              src: $(this).attr('src')
            },
        });
				
        // 2.3. p 태그 높이를 내부 컨텐츠 높이에 자동으로 맞추기.
        $(this).parent('p').css('overflow', 'auto');
				
        // 2.4. 이미지를 감쌀 태그 설정.
        return '<a href="' + $(this).attr('src') + '" style="width:' + $(this).attr('width') +'px; float: left;"><figure> </figure>' + '<figcaption style="text-align: center;" class="caption">' + $(this).attr('alt') + '</figcaption>' + '</a>';
    });
});
```
[Terry (임연우)`s blog](https://terry1213.github.io/blog/jekyll-magnific-popup/)이 블로그에서 그대로 가져왔고 나 같은 경우에는 모든 이미지에 적용하기 위해서 `$('.page__content img').wrap( function(){` 이렇게 바꿔 주었다.  
그리고 블로그가 하란대로 했으나 이상한 오류가 생겼는데...
```
[2023-03-23 15:15:23] ERROR `/image_scale/assets/js/jquery.js' not found.
[2023-03-23 15:15:23] ERROR `/image_scale/assets/js/jquery.magnific-popup.js' not found.
[2023-03-23 15:15:23] ERROR `/image_scale/assets/js/magnific-popup-setting.js' not found.
```
``/image_scale/assets/js/magnific-popup-setting.js'` 이런식으로 post의 이름이 맨앞으로 가는 🐶 같은 상황이 벌어졌다. 이게 뭔가 생각하다가 jquery등을 만들었던 파일 위치를 못찾는 것이고 맨앞에 있는 이름만 앞으로 땡기면 된다 생각이 들어서 다음 코드를
```
<script src="assets/js/jquery.js"></script>
<script src="assets/js/jquery.magnific-popup.js"></script>
<script src="assets/js/magnific-popup-setting.js"></script>
```
다음과 같이 변경하니 잘된다 ㅎㅎ
```
<script src="../assets/js/jquery.js"></script>
<script src="../assets/js/jquery.magnific-popup.js"></script>
<script src="../assets/js/magnific-popup-setting.js"></script>
```

## 새로운 문제 발생
<img src="../post_images/undifined.png" width="1000px"  title="test1" alt=""/>  
위 그림을 보면 갑자기 밑에 undefined가 생기는 것이 아닌가? 이걸 없애기 위한 작업을 시작해 봤다.  
찾아 보니  
`<img src="../post_images/undifined.png" width="1000px"  title="test1"/>`에서  
`<img src="../post_images/undifined.png" width="1000px"  title="test1" alt=""/>`  
이렇게 추가해주니 사라졌다. 왜인지는 모르겠다.

## 후기
배울게 너무 많다. 빨리 블로그 더 꾸미고 싶다.