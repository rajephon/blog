---
title: "D-Link 공유기 파이썬으로 원격 부팅(WOL) 이용하기"
date: 2017-11-28 01:45:14 +0900
categories: network
layout: post
comments: true

---
요즘 나오는 대부분의 공유기는 원격 부팅(WOL) 기능을 제공합니다.

하지만 공유기 관리자 페이지에 로그인을 한 다음, 원격 부팅(WOL) 설정 페이지로 이동하여 원하는 PC를 선택하고 부팅하는 과정을 매번 하기에 번거롭지요.
이런 귀찮은 과정을 한 번에 처리할 수 있도록 파이썬 스크립트를 짜봤습니다. 단지 HTTP 리퀘스트 몇 번으로 손쉽게 가능하더군요.

# 기본사항
- 이 스크립트는 DIR-806A모델, v1.05KR 펌웨어 기준으로 작성되었습니다. 모델이나 펌웨어마다 접속경로 등이 달라 스크립트가 동작하지 않을 수 있습니다.
- 동작에는 다음과 같은 정보가 필요합니다.
   - 공유기 관리자 계정(아이디와 패스워드)
   - 관리자 페이지 주소(DDNS가 설정되어있을 경우 외부 접속이 가능하므로 더욱 좋습니다)
- 동작에는 다음과 같은 사전설정이 필요합니다.
   - 타겟 PC의 WOL 바이오스 설정
   - D-Link 관리자 페이지-원격부팅(WOL) 설정 페이지의 원격부팅(WOL) 항목에 타겟 PC의 MAC 주소 등록

# Gist
<script src="https://gist.github.com/rajephon/f825f1055e909b24f655a75b15d575bb.js"></script>

# 사용법
내려받은 다음 편집기를 이용하여 17~19번째 라인을 자신의 환경에 알맞게 수정해주세요.
```Python
username = "관리자ID"
password = "관리자PW"
url = "http://D-Link 공유기 주소/"
```
그 다음 파이썬을 이용해 실행합니다.
```bash
$ python ./wol.py
Python D-Link 804 WOL Servcie
- rajephon(at)gmail.com

Connecting... 
url : http://D-Link 공유기 주소/
username : 관리자ID
found mac address list
--------------------------
1. 12:34:56:78:90:ab,WIN10
Select MAC address : 1 # 원격 실행을 원하는 PC의 번호를 입력해주세요.
Send request wol 12:34:56:78:90:ab,WIN10...
request success
```

이제 원격 접속 프로그램 등을 이용하여 PC가 켜진 것을 확인하고 이용하실 수 있습니다.

# License
MIT