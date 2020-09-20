---
layout: post
title: 🏡 2019 회고
date: 2020-01-03 17:07 +0900
categories: Blog
tags: [회고록]
comments: true

---


![banner-image]({{ site.url }}/assets/images/2019-12-30-goodbye-2019/banner-image@2x.jpg){: width="720"}

Photo by [Danielle MacInnes](https://unsplash.com/@dsmacinnes?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/begin-again?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

2019년은 새 직장에서 일을 시작했고 여러 새로운 공부와 일을 겪으며 다사다난하게 지나갔다. 많은 일이 있었던 만큼 2019년을 되돌아보며, 내가 잘했던 것과 잘하지 못한 것들을 다시 생각해보는 시간을 가졌다. 되짚어보는 시간을 갖게 해주는 점에서 회고록 작성은 참 좋은 것 같다. :)

## 01. 새 직장생활

2018년 여름부터 12월까지의 길다면 길고, 짧다면 짧은 6개월간의 백수 시절을 청산하고 다시 직장생활을 시작했다. 새 회사에서는 신규 게임 개발팀에서 서버를 개발한다. 다양한 사람들과 소통하며 기존에 내가 써왔던 것과는 다른 기술 스택을 새롭게 배우고 일을 한다. 많은 도전적인 과제들을 마주하기도 했고, 만족스럽게 회사생활을 하고 있다. 회사에 가장 만족스러웠던 부분은 동료분들이 신기술이나 도전에 두려워하지 않는다. 항상 현재에 만족하지 않고 개선과 발전에 관해 연구하며 신기술 또한 회사에 이익을 줄 수 있다면 과감히 도전한다. 이런 문화로 인해 나 또한 자연스럽게 같이 성장할 수 있었다.

## 02. Erlang 학습

![learn-erlang]({{ site.url }}/assets/images/2019-12-30-goodbye-2019/erlang@2x.png){: width="720"}

