---
title: "2018 회고"
date: 2019-01-02 18:37:00 +0900
categories: Blog
tags: [2018, 회고록]
layout: post
comments: true

---
<img src="{{ site.url }}/assets/images/2018_remembrance/2019.jpg" alt="prog2019_goalsrammers">  
_이미지 출처: [flickr.com/@Marco Verch](https://www.flickr.com/photos/30478819@N08/31327168808/in/photostream/)_  
원래 Akka 다음 포스트를 이어가야 하는데, 새로운 회사에 적응하면서 정신없이 보내다 보니 2019년이다.
Akka 포스팅 이전에 2018년 한 해를 정리하는 회고를 작성하기로 했다.

## 2018년
### 2월
<iframe width="560" height="315" src="https://www.youtube.com/embed/OPHfehNb3rw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>  

#### 호주 브리즈번에 여행을 다녀왔다.
딱히 업무나 개발과 관련된 여행은 아니고, 힐링 목적이었다.   
첫 느낌은 정말 "덥고 습하다"였다. 한국은 아직도 시베리아에서 내려온 찬 공기가 지배하고 있었고, _심지어 올해는 3월까지 눈이 내렸다!_, 호주는 남반구에 있어 덥고 비가 많이 내리는 시즌이라고 했다. 여행하는 데 불편함을 줄 정도는 아녔다.  

그리고 도시나 사람들의 문화나 전부 기존에 내가 봐왔던 것들과 다른 새로운 느낌이라 인상 깊었다. 사람들은 전부 친절하고 여유로웠다. 버스에 탑승할 때도 기사님은 자리에 앉을 때 까치 출발하지 않고 휠체어 등의 도움의 손길이 필요한 손님이 탑승할 때도 전혀 불만 없이 적극적으로 도와주셨다. 그리고 햇살이 정말 강렬해서 모든 사람이 선글라스를 쓰고 다녔다. 회사원으로 보이는 사람들이 양복 셔츠를 입고 선글라스를 쓰고 다니는 모습을 보며 매트릭스의 요원이 생각났다.
다음에 영어를 능숙하게 익혀서 또 오고 싶다는 생각이 들었다.

### 3월
![OpenWrtLogo]({{ site.url }}/assets/images/2018_remembrance/openwrt_logo.png)
#### OpenWrt
회사에서 서브 프로젝트로 [OpenWrt](https://openwrt.org/)에 [Nodogsplash](https://github.com/nodogsplash/nodogsplash)를 활용해서, 원격에서 `Captive Portal` 인증이나, 웹 푸시를 전송하는 서버를 개발했다.  
프론트부터 백엔드까지 설계하고, 정말 간만에 `C`를 작성하느라 삽질도 많았다. 그래도 WiFi 라우터를 제어하는 것을 만드는 것도 나름 새로운 재미가 있었다.  
그리고 리서치를 하다가 Star가 300개 이상 달린 captive portal 오픈소스 프로젝트에 비트코인 마이너가 심겨 있는 것을 발견해서 한참 웃었던 기억도 난다.  

![MinerInWifiDog]({{ site.url }}/assets/images/2018_remembrance/miner_in_wifidog.png)  
![MinerInWifiDog2]({{ site.url }}/assets/images/2018_remembrance/miner_in_wifidog2.png)

WiFi 라우터에서 해시파워가 나와봤자 얼마나 나온다고 저런 코드를 넣어두다니... 참으로 대단한 발상인 것 같다.  
혹여나 오픈소스를 많이 활용한다면 한 번쯤은 자신이 활용하는 오픈소스 프로젝트들의 코드를 훑어보길 권장한다.

[문제의 오픈소스 프로젝트](https://github.com/liudf0716/apfree_wifidog/blob/b096f0cd051ee08c7074fe1a333a3f4adfcb4ae0/src/gateway.c)의 현재 버전에서는 마이너가 제거된 것 같다. (Star는 100개나 더 받았다.)

### 4월
![MicroDust]({{ site.url }}/assets/images/2018_remembrance/micro_dust.png)  
별다른 특별한 이벤트는 없었고, 미세먼지가 엄청났다.  
그 때문에 기관지가 좋지 않아 몇 번 병원을 다녀왔다.

![teruwa]({{ site.url }}/assets/images/2018_remembrance/teruwa.jpg)

회사 다니며 자주 가는 카페의 명함 이벤트에 당첨됐다. 상품으로 아메리카노 10잔을 받았다!  
첫 명함 이벤트 당첨이라 신기했다.

### 6월
![resignation]({{ site.url }}/assets/images/2018_remembrance/resignation.jpg)  
#### 퇴사를 했다
모든 굴레와 속박을 벗어던지고 행복을 찾아 떠나기로 했다. (feat. 이누야샤 가영이)
#### [YKSConverter](https://github.com/rajephon/YKSConverter) 리팩토링
퇴사 후 개인개발 프로젝트를 정리하며 기존 코드를 다시 열어보니 도저히 봐줄 수 없었다. STL도 활용하지 못하고, C++로 작성했음에도 하나도 OOP답지 않았다. 이런 이유로 언젠가 손봐야지 생각했다. 마침 시간적 여유가 많이 생겨 전부 다 뜯어고쳤다.

### 7월
#### YKSConverter 업데이트
리팩토링한 [YKSConverter](https://github.com/rajephon/YKSConverter)를 [YokosoComposer](https://itunes.apple.com/us/app/%EC%9A%94%EC%BD%94%EC%86%8C-%EC%BB%B4%ED%8F%AC%EC%A0%80-yokoso-composer/id961664211?mt=8)에 적용했다.  
<img src="{{ site.url }}/assets/images/2018_remembrance/programmers.png" width="50%" alt="programmers">  
#### 프로그래머스 연습문제 풀이
그리고 틈날 때마다 [프로그래머스](https://programmers.co.kr/)에서 문제를 풀었다. 백준보다 I/O처리나 컴파일이 편해서 알고리즘 문제 풀이에 집중할 수 있었다.  
풀 때마다 푸른색 체크 표시☑️가 생기는 게 마음에 들어 다 채우고 싶은 욕심이 생겼었다. 풀이는 [여기](https://github.com/rajephon/programmers_training)에 정리했다.  

<img src="{{ site.url }}/assets/images/2018_remembrance/yokoso.png" width="80%" alt="yokoso">  
#### 요코소 개발 시작
이전에 LAMP 스택으로 운영했던 요코소 프로젝트를 공부삼아서 Node.js로 게시판 엔진사용 없이 직접 개발하고 있다. 틈틈이 하는 프로젝트라 개발속도가 더뎠지만, 조만간 테스트 후 문제없으면 오픈할 수 있을 것 같다.  

### 8월
![ow_fest1]({{ site.url }}/assets/images/2018_remembrance/ow_fest1.jpg)  
![ow_fest2]({{ site.url }}/assets/images/2018_remembrance/ow_fest2.jpg)  
#### 오버워치 팬 페스티벌
마침 가까운 곳에서 열렸다. D.Va의 시네마틱 무비와 신규 전장 "부산"도 공개하고, 각종 팬시 상품을 팔고 이벤트를 진행하고 있어서 재미있게 구경할 수 있었다.

### 9월
![ifkakao1]({{ site.url }}/assets/images/2018_remembrance/ifkakao1.jpg)  
![ifkakao2]({{ site.url }}/assets/images/2018_remembrance/ifkakao2.jpg)  
#### "if kakao 개발자 컨퍼런스 2018"에 참관
생각보다 다양한 주제의 세션이 있었다. 내가 모르는 주제라 어려운 내용도 있었지만, 무엇보다 많은 개발자가 참관하고 열정을 가지고 듣는 모습을 보며 동기부여가 되었다.

<img src="{{ site.url }}/assets/images/2018_remembrance/chicgallers.png" width="50%" alt="yokoso">  
#### iOS, 안드로이드 싴갤러스 앱 업데이트
이미지 리소스 변경, 몇 가지 버그를 수정하고 새로운 OS 버전에 대응했다.

### 10월
#### 유니티 멘토링
서울시립대학교 세운캠퍼스에서 세운상가를 소개할 전시작품을 만드는 프로젝트를 진행하는데, 두 명의 학생이 게임 제작을 희망해서 도움을 줄 수 있냐는 연락을 받았다. 때마침 시간상으로 여유가 있고, 알려주면서 나 자신도 유니티를 공부할 좋은 기회라 생각되어 진행하기로 했다.  
다만 개발 경험이 없는 타 전공에, 팀으로 진행하는 메인 프로젝트가 따로 있어서 원하는 만큼 완성도를 내기에는 할애할 수 있는 시간이 많이 부족했다.

![minisewoon]({{ site.url }}/assets/images/2018_remembrance/minisewoon.png)  
![resewoon]({{ site.url }}/assets/images/2018_remembrance/resewoon.jpg)  

그래도 기대 이상으로 잘 배우고 성과를 내줬다. 한 학생은 2D PC게임을 만들고, 한 학생은 3D 모바일 게임을 개발했다. 자세한 내용은 여기서 확인할 수 있다.
- [미니 세운상가 먼 풍경을 지금 눈앞에서](http://openarchive.uosarch.ac.kr/work.html?sem_uid=23&cla_uid=176&pro_uid=374&por_uid=4054)
- [눌러, 눌러! 다시 세운](http://openarchive.uosarch.ac.kr/work.html?sem_uid=23&cla_uid=176&pro_uid=374&por_uid=4053)

#### 일주일에 블로그 1 포스트 작성하기 시작
'전부터 항상 블로그를 작성해야지'라고 생각만 하고 실천을 하지 않아서, 뭔가 강제되는 수단이 있으면 더 성실히 진행하지 않겠냐는 생각에 친구와 같이 사람들을 모아 [프로젝트](https://github.com/post-a-week/blog)를 진행했다. 룰은 간단하다. 일주일에 1개 이상의 블로그 포스팅을 작성하고, 지키지 못할 경우 벌금 5,000원이다. 포스팅 퀄리티는 각자의 양심에 맡긴다.  
효과는 확실히 좋았다. 돈이 걸려있으니 잊지 않고 작성했다. 피치 못할 사정으로 벌금 1회를 내게 됐지만, 그것 말고는 꾸준히 블로그 포스팅을 작성했다 :)

### 11월  
![gstar1]({{ site.url }}/assets/images/2018_remembrance/gstar1.jpg)  
#### 지스타 게임쇼에 참관
새벽에 KTX를 타고 내려가서 지인을 만나 구경을 쭉 하고, 찜질방에서 자고 다음 날 다시 서울로 돌아왔다. 상상 이상으로 사람들이 많았다! 현장 발권이나 예매 발권이나 상관없이 사람들이 정말 많았다.  
다만 벡스코의 수용 가능 인원 대비 너무 많은 인원이 몰렸는지, 행사장 내부에서 인파에 휩쓸리느라 구경이나 체험을 제대로 할 수 없어 아쉬웠다. 특히 인기 있는 부스의 체험 대기 줄은 40~50분을 기다려야 체험이 가능할 정도였다.

![gstar2]({{ site.url }}/assets/images/2018_remembrance/gstar2.jpg)  
![gstar3]({{ site.url }}/assets/images/2018_remembrance/gstar3.jpg)  
부스의 형태가 인상 깊어서 사진을 찍었다. 많은 대기업 부스들이 게임플레이를 하는 모습을 지나가는 관람객이 볼 수 있게 계단 형태로 되어있었다. 

### 구직활동 시작
포트폴리오 정리를 끝내고 구직활동을 본격적으로 시작했다. 지원할 회사를 고르는 주요 기준 세 가지를 정했다.
- 첫째, '나와 회사가 같이 성장할 수 있는가?'
- 둘째, '도전적인 과제를 맡아 진행할 수 있는가?'
- 셋째, '체계적인 개발 문화가 있는가?'

첫째는 왜 중요한지 다들 알 것이다. 둘째는 이전 회사에서 여러 업무를 경험하며 내가 이런 일을 좋아한다는 것을 깨달았기 때문에 선택했다. 선례가 없어 맨땅에 헤딩도 하고 미친 듯이 R&D 해가며 답을 찾았을 때 그 기분은 말로 표현할 수 없다. 새로 입사할 회사에서도 이런 업무를 맡아 진행할 수 있으면 좋겠다고 생각했다. 셋째는 원활한 협업을 위해 꼭 필요하다고 생각했다.

다만 채용공고를 보고서는 첫째만 알 수 있거나 둘째, 셋째의 일부분만 알 수 있다. 둘째, 셋째가 어떤지는 면접 때 면접관님들과 여러 질문을 나누면서 알아보았다.

### 12월
#### 출근
그리고 드디어 새 직장에 입사하고 출근을 시작했다. 여기서는 배워야 할 기술이 정말 많다. 그래도 대부분 흥미롭거나 알아볼 법한 것들이다. 하루빨리 숙달하고 멋진 프로덕트를 만들어내고 싶다.

### 상시
#### 판교 스터디
매주 월요일 퇴근 후 판교역 근처의 카페에 모여서 각자 할 일을 하는 모임이 있다. 판교에 직장이 있는 6명 정도의 사람들로 이루어져 있으며 개인개발을 하거나 공부를 하기도 한다. 퇴사 후 판교에 갈 일이 없어졌을 때도 나 자신을 부지런히 만들기 위해 참여했다 :)  
이제 직장이 더는 판교가 아니라 참여할 수 없어 아쉽다.

#### 디어화이트
지인들과 유니티 공부를 할 겸 게임개발 프로젝트를 하고 있다.  
팀원들 모두 게임개발 경험이 없어 많이 헤맸다. 처음부터 목표를 크게 잡는 것보다 조그마한 서브 프로젝트를 진행하며 올라가는 게 좋겠다고 생각하고 한 걸음씩 나아가고 있다.

## 2019년 목표
### 요코소 프로젝트 오픈
아무래도 사이드로 조금씩 기술들을 공부하며 개발하느라 많이 늦어졌다. 상반기 내에 만족스러운 퀄리티를 뽑아내고 출시하는 것이 목표다.

### 영어
내 영어의 수준이 기초회화 정도밖에 되질 않아 지난번 여행 때 너무 아쉬웠다. 이야기를 더 나눠보고 싶은 상황에도 말이 나오질 않아 주변의 도움을 받을 수밖에 없었다. 올해는 꼭 일상회화를 매끄럽게 대화를 이어갈 수 있도록 공부할 것이다.

집도 이사하고 새로운 회사와 함께 시작하는 2019년이다. 많은 것들을 이룩할 수 있는 한 해로 꾸며나가고 싶다.