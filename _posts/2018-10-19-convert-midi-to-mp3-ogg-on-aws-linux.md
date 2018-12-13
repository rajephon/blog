---
title: "Amazon Linux AMI에서 midi를 mp3, ogg로 변환하기"
date: 2018-10-19 03:46:00 +0900
categories: AWS
tags: [timidity, lame, AWS]
layout: post
comments: true
pinned: true
---

## Timidity++
> TiMidity++(간단히 TiMidity)는 하드웨어 신시사이저 없이도 미디 파일을 재생할 수 있는 소프트웨어 신시사이저이다. 실시간으로 사운드 카드를 렌더링할 수 있고, PCM 방식의 .wav 파일과 같은 별개의 파일로 결과물을 저장할 수 있다.  
> [Wikipedia: TiMidity++](https://ko.wikipedia.org/wiki/TiMidity%2B%2B)

개발하는 프로젝트의 서버에서 midi 파일에 특정 사운드 폰트의 소리를 가진 사운드 파일로 변환하는 기능이 필요하다. 이러한 동작은 `Timidity++`나 `FluidSynth`를 활용해 가능하다. 하지만 두 프로그램은 기존에 사용하던 우분투에서는 기본적으로 등록된 apt 레포지토리에 포함된 패키지였는데, Amazon Linux에서는 따로 레포를 등록하거나, 패키지를 내려받아 설치해야 한다.  
패키지를 내려받아 설치할까 싶었지만, 레포에 있는 패키지의 버전이 최신 버전도 아닌 관계로 직접 빌드를 해보기로 했다. `Timidity++`와 `FluidSynth` 중에서 `FluidSynth`는 `libsndfile`, `glib` 등 몇 가지 종속성이 필요해서 `Timidity++`로 정했다.

## 의존 패키지 설치
들어가기에 앞서 각종 빌드 및 설치에 필요한 도구들을 설치한다.
```bash
sudo yum install gcc gcc-c++ automake autoconf libtool yasm nasm git
```

## Timidity++ 빌드 및 설치
현재 홈페이지에는 `2.13.3`이 가장 최신 버전이라고 나오지만, [Sourceforge](https://sourceforge.net/projects/timidity/files/TiMidity%2B%2B/)에서 확인하면 `2.15`가 가장 최신이다.  
홈페이지보다 [Mailing lists](https://sourceforge.net/p/timidity/mailman/timidity-devel/)에서 최신 정보를 업데이트를 하고 있는 것 같다. 가장 최근 버전인 `2.15`를 내려받았다. sourceforge에서 버전 리스트를 확인 후 고르면 될 것이다.
```bash
wget https://sourceforge.net/projects/timidity/files/TiMidity%2B%2B/TiMidity%2B%2B-2.15.0/TiMidity%2B%2B-2.15.0.tar.gz
tar -zxvf TiMidity++-2.15.0.tar.gz
cd TiMidity++-2.15.0
./autogen.sh # autogen.sh에 ./configure가 포함되어 있다.
make
sudo make install
```
정상적으로 빌드 및 설치가 되었다면 `timidity --version`을 입력했을 경우 아래와 같이 나올 것이다.

```bash
$timidity --version
TiMidity++ version 2.15.0

Copyright (C) 1999-2018 Masanao Izumo <iz@onicos.co.jp>
Copyright (C) 1995 Tuukka Toivonen <tt@cgs.fi>

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.
```
timidity를 이용해서 midi 파일을 컨버팅을 할 때에는 `timidity.cfg`에 사용할 사운드폰트의 경로 설정이 필요하다. `timidity.cfg` 파일의 기본 경로는 `/usr/local/share/timidity/timidity.cfg`다. 만약 설정을 하지 않았고 기본 경로를 모를 경우, `timidtiy --help`와 같은 `timidity` 명령어를 입력하면 출력 최상단에 `/some/path/timidity/timidity.cfg file or directory not found`와 같이 경로를 알려줄 것이다.  
`timidity.cfg`에 사운드폰트 경로 설정은 다음과 같이 하면 된다. 
```text
soundfont /path/to/my/soundfonts/gm.sf2
```

## Timidity++ 사용 (.midi to .wav)
간단하다. `-Ow` 옵션으로 입력된 파일을 `wav` 파일로 변환하도록 하면, 입력된 미디 파일과 똑같은 파일, 경로에 `wav` 포맷의 파일이 생성된다.
```bash
timidity -Ow ~/target_file.midi 
```
혹은 직접 변환된 파일의 위치를 지정할 수 있다.
```bash
timidity ~/target_file.midi -o ~/target_file.wav -Ow 
```

또한 `--output-24bit`, `--output-16bit`, `--output-8bit`와 같은 옵션으로 퀄리티를 세팅할 수도 있다. 퀄리티가 높을수록 출력 파일의 용량도 커지며, 기본은 `16bit`다.

다만, 이 상태 그대로 사용하기엔 변환된 파일의 크기가 너무 크다. 2분짜리 음악조차도 16bit로 변환했음에도 20MB가 넘는다. `mp3` 포맷으로 변환할 수 있으면 좋지만 `Timidity++`나 `fluidsynth`에서는 바로 `mp3` 포맷으로 변환하는 기능을 제공하지 않는다. 그러므로 `lame`을 활용해서 한 번 더 변환을 거쳐 `mp3` 로 출력하게 시도해보기로 했다.

## LAME 빌드 및 설치
> LAME은 오픈 소스 MP3 인코더이다. LAME이라는 이름은 “LAME Ain't an MP3 Encoder”의 재귀 약자이다. 이름은 이렇게 지었지만, 실제로는 오늘날 가장 널리 쓰이는 MP3 인코더 중의 하나이다.  
> [Wikipedia: LAME](https://ko.wikipedia.org/wiki/LAME)

우분투 사용자라면 그냥 `apt install lame` 명령어로 설치 가능하다. 하지만 우리는 lame도 마찬가지로 직접 빌드를 해서 설치해야 한다.

timidity와 마찬가지로 [sourceforge의 lame 프로젝트](https://sourceforge.net/projects/lame/files/lame/)에서 최신 버전을 확인 후 내려받는다.

```bash
wget https://sourceforge.net/projects/lame/files/lame/3.100/lame-3.100.tar.gz
tar -zxvf ./lame-3.100.tar.gz
cd lame-3.100
./configure
make
sudo make install
```

## LAME 사용 (.wav to .mp3)
`timidity`보다 더 간단하다. 첫 번째 인자로 입력 파일, 두 번째 인자로 출력 파일을 입력하면 된다.
```bash
lame ./target_file.wav ./target_file.mp3
```
기타 `-b` 옵션으로 비트레이트를 설정하거나(기본 128), `-h`로 고음질에 느린 변환 속도, `-f`로 저음질에 빠른 변환 속도 등의 간편 옵션도 있다.  
기타 세세한 설정은 `-longhelp`로 볼 수 있다.

## Timidity++ & LAME 파이프 사용 (.midi to .wav to .mp3)
리눅스 파이프를 이용해 한 줄로 바로 변환이 가능하다.  
`timidity`에서 `-o -`는 stdout(표준 출력)으로 변환 결과를 내보내라는 의미고, `lame`에서 `-` 옵션은 infile 혹은 outfile 위치에 따라 stdin/stdout이라는 의미로 timidity를 이용해 변환된 `wav` 포맷 파일을 바로 `lame`에 입력할 수 있다.
```bash
timidity ~/target.midi -Ow -o - | lame - ./target.mp3
```


## 추가: Timidity++에서 다른 확장자로 변환하고 싶다면? (.midi to .ogg)
`timidity`는 `mp3`를 지원하지 않을 뿐이지 생각보다 다양한 포맷을 지원하고 있다. `ogg`나 `flac`으로 변환하려면 별도의 라이브러리를 설치하고, 처음 `./configure`할 때 옵션을 줘야 한다.
`./configure --help` 명령어로 추가로 동작시킬 수 있는 포맷의 종류를 볼 수 있다.

`ogg` 포맷으로 변환하려면 `libogg`와 `libvorbis` 라이브러리 설치가 필요하다.
```bash
sudo yum install libogg-devel libvorbis-devel
```

그리고 `--enable-audio=vorbis` 옵션을 줘서 다시 빌드한다. 기존에 설치된 `timidity`는 `sudo make uninstall`로 지우고 새 빌드로 다시 설치한다.
```bash
./autogen.sh --enable-audio=vorbis
make
sudo make install
```

그리고 `timidity --help`에 보면 `Available output modes`에 `Ogg Vorbis`가 추가된 것을 볼 수 있다.
```bash
$ timidity --help
...
Available output modes (-O, --output-mode option):
  -Ow          RIFF WAVE file
  -Or          Raw waveform data
  -Ou          Sun audio file
  -Oa          AIFF file
  -Ov          Ogg Vorbis
  -Ol          List MIDI event
  -Om          Write MIDI file
  -OM          MOD -> MIDI file conversion
...
```
도움말에 나와있는 것과 같이 아웃풋 모드를 `-Ov`로 하면 `ogg` 파일로 변환할 수 있다.
```bash
timidity ~/maple.midi -o ~/maple.ogg -Ov
```


## Reference
- [Sourceforge: Timidity++](https://sourceforge.net/projects/timidity/)
- [Archlinux WiKi: timidity#SoundFonts](https://wiki.archlinux.org/index.php/timidity#SoundFonts)
- [Mankier.com: TIMIDITY MAN PAGE](https://www.mankier.com/1/timidity)
- [Mankier.com: LAME MAN PAGE](https://www.mankier.com/1/lame)
- [xiph.org: vorbis](https://xiph.org/vorbis/)