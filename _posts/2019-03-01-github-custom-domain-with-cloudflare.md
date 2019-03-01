---
title: "깃허브 페이지에 커스텀 도메인 연결하기 🛰"
date: 2019-03-01 15:34:00 +0900
categories: Programming
tags: [Github, Domain, Cloudflare]
layout: post
comments: true
pinned: true
---
![banner-image]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/banner-image.png)

## Intro

[Google Domains](https://domains.google)에서 지난 밤 새벽 1시부터 `.dev` TLD를 등록할 수 있게 되었습니다! 🎉

이참에 저도 `rajephon.dev`를 구입해서 등록하고, 서브 도메인으로 [https://blog.rajephon.dev](https://blog.rajephon.dev)를 깃허브 정적 블로그에 연결했습니다.  
이 작업 과정을 포스팅으로 남겨봅니다.

## 도메인 등록

먼저 사용할 도메인을 등록해주세요. [Google Domains](https://domains.google), [gandi.net](https://www.gandi.net/en), [namecheap](https://namecheap.com), 혹은 [호스팅케이알](https://www.hosting.kr/), [후이즈](http://whois.co.kr/), [가비아](https://domain.gabia.com/)같은 국내 업체도 좋습니다. 개인적으로는 작은 기업보다 신뢰할만한 큰 기업에서 구매하는 것을 선호합니다.

## 네임서버 연결

다음으로 도메인 레코드를 관리할 서비스로 DNS 네임서버를 연결합니다. 대부분의 도메인 등록 대행업체에서 해당 서비스를 제공해주고있습니다. 해당 서비스를 이용하거나, [클라우드플레어(Cloudflare)](https://www.cloudflare.com), [DNSZi](https://dnszi.com/) 같은 별도의 서비스를 이용하셔도 됩니다. 여기서는 클라우드플레어를 기준으로 설명합니다.

![00]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/00.png)

클라우드플레어에서 로그인 후 `+ Add a Site`를 눌러 새 도메인을 등록합니다. 아직 사이트가 없더라도 괜찮습니다. 기존 DNS 레코드 정보가 있으면 자동으로 이전된다는 이야기지만 저희는 신규등록이라 정보가 없으므로 해당되지 않습니다. 도메인을 입력 후 `Add Site`를 눌러주세요.

![01]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/01.png)

다음으로 Plan 선택지가 나타납니다. 클라우드플레어는 단순 DNS서비스뿐만아니라 CDN, SSL 인증서 서비스, DDoS 방어, 방화벽, 모바일 이미지 최적화 등 다양한 서비스를 제공해주고 있습니다. 다만 저희는 단순 DNS만 이용하면 되므로 Free 플랜을 선택합니다.

![02]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/02.png)

클라우드플레어 네임서버를 등록할 차례입니다. `To`에 나타난 네임서버 주소로 옮겨 등록합니다. 다시 도메인 등록 대행 업체 사이트로 들어가서 네임서버 주소를 바꿔줍니다. (스크린샷은 `rajephon.dev` 등록할 때 스크린샷을 못찍어 다른 도메인으로 찍었습니다.)  

![03]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/03.png)

저는 구글 도메인에서 구입했으므로 구글 도메인 관리자 페이지로 접속해서 네임서버를 바꿔줬습니다. ISP의 캐싱 DNS 서버의 갱신 주기, TTL(Time To Live) 설정값 등의 문제로 DNS새로 갱신된 정보가 전파되기 까지 빠르면 10분 이내, 최대 24시간이 걸릴 수 있습니다.

## A 레코드 등록

![04]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/04.png)

클라우드플레어에서 확인 큐에 등록하고 갱신이 확인되면 이메일이 옵니다. 그러면 이제 깃허브 서버의 IP를 사용할 주소의 A레코드로 등록합니다. 클라우드플레어 기준으로 `DNS`메뉴에 들어가서 `Name`에 사용할 서브도메인을 적고(루트 도메인 그대로 사용하고 싶으면 `@`를 입력합니다), `IPv4 Address`에 깃허브 IP주소를 입력합니다. 그리고 `Add Record`를 클릭합니다.  
깃허브의 IP 주소는 [Setting up an apex domain](https://help.github.com/en/articles/setting-up-an-apex-domain#configuring-a-records-with-your-dns-provider)에서 알 수 있습니다.

## Github Pages 레포 세팅

![05]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/05.png)

A 레코드 등록 작업이 완료되면 정적 페이지를 관리하고 있는 깃허브 레포의 설정으로 들어갑니다. `GitHub Pages`의 `Custom domain`에 사용할 도메인을 입력후 `Save`를 클릭합니다.

그리고 사용하는 정적 블로그 엔진에 따라 호스트주소를 수정해주면 끝납니다. Yeah!

![06]({{ site.url }}/assets/images/2019-03-01-github-custom-domain-with-cloudflare/06.png)

## 관련링크

- [Google Domains](https://domains.google)
- [Cloudflare](https://www.cloudflare.com)
- [GitHub: Setting up an apex domain](https://help.github.com/en/articles/setting-up-an-apex-domain#configuring-a-records-with-your-dns-provider)