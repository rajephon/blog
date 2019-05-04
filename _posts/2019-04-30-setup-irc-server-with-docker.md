---
layout: post
title: 🐳 도커를 이용해 쉽게 IRC 서버 구축하기
date: 2019-04-30 21:47 +0900
categories: Docker
tags: [Docker,IRC]
comments: true
pinned: true
---

![banner-image]({{ site.url }}/assets/images/2019-04-30-setup-irc-server-with-docker/banner-image.jpg)

Photo by [SpaceX](https://unsplash.com/photos/VBNb52J8Trk?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/satellite?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

오늘은 [Let's Encrypt](https://letsencrypt.org/)와 [Docker](https://www.docker.com/)를 활용해 빠르고 간단히 IRC 서버를 구축해봅시다. IRC는 공개된 명세가 있는 프로토콜이기에 다양한 클라이언트 소프트웨어가 있고, 서버 구현체 또한 [UnrealIRCd](https://www.unrealircd.org/), [Bahamut](https://github.com/DALnet/bahamut), [Inspircd](http://www.inspircd.org/) 등 다양하게 존재합니다.

또한 IRC는 오래된 프로토콜이지만 오늘날 예전만큼 많이 사용되지 않아 dockerhub에 공식 컨테이너가 별로 없었습니다. 그런 와중에 다행히 [Inspircd](http://www.inspircd.org/)는 공식 컨테이너가 등록되어 있고, 최근 업데이트도 되고 있습니다. 오늘 실습에는 [Inspircd]를 이용해 서버를 구축해봅시다!

## Ready

### 💬 IRC가 무엇인지 몰라요

한 단어로 요약을 하면, **채팅 프로토콜**입니다. 자세한 설명은 위키백과 [인터넷 릴레이 챗](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7_%EB%A6%B4%EB%A0%88%EC%9D%B4_%EC%B1%97)을 추천합니다.  
특히 영문판 문서는 매우 상세하게 작성되어 있으므로 많은 궁금증을 해결할 수 있을 것입니다.

### 🛰 Server Setup

IRC 서버를 구축할 환경을 준비합니다. 저는 AWS의 [Lightsail](https://lightsail.aws.amazon.com) 인스턴스를 이용해보겠습니다.

![docker01]({{ site.url }}/assets/images/2019-04-30-setup-irc-server-with-docker/docker01.png)

먼저 도커를 설치해주시고..

방화벽이 있을 경우 **잊지 말고** 포트를 열어줍시다. 기본적으로 6667, 6697, 7000, 7001를 이용합니다. 6667과 6697은 클라이언트와 통신할 때 사용하고, 7000, 7001은 다른 IRC 서버와 링크 등 통신을 할 때 활용합니다. 또한 6667, 7000은 plain text, 6697, 7001은 TLS 통신입니다.

![lightsail_port]({{ site.url }}/assets/images/2019-04-30-setup-irc-server-with-docker/lightsail_port.png)

저는 다른 서버와 연결 없이 단독으로 운영할 예정이기에, 6667, 6697 포트만 열어줬습니다.

### 🔏 Let's encrypt!

그리고 인증서를 준비합시다. 사실 단지 테스트만을 위함이라면 `self-generated certificate`를 사용하셔도 무방합니다([inspircd-docker](https://hub.docker.com/r/inspircd/inspircd-docker)에서 관련 환경 변수를 지원합니다). 하지만 일부 IRC 클라이언트에서는 보안을 위해 보증된 인증서만 허용하므로, Let's encrypt를 통해 인증서를 발급하여 이용해봅시다.

```bash
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
```

인증서 발급 클라이언트 `certbot`을 내려받고, 원활한 동작을 위해 권한을 수정합니다.

```bash
./certbot-auto certonly --standalone -d your.irc.domain.com
Requesting to rerun ./certbot-auto with root privileges...
FATAL: Amazon Linux support is very experimental at present...
if you would like to work on improving it, please ensure you have backups
and then run this script again with the --debug flag!
Alternatively, you can install OS dependencies yourself and run this script
again with --no-bootstrap.
```

만약, Amazon Linux를 이용한다면, 다음과 같은 오류를 마주할 수 있습니다. 오류 문구대로 아직 Amazon Linux 용 동작은 `experimental`라서 사용은 `--debug` 플래그를 추가해야 합니다.

`--debug` 플래그를 추가하여 다시 시도해봅시다. 참고로, **발급 도중에는 웹 서비스를 잠시 내려주셔야 합니다!**

```bash
./certbot-auto certonly --standalone --debug -d your.irc.domain.com
# 주의 : 이미 웹 서비스가 가동 중이라면 잠시 내려주세요.
# ...
# ...
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): # 이메일 입력

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: a # 동의

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: n # 뉴스레터를 받고싶다면 y
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for your.irc.domain.com
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your.irc.domain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your.irc.domain.com/privkey.pem
   Your cert will expire on 2019-07-27. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

성공! 몇 가지 질의응답을 거치면 인증서 발급이 완료됩니다.  
인증서는 `/etc/letsencrypt/live/your.irc.domain.com/` 에 생성됩니다.

자! 이제 모든 준비는 끝났습니다. 이제 `Docker-compose`를 작성해봅시다.

## Docker Compose

### 왜 컨테이너 하나 띄우는데, docker-compose를 작성하나요? docker run 하면 안 되나요

안될 이유 없습니다 :D 실제로 아래의 한 줄 명령어로 IRC 서버를 구동할 수 있습니다.

```bash
docker run --name ircd -p 6667:6667 inspircd/inspircd-docker
```

하지만 우리가 `docker-compose`를 작성하는 이유는 단지 좀 더 쉽고 편하게 원하는 대로 설정하기 위함입니다. 여러 커스터마이징을 하고 싶은데, 명령어 한 줄에 여러 환경 변수와 포트 등의 설정을 작성하기엔 실수가 발생하기 쉬우니까요. 그래서 이 포스트에서는 `docker-compose`를 활용하는 방안을 기반으로 작성했습니다.

자, 그러면 적당한 위치를 지정하고, `docker-compose.yml` 파일을 생성해주세요. 저는 `inspircd-docker`라는 디렉터리를 생성하고 그곳에 작성하겠습니다.  
또한 손쉽게 설정 파일들을 관리하기 위해 컨테이너와 연결할 config 디렉터리를 생성해주세요.

```bash
mkdir inspircd-docker
cd inpircd-docker
touch docker-compose.yml
mkdir config
```

그리고 `docker-compose.yml`를 작성해봅시다.

```yml
version: "3.5"
services:
        irc:
                container_name: irc_server
                image: inspircd/inspircd-docker:2.0.27
                ports:
                        - "6667:6667"
                        - "6697:6697"
                environment:
                        - INSP_NET_SUFFIX=.irc.your.domain.com
                        - INSP_ADMIN_NAME=Admin Name
                        - INSP_ADMIN_NICK=admin
                        - INSP_ADMIN_EMAIL=admin@your.domain.com
                        - INSP_CONNECT_HASH=sha256
                        - INSP_CONNECT_PASSWORD=aaaaaaabbbbbbbbccccccdddddddeeeeefffffggggggghhhhhh
                volumes:
                        - ./config:/inspircd/conf/
```

`irc`라는 이름의 서비스를 만들었습니다. 도커 이미지는 `inspircd/inspircd-docker`의 가장 최근 버전인 `2.0.27`로 지정했습니다. 가급적 `latest` 태그 사용은 자제해주세요. `latest`는 무조건 가장 최신 버전을 따르기 때문에, 추후 버전이 바뀌고 셋업이 맞질 않아 동작하지 않는 문제가 발생할 수 있습니다.

그리고 채팅을 위해 6667, 6697 포트를 연결해주고, 환경 변수를 셋업 합니다. `INSP_NET_SUFFIX`와 어드민 지정 관련 설정, 그리고 저는 제가 아는 사람들하고만 IRC 서버를 이용하길 원하므로, 접속 비밀번호를 아는 사람들만 접근을 허용하기 위해 SHA256으로 해시된 비밀번호를 설정했습니다.  
`INSP_CONNECT_HASH`를 지정하지 않고, `INSP_CONNECT_PASSWORD`에 plaintext를 입력하여 암호를 사용할 수 있습니다. 물론 권장하지 않는 방법입니다.

만약 Let's Encrypt를 이용하지 않고 `self-generated certificate`를 이용한다면 여기서 환경 변수로 `INSP_TLS_CN`, `INSP_TLS_MAIL` 등을 설정할 수 있습니다. 자세한 사항은 [inspircd/inspircd-docker](https://hub.docker.com/r/inspircd/inspircd-docker/)에서 확인할 수 있습니다.

마지막으로 호스트 `./config` 디렉터리를 컨테이너의 `/inspircd/conf/`에 연결합니다. 이후 `inspircd`의 설정 파일들이 호스트로 노출되어 제어가 훨씬 편해집니다.

## 인증서 적용

앞서 생성한 인증서를 활용할 시간입니다. config 디렉터리로 인증서를 복사합니다.  
`fullchain.pem` 은 `config/cert.pem`로, `privkey.pem`는 `config/key.pem`로 복사합니다.

```bash
sudo cp /etc/letsencrypt/live/your.irc.domain.com/fullchain.pem ./config/cert.pem
sudo cp /etc/letsencrypt/live/your.irc.domain.com/privkey.pem ./config/key.pem
```

그리고, 인증서 접근에 권한으로 인한 문제가 발생하지 않도록 config 디렉터리의 owner UID를 `10000`로 설정합니다. [참고](https://github.com/inspircd/inspircd-docker/blob/master/README.md#how-to-use-this-image)

```bash
chown 10000 ./config/ -R
```

## 🚀 docker-compose up!

모든 준비는 끝났습니다. 이제 IRC 서버를 가동해봅시다.  
`docker-compose.yml`를 작성한 디렉터리로 이동한 다음, `docker-compose up` 명령어를 실행합니다.

```bash
$ docker-compose up
Starting irc_server ... done
Attaching to irc_server
irc_server | Inspire Internet Relay Chat Server, compiled on Apr  9 2019 at 04:07:34
irc_server | (C) InspIRCd Development Team.
irc_server |
irc_server | Developers:
irc_server | 	Brain, FrostyCoolSlug, w00t, Om, Special, peavey
irc_server | 	aquanight, psychon, dz, danieldg, jackmcbarn
irc_server | 	Attila
irc_server |
irc_server | Others:			See /INFO Output
irc_server |
irc_server |
irc_server | Loading core commands.Sat May  4 08:27:55 2019: Log started for InspIRCd-2.0.27 (v2.0.27, inspircd_module_200_11) - compiled on Linux 
### .....
### 생략
### .....
irc_server | Sat May  4 08:27:55 2019: New module introduced: m_userip.so (Module version InspIRCd-2.0.27 rv2.0.27)
irc_server | [*] Loading module:	m_spanningtree.so
irc_server | Sat May  4 08:27:55 2019: New module introduced: m_spanningtree.so (Module version InspIRCd-2.0.27 rv2.0.27)
irc_server | Sat May  4 08:27:55 2019: m_ssl_gnutls.so: Enabling SSL for port [::]:6697
irc_server | InspIRCd is now running as '0644f77bfd34.irc.your.domain.com'[973] with 1024 max open sockets
irc_server | Sat May  4 08:27:55 2019: Keeping pseudo-tty open as we are running in the foreground.
irc_server | Sat May  4 08:27:55 2019: Startup complete as '0644f77bfd34.irc.your.domain.com'[973], 1024 max open sockets

```

중간에 오류 문구 없이, `Startup complete as ~ max open sockets`가 나타났다면 성공적으로 구동이 완료된 것입니다! Yeah!  
만약 중간에 에러 로그가 나타났다면 침착히 원인을 파악해봅시다. 꽤나 디테일하게 나와서 원인을 찾기 쉬울 것입니다.

## 📡 Connect to IRC server

IRC 클라이언트에서 우리가 만든 서버로 접속해봅시다. IRC 클라이언트 소프트웨어는 정말 다양하게 존재합니다. 그중에서 저는 [Irssi](https://irssi.org/)를 이용해보겠습니다.

### Install Irssi

[여기](https://irssi.org/download/)서 각 플랫폼 별 설치법을 확인할 수 있습니다. 다양한 패키지 매니저에 등록되어 있으므로 어렵지 않을 것입니다. 정 안되면 직접 내려받고 빌드를 해도 무방합니다.  
저는 macOS 환경이므로 homebrew를 이용했습니다.

```bash
brew install irssi
```

### Connect

설치가 완료되었으면 접속을 해봅시다. 먼저, `irssi`를 실행해주세요.

![irssi01]({{ site.url }}/assets/images/2019-04-30-setup-irc-server-with-docker/irssi01.png)

`/connect` 명령을 이용해 서버에 접속합니다.

```plain
/connect -tls irc.your.domain.com 6697 password
```

저는 패스워드를 셋업 해뒀고, 보안 접속을 이용하길 희망하므로 여러 옵션을 줬습니다.

![irssi02]({{ site.url }}/assets/images/2019-04-30-setup-irc-server-with-docker/irssi02.png)

IRC 서버 접속 성공!  
Docker 로고의 아스키 아트가 인상적입니다. 서버 사이드에서도 접속 로그를 확인할 수 있습니다.

![irssi03]({{ site.url }}/assets/images/2019-04-30-setup-irc-server-with-docker/irssi03.png)

기왕 접속한 김에 채널을 만들고, 간단한 대화를 나눠봤습니다.  
혹시 Irssi 명령어에 대한 설명이 필요한 분들이 있을까 하여 몇 가지를 적어둡니다.

#### Irssi 명령어

- `/nick new-nick` 닉네임 변경
- `/channel #channel-name` 채널 접속
- `/leave` 채널 나가기
- `/help` 도움말
- `/exit` irssi 종료

## 🍎 One more thing

`docker-compose up`으로 도커 서버를 한 번 구동하고 나면 `./config` 디렉터리에 여러 설정 파일이 생성된 것을 보실 수 있습니다.  해당 파일들을 수정하여 서버를 알맞게 다듬을 수 있습니다.  
예를 들어 `docker.motd`는 클라이언트가 서버에 접속 시 보이는 메세지입니다. 이 파일을 수정 후 `docker-compose restart`를 할 경우 반영됩니다.

## 기타

- `docker-compose up` 명령을 한 다음에, 로그 출력 모드에서 나오고 싶으면(detach mode), `Ctrl + z`를 눌러주세요.
- `docker-compose up` 명령을 할 때부터 Detach mode로 하고 싶으면, `-d` 혹은 `--detach` 옵션을 추가해주세요.
- Detach mode에서 다시 로그를 보고 싶으면 `docker-compose logs` 를 활용할 수 있습니다.
  - 로그를 계속 출력하는 상태로 두고 싶을 경우 `--follow` 옵션을 붙여주세요.
- `motd`는 'message of the day(오늘의 메시지)'라는 뜻입니다.
- TLS로만 서비스를 이용하도록 하고 싶을 경우, 방화벽에서 plain text를 이용하는 `6667`포트를 닫아도 무방합니다. 단, 접속을 할 때 꼭 TLS 옵션과 포트를 입력해야 합니다.

## Reference

- [🐙 Quick Start: Docker Compose](https://velog.io/@godori/-Docker-Compose-Quick-Guide)
- [Docker hub - inspircd-docker](https://hub.docker.com/r/inspircd/inspircd-docker)
- [RFC 7194 - Default Port for Internet Relay Chat (IRC) via TLS/SSL](https://tools.ietf.org/html/rfc7194)
- [suchsecurity.com - Using LetsEncrypt for your IRCd](https://suchsecurity.com/using-letsencrypt-for-your-ircd.html)
- [Wikipedia - motd (Unix)](https://en.wikipedia.org/wiki/Motd_(Unix))