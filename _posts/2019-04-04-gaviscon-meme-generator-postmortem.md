---
layout: post
title: 개비스콘 짤 생성기 포스트모템
date: 2019-04-04 21:27 +0900
categories: Programming
tags: [Web,Programming,개비스콘]
comments: true
pinned: true
---

![banner-image]({{ site.url }}/assets/images/2019-04-04-gaviscon-meme-generator-postmortem/banner-image.jpg)

사실 이 주제로 포스트를 작성할 계획은 전혀 없었습니다. 개비스콘 짤 생성기는 2017년 11월 처음으로 공개한 오래된 프로젝트입니다. 그런데 며칠 전 매우 많은 사람들이 [개비스콘 짤 생성기](https://rajephon.github.io/gvsc/)를 이용해주셔서 포스트 모템을 작성하면 좋을 것 같다는 생각이 들었습니다. 그래서 이번 주는 얼랭 시리즈의 다음 편이 아니라 `개비스콘 짤 생성기 포스트모템`으로 주제를 정했습니다.

## 탄생

어느 날 친구와 대화하던 도중 제 기분을 짤로 표현하고 싶었습니다. 개비스콘 짤로 기분을 표현하면 재미있고 정말 좋을 것 같았죠. 하지만 제 노트북에는 포토샵과 같은 그래픽 편집 프로그램이 없었습니다. 또 그렇다고 불법으로 설치하고 싶지는 않았습니다.  
그래서 기왕 이렇게 된 김에 _토이 프로젝트로 짤 생성기를 만들어보면 어떨까?_ 하고 생각했습니다. 그리고 바로 개발에 착수했습니다.

## 개발

기왕이면 규모를 크게 키우고 싶지 않았습니다. 어디까지나 토이 프로젝트였고, 단지 나 자신이 쓰기 위해 만들고자 하는 생각도 있어서 야크털[^1]을 깎고싶지 않았습니다.

그래서 플랫폼을 앱이 아닌 비교적 빨리 결과를 볼 수 있는 웹으로 정하고, 호스팅 서비스도 쉽고 **무료**인 `Github Pages`를 활용했습니다. 이미지에 텍스트 합성 작업은 HTML5의 `canvas`를 잘 써먹으면 백엔드 서버 작업 없이도 가능할 것으로 보였습니다.

웹 디자인(이라 말하기도 뭐하지만)은 [Bootstrap](https://getbootstrap.com/)과 [jQuery](https://jquery.com/)로 작업했습니다. [ReactJS](https://reactjs.org/)도 생각해봤지만, 앞서 말한 바와 같이 빠르고 간단히 진행하고 싶었습니다.

![gvsc]({{ site.url }}/assets/images/2019-04-04-gaviscon-meme-generator-postmortem/gvsc.png)

개비스콘 짤방엔 상단에 붉은색 그림자로 _속쓰림 키워드_ 6개, 하단에 푸른색 그림자로 _개운함 키워드_ 1개로 이루어져 있습니다. 이 모양새는 모두 같기에, 복잡하게 갈 필요 없이 키워드의 위치를 하드코딩했습니다.  
그리고 사용자가 텍스트 필드에 키워드를 입력할 때마다 바로 `canvas`의 `strokeText`, `fillText`를 활용해 짤방 위에 그렸습니다.

```javascript
const drawStroked = function (ctx, text, x, y) {
    ctx.strokeStyle = '#222';
    ctx.lineWidth = 4;
    ctx.strokeText(text, x, y);
    ctx.shadowColor = "transparent";
    ctx.fillStyle = 'white';
    ctx.fillText(text, x, y);
}
const fncAddKeyword = function (ctx, txt, pos, color) {
    ctx.shadowColor = color;
    ctx.font = "20pt 'Nanum Gothic'";
    ctx.lineWidth = 3;
    drawStroked(ctx, txt, pos.x, pos.y);
}
$txtBefore.keyup(function(e) {
    const bf = $(this).val();
    const bfSplit = bf.split(',');
    const ctx = mainCanvas.getContext("2d");
    ctx.shadowColor = "transparent";
    ctx.drawImage(beforeImg, 0, 0);
    const length = Math.min(bfSplit.length, beforePositionArr.length);
    for (let idx = 0; idx < length; idx++) {
        fncAddKeyword(ctx, bfSplit[idx], beforePositionArr[idx], "#ff5555");
    }
});
```

사용자가 키워드를 입력할 때마다 Canvas에 짤방 이미지를 다시 깔고, 키워드에 따라 글씨를 적습니다. 속 쓰림 키워드는 오리지널 짤방처럼 붉은색 그림자를, 개운함 키워드는 푸른색 그림자 효과를 줍니다. 글씨를 강조하기 위해 `strokeText`로 색상 그림자 및 테두리 글씨를 한 번 그려주고, `fillText`로 흰색 글씨를 덧그려줍니다.  
`strokeText`와 `fillText`의 차이점은 [stackoverflow 게시물](https://stackoverflow.com/a/25817125/5286905)의 예시에서 쉽게 이해할 수 있을 겁니다.

```javascript
$('#btnDownload').click(function(){
    const href = mainCanvas.toDataURL();
    this.href = href;
    this.download = "gvsc.png";
});
```

`내려받기`는 훨씬 더 쉽습니다. `cavas.toDataURL`로 만든 URL을 `this.href`로 넘겨주고, `this.download`에 파일 이름을 지정합니다. 그러면 끝납니다. `DataURL`은 canvas에 있는 이미지 버퍼를 URL 형태로 만들어줍니다.

![data-url]({{ site.url }}/assets/images/2019-04-04-gaviscon-meme-generator-postmortem/data-url.png)

`내려받기` 버튼을 우 클릭해서 `새로운 탭에 열기`로 보시면 다음처럼 이미지 버퍼를 Base64 인코딩해서 타입과 함께 URL로 넘어온 것을 확인할 수 있습니다. 이 방법은 알아두면 다른 곳에 써먹기 좋습니다. PNG 형태의 이미지 뿐만이 아니라 여러 미디어(음악파일 등)도 가능합니다.

```html
<canvas id="mainCanvas" width="480" height="722"></canvas>
```

`canvas`를 이용해서 생성된 이미지의 실제로 저장되는 사이즈는 태그 속성의 `width`, `height`로 정해집니다. 여기에 저장될 이미지의 사이즈를 지정하고, css로 적절히 상황에 맞게 변하도록 셋업 합니다.

## 잘한점

### 빠른 개발

![149]({{ site.url }}/assets/images/2019-04-04-gaviscon-meme-generator-postmortem/149.png)  
욕심을 부리지 않고 처음 목표했던 바에 맞춰서 빠르게 개발했습니다.  
코드도 CSS 모두 합쳐도 200줄도 안 돼서 처음 목표했던 바를 달성한 게 성공적이었다고 생각합니다.

### 사용하기 쉬움

별다른 설명이 없어도 사용자들이 잘 이용했습니다. 사실 별다른 기능이 없어서 바로 이해한 바도 없진 않지만...  
그래도 placeholder와 샘플 키워드가 입력되어있는 모습을 보고 헤매지 않고 쉽게 따라만들었습니다.

### 모바일 및 해상도 대응 추가

![platform]({{ site.url }}/assets/images/2019-04-04-gaviscon-meme-generator-postmortem/platform.png)  
PC 사용자가 모바일보다 많을 것이라 생각했는데, 모바일 이용자가 PC보다 훨씬 많았습니다. 통계상 방문자의 75%가 모바일 플랫폼이었습니다. 완벽하진 않지만 모바일에서의 레이아웃 작업을 추가했었는데, 좋은 결과를 가져왔다고 생각합니다.

### 인기(!!)

![wikitree]({{ site.url }}/assets/images/2019-04-04-gaviscon-meme-generator-postmortem/wikitree.png)  

예상 이상으로 많은 분들이 이용해줬습니다. SNS에 [개비스콘 짤 생성기](https://twitter.com/search?q=%EA%B0%9C%EB%B9%84%EC%8A%A4%EC%BD%98+%EC%A7%A4+%EC%83%9D%EC%84%B1%EA%B8%B0)라 검색하면 자신의 스타일에 맞게 짤을 생성해서 공유하는 모습을 볼 수 있었습니다.  
따로 홍보를 하지 않아도 사람들이 재미있으면 공유하는 것에서 SNS의 효과를 느낄 수 있었습니다.

## 아쉬운 점

### 새로운 기술 배움이 없다

빠르게 개발하는 것에 초점을 두다 보니 알고 있는 지식만 활용하는게 거의 대부분이었습니다.  
그래서 새로운 배움이 거의 없어 약간의 아쉬움이 남았습니다. 리액트를 써보거나, `Bootstrap` + `jQuery` 말고 다른 기술을 활용해보는 것도 좋지 않았을까 싶었습니다.

### SNS 이미지 공유 기능이 없다

사용자는 짤방을 만들고 내려받을 수는 있지만 바로 SNS(트위터, 페이스북 등)에 이미지를 공유하고 게시물을 작성할 수 없습니다. 대부분의 SNS가 단순 URL 공유는 서버 없이 가능하지만, 트위터 등에서 이미지 업로드를 포함한 공유 링크는 서버에서 API를 활용하여 가능합니다. 개비스콘 짤 생성기는 해당 기능을 수행할 서버가 없어 조금 아쉬움이 남았습니다.  
추후에 섬네일, 파라미터를 통한 이미지 렌더링 등으로 대체수단을 마련해볼까 합니다.

### 약간 아쉬운 퀄리티

조금 더 신경 써주면 좋지 않을까? 하는 부분들이 있습니다. 예를 들어 브라우저 width 740px 가량일 때 canvas 크기가 어색합니다.  
카카오톡 등 여러 인앱 브라우저에서 실행 시 권한 문제 등으로 짤방을 내려받지 못하는 경우가 있습니다. 트위터만 셋업 해뒀는데 여러 인앱 브라우저에서 실행할 경우 내려받기가 안될 수 있다는 안내를 고지하는 작업을 해뒀으면 유저들이 사용하는데 더 편리했을 것이라고 생각합니다.

## 결론

기술적인 새로운 배움과 경험이 있지는 않았지만, 그 이외의 배움과 경험을 준 프로젝트라 생각합니다.  
다음에도 사람들이 즐겁게 즐기고 공유할 수 있는 무언가를 만들어낼 수 있었으면 좋겠습니다.

## 참고 링크

- [개비스콘 짤 생성기](https://rajephon.github.io/gvsc/)
- [위키트리 - 커뮤니티서 재밌다고 난리 난 '개비스콘 짤 생성기'](https://t.co/wmooyilljz)

[^1]: [야크 털 깍기(Yak Shaving)](https://www.lesstif.com/pages/viewpage.action?pageId=29590364)  
