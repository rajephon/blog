---
layout: post
title: 달고나 짤 생성기를 만들어보자
date: 2021-11-02 01:50 +0900
categories: Blog
tags: [포스트모템]
comments: true
---

![banner-image]({{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/banner-image.png){: width="720"}  

넷플리스의 '오징어 게임'이 글로벌하게 흥행을 이끌었다. 세계 각지의 나라에서 드라마 리뷰, 연출 분석, 짤방 등 여러 오징어 게임 관련 콘텐츠가 쏟아져나왔다. 나 또한 오징어 게임에 나온 달고나 짤방이 퍼져나가는 모습을 보고 [짤방 생성기(https://dalgona.games/)](https://dalgona.games/)를 만들어봤다. :) 

기술 스택은 [React](https://ko.reactjs.org/), [TypeScript](https://www.typescriptlang.org/)에 [mui](https://mui.com/)로 선정했다. 지난 [개비스콘 짤 생성기](https://gvsc.rajephon.dev/)는 jQuery를 사용했으므로 이번에는 리액트를 선택했다. 합성은 Canavs를 사용해 백엔드없이 끝내려한다.

이를 위해 두 가지 확인이 필요했다. 하나는 캔버스에서 이미지 마스킹 기능 사용 가능 여부, 또 하나는 사진에서 윤곽선만 뽑아올 방법이다. 달고나 이미지를 배경에 깔고, 달고나 부분으로 마스킹된 영역에 윤곽선을 딴 이미지를 올려주는 방식으로 구현할 수 있지 않을까 싶다.

### 마스킹

사용자가 올린 이미지를 달고나 영역에만 표시하기 위해서는 마스킹 기능을 사용할 수 있어야 한다. 구글링 결과 [스택 오버플로우에 올라온 방법](https://stackoverflow.com/questions/18379818/canvas-image-masking-overlapping)은 생각보다 간단했다.

```javascript
/// draw the shape we want to use for clipping
ctx.drawImage(imgClip, 0, 0);

/// change composite mode to use that shape
ctx.globalCompositeOperation = 'source-in';

/// draw the image to be clipped
ctx.drawImage(img, 0, 0);
```

먼저 투명한 배경의 마스킹 영역 이미지(imgClip)를 그리고, `globalCompositeOperation` 를 'source-in'으로 바꾸면 기존에 그려진 이미지 영역 안으로만 그려진다.

![mask-image]({{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/mask.png){: width="720"}  

그러므로 배경이 투명하고, 달고나 영역만 잘 따서 표시한 마스킹 이미지를 준비하고,

![dalgona-meme-image]({{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/background.png){: width="720"}  

달고나 속에 그려진 도형을 지운 배경 이미지도 같이 준비한다.

그리고, 이미지를 업로드하면 달고나 속에 그려주도록 만들면...

![cat_in_dalgona]({{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/masking_result.png){: width="720"}  
고양이 사진 출처: [디시 인사이드 야구 갤러리](https://gall.dcinside.com/board/view/?id=baseball_new10&no=457494)

다음과 같이 달고나 속에 고양이를 넣을 수 있다.

1차 고민은 해결!

### 윤곽선

다음으로 이미지에서 윤곽선을 뽑아야 한다. 윤곽선 검출은 [canny edge detector](https://en.wikipedia.org/wiki/Canny_edge_detector)를 자바스크립트로 구현한 [라이브러리](https://github.com/petarjs/js-canny-edge-detector)를 발견했으나, 원하는 처리속 도가 나오지 않았다.
Loading spinner라도 달아야하나 싶었는데, [opencv-js](https://github.com/TechStark/opencv-js)를 발견했고 처리가 훨씬 빨랐다! yeah!


자 이제 적용해 보자. 이미지 파일을 선택하면 버퍼를 가져와서 윤곽선을 따고, 마스킹을 적용해서 달고나 속에 그려주도록 적용했다. 그리고 색상을 반전시키고 선을 달고나 색상으로 바꿔준다.

<video width="729" controls="controls">
  <source src="{{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/process.mp4" type="video/mp4">
</video>  

그리고 위치를 조절할 수 있도록 드래그 기능, 크기 조절, 등을 넣어주고 png 파일로 출력할 수 있는 다운로드 버튼을 추가한다.


### 다운로드 버튼

```typescript
const downloadImage = () => {
    const backgroundCanvas = canvasBackgroundRef.current!;
    const pictureCanvas = canvasPictureRef.current!;

    const downloadCanvas = canvasDownloadRef.current!;
    const downloadContext = downloadCanvas.getContext('2d')!;

    downloadContext.drawImage(backgroundCanvas, 0, 0);
    downloadContext.drawImage(pictureCanvas, 0, 0);

    const newResultImageState:ResultImageState = {
        dataUrl: downloadCanvas.toDataURL()
    };
    dispatch(changeResultImageState(newResultImageState));

    const newDalgonaState:DalgonaState = {...dalgonaState};
    newDalgonaState.generate = false;
    dispatch(changeDalgonaState(newDalgonaState));
}
```

다운로드 버튼을 누르면, 배경 이미지(`backgroundCanvas`)와 표시되고 있는 유저 이미지(`pictureCanvas`)를 가져와서 `downloadCanvas` 에 합쳐 그린다. 그리고 DataURL을 뽑아내어 img 파일로 화면에 그린다.

사용자가 바로 다운로드할 수 있도록 `download` 속성을 가진 a 태그를 만들어보기도 하고, DataURL로 바로 페이지를 이동시켜보기도 했는데, 브라우저 호환 문제가 발생하거나 하는 등 생각보다 만족스러운 동작을 보여주질 않았다. 이에 타협안으로 img 태그를 이용해 화면 하단에 출력하도록 했다.

### 퍼블리싱

![google-domain]({{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/google-domain.png)

기왕 만드는 김에 제대로 해보고 싶어서 [구글 도메인](https://domains.google.com/)에서 도메인도 구입했다. 백엔드 처리가 없으니 [GitHub pages](https://pages.github.com/)로 가볍게 호스팅 한다.

![cloudflare]({{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/cloudflare.png)

도메인 네임서버를 [클라우드플레어](https://cloudflare.com/)으로 설정하고, 클라우드플레어에서 [깃허브 도큐먼트](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)에 나온 대로 A레코드를 설정한다.

DNS 적용을 기다리는 동안 프로젝트를 빌드 및 배포한다. [gh-pages](https://github.com/tschaub/gh-pages)를 사용하면 편하다. `package.json`에 스크립트로 추가하거나 [GitHub Action](https://github.com/tschaub/gh-pages#deploying-with-github-actions)으로 사용 가능하다.  
CNAME 파일을 생성하고 도메인 주소를 등록하는 것도 잊지 말자.

![result]({{ site.url }}/assets/images/2021-10-26-dalgona-postmortem/result.png)

짜잔!! 🎉

약간 아쉬운 점은 조금씩 작업하다 보니 오징어 게임의 인기가 절정일 때 공개하지 못했다. 또 여기에 SNS 공유 기능을 넣으면 좋을 것 같은데, 이미지 파일을 포함한 공유 기능은 백엔드 작업이 필요하여 공개할 때 포함하지 못한 것이 아쉽다.  

그래도 나름 재미있게 만들어져 만족스럽다. 😝

## 레퍼런스

- [Dalgona meme generator](https://dalgona.games/)
- [Canvas image masking / overlapping - stackoverflow](https://stackoverflow.com/questions/18379818/canvas-image-masking-overlapping)
- [[React] 프로젝트를 Github pages로 배포하기 - donggoolosori](https://donggoolosori.github.io/2020/11/26/ghpage/)  
