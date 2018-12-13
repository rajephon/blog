---
title: "OpenWrt 패키지 빌드 환경 세팅하기"
date: 2018-11-09 18:47:00 +0900
categories: OpenWrt
tags: [OpenWrt, FREEZIO, WiFi]
layout: post
comments: true
pinned: true
---
![OpenWrt]({{ site.url }}/assets/images/OpenWrt-00/00.jpg)  
<sub>출처 [flickr:Gareth Halfacree](https://www.flickr.com/photos/120586634@N05/15614095254)</sub>

## Intro
`OpenWrt`는 WiFi 라우터를 위한 리눅스 기반의 OS입니다. 오픈 소스이기 때문에 쉽게 자신이 원하는 기능을 올리거나 수정하는 등 손쉽게 커스터마이징을 할 수 있다는 점에서 라즈베리파이나 `OpenWrt` 호환 라우터를 구입해서 나만의 WiFi 공유기를 만들 때 인기가 많습니다. 오늘은 라우터의 아키텍쳐에 맞춰 `Openwrt`의 [툴체인](https://ko.wikipedia.org/wiki/%ED%88%B4%EC%B2%B4%EC%9D%B8)을 컴파일하고, 해당 툴체인을 활용해 패키지를 컴파일하는 방법을 설명하겠습니다.  

참고로 저는 `FREEZIO`라는 공유기를 기준으로 설명하겠습니다.  
`FREEZIO`는 국내에서 가장 쉽게 구할 수 있는 `OpenWrt`기반의 WiFi 라우터입니다. 출시부터 커스터마이징을 하기 편하게 `OpenWrt` 가 올려진 채로 나왔고, `OpenWrt`가 GPL라이선스인 만큼 [깃허브](https://github.com/ZIORouter/openwrt_for_mtk)에 펌웨어 코드도 같이 올라와 있어 사용하기 편리합니다.  
기타 [샤오미 WiFi](https://oldwiki.archive.openwrt.org/toh/xiaomi/start) 등 여러 공유기 또한 `OpenWrt`를 사용하기 편리하다고 하니 아직 공유기를 정하지 못하신 분께서는 참고 바랍니다.

※ `OpenWrt`는 `GPL-2.0` 라이선스입니다. 이에 따라 자신이 `OpenWrt`를 활용해 작성한 소스코드를 공개해야 할 수 있다는 점 주의 바랍니다. [관련 링크](https://openwrt.org/license)

## 개발환경
`Ubuntu`에서 컴파일을 진행하는 것을 기준으로 작성되었습니다.

## Install
가장 먼저 의존성 패키지들을 설치합니다.
```bash
# Install build dependencies
sudo apt-get install subversion g++ zlib1g-dev build-essential git python
sudo apt-get install libncurses5-dev gawk gettext unzip file libssl-dev wget
```
정상적으로 설치가 완료되면 `FREEZIO` 모델에 맞춰진 빌드시스템을 내려받습니다. 필요에 따라 [공식 레포](https://github.com/openwrt/openwrt) 혹은 기타 제작사, 사용자가 올린 레포를 활용할 수 있습니다.
```bash
git clone https://github.com/ZIORouter/openwrt_for_mtk
```
레포 내부의 `feeds`를 이용해 기본 패키지를 인스톨합니다.
```bash
cd openwrt_for_mtk
./scripts/feeds update -a
./scripts/feeds install -a
```
추가로 설치하고 싶은 패키지가 있을 경우, `package/`디렉토리에 추가합니다.  
[github.com/openwrt-routing/packages](https://github.com/openwrt-routing/packages.git)와 같은 레포를 내려받고, 원하는 기능만 골라서 옮겨 담아도 좋습니다.  
여기서 `package/`에 설치된 패키지는 `make menuconfig`를 할 때 패키지 선택지에 나타납니다.

## Build
본인의 WiFi 라우터 모델과 설치하고 싶은 패키지에 알맞게 설정을 진행합니다.
```bash
make defconfig
make menuconfig
```
`make menuconfig`를 수행할 경우 선택 옵션창이 나타납니다. 거기서 `FREEZIO`기준 다음과 같이 세팅합니다.
- `Target System` > `MTK/Ralink APSoC (MIPS)` > `<*>` 체크
- `Subtarget` > `MT7621 based boards` > `<*>` 체크

설명을 추가하면, `Target System`은 공유기의 시스템 아키텍쳐를 의미합니다. 
 `<*>`는 펌웨어 이미지에 빌트인으로, `<M>`은 패키지 형태로 빌드를 한다는 의미입니다. 
빌드하고 싶은 패키지 선택이나 모든 세팅이 끝나면 `.config`로 저장 후 빠져나옵니다.  

그리고 툴체인 인스톨을 진행합니다.
```bash
make tools/install
make toolchain/install
```

각 패키지별 빌드는 다음과 같이 진행합니다.
```bash
make package/<package_name>/compile
make package/<package_name>/install
```

정상적으로 빌드가 성공할 경우 `bin/`디렉토리에 .ipk 파일이 생성됩니다.

## 예상 문제 발생 지점
### 툴체인 인스톨이 정상적으로 이루어지지 않아요.
`./scripts/download.pl` 파일의 커널 아카이브 링크를 한번 체크합니다. 공식 레포 기준 [이 라인](https://github.com/openwrt/openwrt/blob/master/scripts/download.pl#L236)의 주소가 `cdn.kernel.org` 혹은 `www.kernel.org`가 아닌 `ftp.all.kernel.org`와 같이 더 이상 사용하지 않는 주소로 되어있을 경우 오류가 발생합니다. 오래전 `OpenWrt`에서 fork되어 나온 레포일 경우 수정해줘야 할 가능성이 큽니다. `FREEZIO` 같은 경우 본인이 풀리퀘스트를 올려 변경해두었습니다.

## Outro
예전에 회사다니면서 서브 프로젝트로 `OpenWrt`를 접하면서 익혔던 지식을 복습할 겸 작성했습니다.  
추후에 실제로 패키지를 빌드해보면서 내용을 추가 및 수정하도록 하겠습니다.  
잘못된 정보가 있다면 지적 달게 받겠습니다 :D  

## Reference
- [openwrt.org](https://openwrt.org/)
- [github.com/openwrt/openwrt](https://github.com/openwrt/openwrt)
- [github.com/ZIORouter/openwrt_for_mtk](https://github.com/ZIORouter/openwrt_for_mtk)
- [github.com/openwrt-routing/packages.git](https://github.com/openwrt-routing/packages.git)