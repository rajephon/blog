---
layout: post
title: 🏡 라즈베리파이 3 B+를 이용해 홈 서버 구축하기
date: 2019-07-12 19:01 +0900
categories: [Docker, RaspberryPi, Server]
tags: [RaspberryPi, Docker, Ubuntu]
comments: true
pinned: true
---

![banner-image]({{ site.url }}/assets/images/2019-07-12-setup-raspberrypi-home-server/banner-image.jpg)

Photo by [Harrison Broadbent](https://unsplash.com/@hbtography?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/raspberry-pi?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

저는 [AWS Lightsail](https://aws.amazon.com/ko/lightsail/)로 개인 서버를 이용하고 있습니다. 간단한 웹 서비스나 여러 가지 테스트를 할 때 사용하는데요, 가장 작은 인스턴스를 이용해도 매달 약 4.3달러가량의 금액이 청구됩니다. EC2의 `t2.micro`가 가장 작은 인스턴스였던 때에 비하면 절반 가격으로 저렴하지만(2019-07-12 서울 리전 기준 약 10달러/월), 그래도 별달리 사용하지 않을 때에도 지속적으로 청구되는 게 아깝게만 느껴집니다.  
때 마침, 책상 옆 박스에 담져겨있는 라즈베리파이가 눈에 들어왔습니다. 호기심에 구입했던 라즈베리파이는 3 모델 B+로 1.4GHz 쿼드코어, 1GB RAM로 홈 서버를 구동하기엔 충분한 성능을 가지고 있습니다. (심지어 단순 비교만 놓고 보면 내가 사용하던 1코어, 512MB RAM의 Lightsail 인스턴스보다 사양이 좋습니다). 나는 고민할 필요도 없이 바로 홈서버 구축 작업에 들어갔습니다.

## 서버 구축 전 확인 필요 사항

### 외부에서 홈 서버의 80, 443 포트로 접근이 가능한가

80, 443 포트를 이용한 웹 서비스를 할 경우 반드시 확인해야 하는 사항입니다. 가정용 공유기를 사용하여 포트가 막히는 경우에 대한 해결 및 셋업은 따로 적지 않겠습니다. 이 글을 읽으시는 분들께선 포트 포워딩, DMZ을 잘 하시리라 믿습니다.  
몇몇 ISP(Internet service provider; 인터넷 서비스 제공자)는 가정용 인터넷에서 서버 운영을 막기 위함인지 80, 443 등 알려진 포트를 막고 있습니다. 서버를 다 구축한 다음에서야 접속이 안된다는 것을 깨닫고 좌절하지 않도록 포트가 열려있는지 작업 전 미리 확인해봅시다.

방법은 간단합니다. 컴퓨터에 간단한 웹 서버를 띄우고, 외부망(예: LTE로 연결된 스마트폰)으로 서버에 접속을 시도합니다. 웹 페이지가 잘 로드되면 포트가 열려있는 것입니다.

1. 인터넷 모뎀(겸 공유기)에 컴퓨터를 직접 연결하여 공인 IP를 할당받습니다.
2. 간단한 웹서버를 돌려봅시다. Python http.server 라이브러리를 이용하여 "Hello"라고 적힌 `index.html`을 호스팅 해봅시다.

```console
$ mkdir www; cd www
$ touch index.html
$ echo Hello > index.html
$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

이제 스마트폰 등을 이용해 외부망에서 접속을 시도해봅니다. 접속에 성공하면 별문제 없이 80번 포트를 이용해 서비스를 할 수 있는 겁니다.

### IP가 지속적으로 바뀔 수 있다

가정용 인터넷은 DHCP로 할당받기에 지속적으로 IP가 바뀔 수 있음을 인지해야 합니다. IP가 바뀌면 DNS 레코드도 새로운 IP에 알맞게 변경될 수 있도록 [Dynamic DNS](https://en.wikipedia.org/wiki/Dynamic_DNS) 등의 셋업이 필요합니다.

### micro SD

micro SD를 옛날 버전을 사용한다면 IO가 다소 느릴 수 있습니다. 기왕이면 읽기/쓰기 속도를 보고 빠른 모델로 구매 및 사용을 권장합니다. 읽기/쓰기 속도가 느릴 경우 사용하시며 답답함을 느낄 수 있습니다.

## 라즈베리파이 셋업

- 모든 셋업 작업은 `macOS` 위에서 이루어졌습니다. 명령어 또한 `macOS`의 명령어가 포함되어 있습니다.

### OS 선택 및 내려받기

오래 방치된 라즈베리파이에 새 OS를 올려줍시다. [라즈베리파이 공식 홈페이지](https://www.raspberrypi.org/downloads/)에는 `NOOBS`, `Raspbian` 등 다양한 이미지가 있습니다. 저는 서버로 사용할 목적이라 `Ubuntu Server`를 골랐습니다. 우분투는 익숙하고 사용하기도 쉬워 제가 좋아하는 OS입니다.

### SD Card 포맷

내려받기를 누르고, 라즈베리파이의 기본 스토리지로 이용될 micro SD카드를 컴퓨터에 연결하여 FAT32로 포맷합니다.
먼저 `diskutil list`를 이용해 Device Node를 확인합니다. Device Node는 드라이브를 밀고 OS 이미지 기록할 때 사용합니다.

```console
$ diskutil list
/dev/disk0 (internal):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                         251.0 GB   disk0
   1:                        EFI EFI                     314.6 MB   disk0s1
   2:                 Apple_APFS Container disk1         250.7 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +250.7 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD            119.4 GB   disk1s1
   2:                APFS Volume Preboot                 43.9 MB    disk1s2
   3:                APFS Volume Recovery                509.8 MB   disk1s3
   4:                APFS Volume VM                      6.4 GB     disk1s4

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *127.9 GB   disk2
   1:               Windows_NTFS                         127.8 GB   disk2s1
```

용량, 이름 등으로 micro SD카드의 Device Node를 찾을 수 있습니다. 여기서 제 micro SD카드는 `/dev/disk2`입니다.
그리고 `diskutil partitionDisk` 명령어를 이용해 micro SD카드를 FAT32로 포맷합니다. 사용법은 다음과 같습니다.

```plain
diskutil partitionDisk [DeviceNode] [파티션 갯수] [파티션 테이블 종류] [파티션0포맷 파티션0사이즈 파티션1포맷 파티션1 사이즈 ...]
```

일단 [MBR](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%8A%A4%ED%84%B0_%EB%B6%80%ED%8A%B8_%EB%A0%88%EC%BD%94%EB%93%9C) FAT32 단일 파티션으로 포맷할 예정이므로 다음과 같이 명령어를 입력합니다.

__실수로 다른 디스크를 포맷하여 되돌릴 수 없는 사고가 발생하지 않도록 신중히 작업합니다.__

```sh
$ # 파티션 사이즈를 0b로 하면 사용 가능한 모든 영역을 할당합니다.
$ diskutil partitionDisk /dev/disk2 1 MBR "MS-DOS FAT32" ROOT 0b
Started partitioning on disk2
Unmounting disk
Creating the partition map
Waiting for partitions to activate
Formatting disk2s1 as MS-DOS (FAT32) with name ROOT
512 bytes per physical sector
/dev/rdisk2s1: 249674176 sectors in 3901159 FAT32 clusters (32768 bytes/cluster)
bps=512 spc=64 res=32 nft=2 mid=0xf8 spt=32 hds=255 hid=2048 drv=0x80 bsec=249735168 bspf=30478 rdcl=2 infs=1 bkbs=6
Mounting disk
Finished partitioning on disk2
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *127.9 GB   disk2
   1:                 DOS_FAT_32 ROOT                    127.9 GB   disk2s1
```

이전에 내려받은 OS 이미지 파일은 `-.img.xz` 형식으로 압축되어있습니다. 압축을 풀어 `-.img` 형태로 만듭니다. 그리고 디스크를 언마운트한 다음 디스크에 OS 이미지를 기록합니다.

```console
$ diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
sudo dd bs=1m if=/path/to/image/ubuntu-18.04.2-preinstalled-server-arm64+raspi3.img of=/dev/disk2
```

혹시 이 과정에서, micro SD카드가 64GB보다 커 FAT32 포맷에 문제가 있다면 윈도 컴퓨터에서 `Win32 Disk Imager`를 사용해보는 것을 권장합니다. [가이드는 여기에 있습니다](https://ubuntu.com/download/iot/installation-media#windows).

정상적으로 완료되면 이미지 준비는 끝났습니다.

### OS 셋업

기록이 완료되면 SD카드와 키보드, 마우스, 모니터, 랜선 등을 라즈베리파이에 연결한 다음 가동합니다. 작업 도중 저전압 경고가 계속 나타날 것입니다. 모든 IO장치를 전부 사용하여 그런 것이므로 무시합니다.  

초기 어카운트와 패스워드는 `ubuntu`/`ubuntu` 입니다. 가장 먼저 할 일은 역시 계정 이름과 패스워드를 변경합니다. `root` 계정으로 로그인한 다음 계정 이름 전환 작업 후 루트 계정을 잠가줍니다.

```sh
sudo passwd root
exit
# root 계정으로 다시 로그인합니다.
usermod -l rajephon -d /home/rajephon -m ubuntu
groupmod -n rajephon ubuntu
passwd -l root
exit
```

새로 바뀐 계정 이름으로 로그인 후 패스워드를 바꿔줍니다.

```sh
passwd rajephon
```

이제 패키지들을 최신으로 업데이트하고...

```sh
sudo apt update
sudo apt dist-upgrade
```

시간대도 우리가 사는 지역에 알맞게 설정합시다.

```sh
$ timedatectl set-timezone 'Asia/Seoul'
$ date
Sun Jul  7 21:10:37 KST 2019
```

원격에서 작업하기 편하도록 ssh를 설치합니다.

```sh
sudo apt install openssh-server openssh-client
```

설치가 완료되면 ssh 접속이 잘 되는지 확인합니다. ip는 `ip a` 명령어를 통해 손쉽게 확인이 가능합니다.

### 도커 설치

도커는 진리입니다. 언제 어디에서나 유익하죠. 라즈베리파이에서도 마찬가지입니다. 도커를 설치합시다.

```sh
# WARNING: Always examine scripts downloaded from the internet before running them locally.
curl -fsSL get.docker.com -o get-docker.sh
sudo bash get-docker.sh
```

유저 계정에서도 이용할 수 있도록 그룹 추가도 잊지 맙시다.

```sh
sudo usermod -aG docker rajephon
```

### DDNS 셋업

홈 서버에서 동적으로 바뀌는 IP에 대응하기 위해 필수적인 작업입니다. `google domain`, `cloudflare`, `dnszi`, `dnsever` 등 대부분의 DNS 서비스는 모두 A 레코드를 바꿀 수 있는 API를 제공합니다. 저는 Cloudflare를 사용 중이므로 Cloudflare를 기준으로 작성합니다.

Cloudflare API에 맞춰 동작하는 클라이언트로 [ddclient](https://sourceforge.net/p/ddclient/wiki/Home/)가 있습니다.  
여기서 약간 번거로운 작업이 있는데, `apt` 레포에 등록되어있는 가장 최신 버전은 3.8.x 지만 cloudflare에 맞춰 사용하려면 3.9.0 보다 높아야 합니다.  
이러한 문제로 직접 다운로드해서 설치해야 하지만, perl로 작성되어 별도의 빌드 과정이 없어도 되므로 어렵지 않습니다.

```sh
# 동작에 필요한 perl 라이브러리를 설치합니다.
sudo apt install libjson-any-perl libdata-validate-ip-perl libio-socket-ssl-perl
# 최신 ddclient를 내려받고,
wget https://sourceforge.net/projects/ddclient/files/ddclient/ddclient-3.9.0/ddclient-3.9.0.tar.gz
# 압축을 풀어줍니다.
tar -xvzf ddclient-3.9.0.tar.gz
cd ddclient-3.9.0/
# /sbin 로 ddclient를 옮겨줍니다.
sudo cp ./ddclient /sbin/ddclient
# 캐시 디렉토리와 기본 설정 파일을 만들어주고, 권한 조절도 해둡니다.
sudo mkdir /var/cache/ddclient
sudo chown rajephon /var/cache/ddclient
sudo mkdir /etc/ddclient
sudo touch /etc/ddclient/ddclient.conf;
sudo chown rajephon /etc/ddclient/ddclient.conf
```

이제 ddclient 설정 파일을 작성합니다.

```plain
# Configuration file for ddclient
#
# /etc/ddclient/ddclient.conf

ssl=yes
use=web
protocol=cloudflare
server=api.cloudflare.com/client/v4
login=[Cloudflare 이메일 주소]
password=[Cloudflare APIKEY]
zone=domain.com
domain.com, my.domain.com
```

API Key는 Cloudflare의 `My Profile` 항목에서 찾을 수 있습니다.
`zone`에는 내 도메인 이름을 적고, 마로 밑 줄에 A 레코드 IP를 변경할 서브 도메인들을 콤마(`,`)로 구분하여 나열합니다.

이제 정상적으로 작동하는지 테스트를 해봅시다.

```console
$ sudo /sbin/ddclient --force
[sudo] password for rajephon:
SUCCESS:  domain.com -- Updated Successfully to 111.222.333.444
SUCCESS:  my.domain.com -- Updated Successfully to 111.222.333.444
```

cloudflare console에 들어가 보면 IP 주소가 업데이트된 것을 확인할 수 있습니다.
정상 동작도 확인했으니 이제 주기적으로 IP 변경 확인 및 업데이트를 할 수 있게 만들어야합니다. ddclient를 시작 프로그램 리스트에 걸어두고, ddclient의 daemon 기능을 활용하는 방법, cron에 등록해 사용하는 방법이 있습니다. 저는 cron으로 해보겠습니다.  
`sudo crontab -e` 로 크론탭 편집모드로 들어가 아래에 다음과 같이 추가합니다.

```sh
# 2분마다 IP 업데이트 확인 및 갱신
*/2 * * * * /sbin/ddclient
```

좀 더 많은 로그를 남기고 싶을 경우, `-noquiet -query -verbose -debug` 등의 옵션을 추가해줄 수 있습니다.

- {no}quiet : print {no} messages for unnecessary updates (default: noquiet).
- {no}query : print {no} ip addresses and exit.
- {no}verbose : print {no} verbose information (default: noverbose).
- {no}debug : print {no} debugging information (default: nodebug).

cron이 잘 돌고 있는 지 궁금하면, syslog를 이용해 확인할 수 있습니다.

```console
$ tail /var/log/syslog | grep CRON
Jul 12 19:40:01 ubuntu CRON[13164]: (root) CMD (/sbin/ddclient)
Jul 12 19:42:01 ubuntu CRON[13181]: (root) CMD (/sbin/ddclient)
```

## 웹 서버 띄우기

이제 기본 준비는 끝났으니 서버를 올려봅시다. 앞서 설치한 도커를 활용합니다.  
도커 이미지는 미리 준비한 웹 서비스를 라즈베리파이로 가져와 빌드했습니다.

```sh
docker run --name my-home-page -p 443:443 my-home-image:1.0 -d
```

자 그럼 이제 웹 브라우저로 앞서 DDNS로 연결한 도메인에 접속하면,

![01]({{ site.url }}/assets/images/2019-07-12-setup-raspberrypi-home-server/01.png)

짜잔! 정상적으로 라즈베리파이 웹 서버로 접속됩니다.  
모든 작업이 완료되었습니다.

## 마무리

사실, DDNS 셋업을 해야 한다는 것 말고는 일반적인 우분투 서버와 다를 게 없습니다. 더 이상 돈을 내고 사용하는 클라우드 서비스가 아니므로 별도의 방화벽이나 모니터링 및 대응 시스템이 없는 점이 조금 아쉽지만 하드하게 사용할 것이 아니므로 문제가 되지 않습니다. 작업하면서 실수했던 점은 위에서 설명은 안 했지만 처음에 웹 서버 도커 이미지를 만들 때 라즈베리파이의 아키텍처가 arm64로 다르다는 것을 놓쳤습니다. macOS에서 다른 아키텍처로 도커 이미지를 굽는 것은 현재 `Edge`에서만 되는데, Edge는 Stable이 아니라 빠르게 포기하고 일단은 홈 서버 로컬에서 이미지를 구웠습니다.

## 참고

- [Amazon Lightsail](https://aws.amazon.com/ko/lightsail/)
- [Amazon EC2 요금](https://aws.amazon.com/ko/ec2/pricing/on-demand/)
- [Raspberry Pi](https://www.raspberrypi.org/)
- [Python3 http.server — HTTP servers](https://docs.python.org/3/library/http.server.html)
- [Building Multi-Arch Images for Arm and x86 with Docker Desktop](https://engineering.docker.com/2019/04/multi-arch-images/)
- [Create installation media for Ubuntu](https://ubuntu.com/download/iot/installation-media#windows)