[Akka](https://akka.io/)를 공부하며 Akka에 큰 영향을 줬던 얼랭에 대한 호기심이 생겨 [얼랭(Erlang)](https://ko.wikipedia.org/wiki/%EC%96%BC%EB%9E%AD)을 공부했다. 기본 문법 정도 공부한 수준이지만 함수형 언어와 액터 모델이 주는 새로움의 재미는 꽤 컸다. [Mnesia](https://erlang.org/doc/man/mnesia.html)나 MongoDB를 연결해 사용해보기도 하고 [cowboy](https://github.com/ninenines/cowboy)를 이용해 간단히 사용자 회원가입, 로그인 시스템을 갖춘 웹 API를 만들어보기도 했다. 사용자 풀이 작은 탓인지 자료 찾기가 다소 힘들었다. 액터 모델과 함수형에 관해 관심을 계속 가지고 있기 때문에, 다음에는 엘릭서나 러스트에 대해서도 알아보고 직접 써보며 프로젝트를 진행해볼 예정이다.

## 03. GitHub Satellite 2019 in Berlin 참관

![github-satellite]({{ site.url }}/assets/images/2019-12-30-goodbye-2019/github-satellite.jpg){: width="720"}

기회가 닿아 독일의 베를린에서 열린 [Github Satellite](https://githubsatellite.com/) 행사에 참여했다. 혹시나 하는 마음에 신청한 장학 혜택을 받을 수 있다는 연락을 받았다. 깃허브는 행사에서 보안 취약점을 발견할 뿐만 아니라 알아서 수정 후 풀리퀘스트를 올리는 [Dependabot](https://dependabot.com/), 개발자에게 후원할 수 있는 [GitHub Sponsors](https://github.com/sponsors), CI/CD의 [GitHub Actions](https://github.com/features/actions) 등 다양한 기능을 공개 및 발표했다. 그리고 여러 세션이 열렸는데 무엇보다 블랙홀 그림자 촬영에 성공한 [EHT](https://eventhorizontelescope.org/) 팀의 발표가 있었다. 이들은 Numpy, SciPy, Matplotlib, Astropy 등 다양한 오픈소스의 도움을 받은 것을 강조하며 모든 오픈소스 기고자들에게 감사의 인사를 전했다. 세계 여러 개발자를 만날 수 있었고, 넓은 세상과 앞으로 내가 나아가야 할 방향에 대해 생각할 좋은 기회였다.

## 04. yoko.so preview 제작

![aws-diagram]({{ site.url }}/assets/images/2019-12-30-goodbye-2019/aws-diagram@2x.jpg){: width="720"}

요코소([yoko.so](https://yoko.so)) 사이트를 업데이트했다. 요코소는 마비노기 MML 악보를 공유하는 사이트다. 이전에 개발 및 운영하다 문을 닫고, 현재 공부할 겸 다시 개발 중이다. 제작이 늦어져 MML 컨버터를 먼저 개발하고 공개했다. MML 컨버터는 마비노기 MML 악보를 오디오 사운드로 변환해준다. 컨버터 프리뷰 사이트를 제작하며 이전에 알기만 하고 써보지 않았던 Serverless, AWS Lambda, CloudFront, API Gateway, Route 53등을 배우며 써봤다. 정말 엄청난 삽질의 연속이었다. Lambda에서 네이티브 바이너리를 동작시키기 위한 셋업, 250MB 용량 제한 문제, 출력 오디오 포맷 등 모든 것들이 쉽지 않았다. 하지만 고생한 만큼 배포하고 잘 동작하는 모습을 볼 때 정말 뿌듯했다.

관련 작업을 하며 새롭게 배운 게 많아 포스트모템 작성을 생각해봤는데, 요코소를 정식으로 오픈하면 그것과 관련하여 한 번에 작성하는 것이 더 좋아 보여 따로 포스트를 게시하지 않았다.

## 05. yoko.so 사이트 제작

요코소 사이트 제작은 중간에 회사가 바빠지기도 하고, 우선순위에서 밀리며 생각보다 오래 걸리고 있다. 사실 늦어진 가장 큰 이유는 중간에 새로 만들던 레포를 버리고 처음부터 다시 시작했다. 이유는 시간 투자 대비 배울 수 있는 것들이 별로 없다고 느꼈다. 처음 다시 만들기 시작할 때 빨리 만들고 싶은 욕심으로 기술 스택을 잘 알고있고 능숙한 것들(express, mysql, ejs, jquery, bootstrap, 등)로 골랐다. 익숙한 만큼 결과물은 금방 나왔지만, 개발을 진행할수록 더 새로운 배움을 얻고 싶은 욕심이 밀려왔다. 결국 레포를 새로 팠다. 비록 처음부터 다시 만들어야 했지만, 타입스크립트나 리액트, ELK(ElasticSearch, Logstash, Kibana) 등 많은 것들을 배울 수 있었고 지금은 나름 속도가 붙었다. 핵심은 다 만들어 상반기에 오픈할 수 있을 것으로 보인다.

## 06. 개비스콘

![learn-erlang]({{ site.url }}/assets/images/2019-04-04-gaviscon-meme-generator-postmortem/banner-image.jpg){: width="340"}

재미로 만든 [개비스콘 짤 생성기](https://gvsc.rajephon.dev/)를 생각 이상으로 많은 사람이 이용해줬다. SNS나 커뮤니티에서 인기를 끌고, 유사 언론(?)에도 소개가 되었다. 관련하여 [포스트모템](https://blog.rajephon.dev/2019/04/04/gaviscon-meme-generator-postmortem/)을 작성했다.

## 07. 컨퍼런스 참관

[넥슨 개발자 컨퍼런스](https://ndc.nexon.com/main), [Women Techmakers Seoul](https://wtm-seoul-2019.firebaseapp.com/), Google Cloud Advanced Workshop 등 행사에 참관했다.

![ndc]({{ site.url }}/assets/images/2019-12-30-goodbye-2019/ndc.jpg){: width="720"}

NDC2019에서는 김에스더 님의 [〈FIFA 온라인 4〉 서버 포스트모템](http://ndcreplay.nexon.com/NDC2019/sessions/NDC2019_0058.html) 세션이 가장 기억에 남는다. FIFA 온라인 4의 서비스 런칭과 라이브 운영까지의 경험으로 마이크로서비스아키텍쳐와 쿠버네티스의 장단점을 성명해주셨는데, 사례 기반으로 설명해주시다보니 이해하기 쉬웠다.

![wtm]({{ site.url }}/assets/images/2019-12-30-goodbye-2019/wtm.jpg){: width="720"}

우먼 테크 메이커스에서는 에이미장 님의 `원하는 것을 얻는 방법 - 구글 개발자/개발팀장/프로덕트 매니저를 거쳐온 커리어 이야기` 세션이 인상 깊었다. 개발과 관련된 이야기가 아닌 나를 성장시키고 커리어를 계획하는 방법, 원하는 것을 얻는 커뮤니케이션 방법, 매니저와의 효과적인 관계설정 방법에 관련된 이야기다. 나에 대해 다시 되돌아보고 목표와 방향을 설정하고 성장하는 방법에 대해 배울 수 있는 시간이었다.

## 정리

2019년은 2018년보다 훨씬 더 많은 성장을 했다. 2018년에는 퇴사를 하고 나 자신을 되돌아보고 앞으로 나아갈 방향성을 정하는 시간을 가졌다면, 2019년에는 본격적으로 발전에 속도를 붙이는 한 해였다. 말로만 듣고 직접 써보지 않았던 React나 Serverless, AWS Lambda, Erlang, ELK 등 많은 것들을 직접 사용해보고 익혔다. 설계를 바라보는 시점도 고가용성과 대용량 트래픽 처리의 관점으로 더 넓게 생각하게 되었다. 회사에서 다양한 직군의 동료들과 일을 함께하며 '어떻게 하면 상대방에게 더 좋은 청자가 될 수 있는지, 내 의견과 목적을 잘 전달하고 설득할 수 있을지'와 같이 커뮤니케이션에 대해서도 더욱더 깊게 생각하게 되었다.

## 2020 목표

![kubernetes]({{ site.url }}/assets/images/2019-12-30-goodbye-2019/kube-logo@2x.png){: width="320"}

- 쿠버네티스, 쿠버네티스, 쿠버네티스!
  - 쿠버네티스를 능숙하게 사용하고 싶다. 마이크로서비스아키텍처가 널리 퍼지며 컨테이너 오케스트레이션 플랫폼으로 쿠버네티스는 사실상 표준과 같이 자리 잡고 있다. Kubernetes, Prometheus, Helm, Istio... 단순 이름만 아는 게 아니라 각각의 역할과 조작을 익히고 필요한 시기, 알맞은 곳에 사용할 수 있게 배울 것이다.
- 영어
  - 귀는 어느 정도 뚫린 것 같지만 아직 한참 부족하다. 작문이나 말문을 열려고 하면 막힌다. 2019년 영어 수업을 들을 때 더 열심히 참여하고 노력하지 않은 나 자신에게 아쉬움이 남는다. 올해는 좀 더 내가 하고 싶은 말을 자연스럽게 할 수 있도록 노력할 것이다.
- 대외활동
  - 2019년에는 회사에 적응하고 기술을 익히느라 대외활동을 많이 하지 못했는데, 2020년에는 많은 대외활동을 하며 다양한 사람들을 만나보고 싶다. 여러 경험을 쌓으며 견문을 넓히고 보는 눈을 키우고 싶다.

2020년에도 다방면으로 갈고 닦으며 더 큰 사람이 되고 싶다. 좀 더 시야를 넓히고, 나의 스킬이 내 발목을 붙잡지 않게 준비하며 필요한 곳에 적절한 기술을 선택할 수 있는 사람이 되고 싶다. 올 한 해도 열심히 달려봐야겠다.
