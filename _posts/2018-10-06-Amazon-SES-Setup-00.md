---
title: "아마존 SES 세팅부터, 개인 도메인 메일 송수신, Node.js 활용 메일 전송까지 (22.04.30 업데이트)"
date: 2018-10-06 17:59:25 +0900
categories: amazon SES
tags: [AWS, SES, NodeJS]
layout: post
comments: true

---

## 수정기록

- 2018-10-06 최초작성
- 2022-04-30 내용 업데이트
  - 아마존 SES 콘솔이 전반적으로 바뀌어 내용을 전반적으로 업데이트

## Intro

개인 도메인의 이메일 주소를 사용하는 방법은 다양하다. [LINK WORKS](https://line.worksmobile.com)(구 네이버 웍스)나 [Google Apps](https://apps.google.com/user/hub)와 같은 그룹웨어를 이용하는 방법, [mailgun](https://www.mailgun.com/), [Amazon SES](https://aws.amazon.com/ko/ses/)와 같은 이메일 서비스를 이용하는 방법, 직접 SMTP 서버를 구축하는 방법 등이 있다.

라인 웍스나, 구글 앱스 모두 예전엔 무료 요금제도 존재했지만 지금은 모두 유료로 전환되어 개인이 사용하기에 부담감이 있다. 또한 본인이 본인 도메인의 이메일 주소를 얻기 위해서 직접 SMTP 서버를 구축 및 유지관리하는 것은 그다지 효율성 있는 방법은 아니라 생각한다.

더구나 개인이 집에서 운영하여 유동 IP를 할당받아 지속적으로 IP가 변동되는 사설서버라면 더욱 그렇다. 이메일의 발신자가 위조된 것이 아니라는 것을 검증하기 위하여 DNS에 SPF(Sender Policy Framework) 레코드 등록해야 하는데, 메일 서버 IP 주소가 지속적으로 바뀌면 SPF에 설정된 IP도 계속 바꿔줘야 한다. 또한 화이트 도메인 신청도 해줘야 한다.

mailgun은 한 달에 1만 건까지 이메일을 무료로 전송할 수 있으며, 다양한 내부 서비스를 제공하고 있다. 그리고 SES에 비해 등록이 간편하고 바로 사용할 수 있다

아마존 SES는 등록된 이메일이 아닐 경우 샌드박스 해제 과정을 거쳐야 하는 번거로움이 있다. 대신 Amazon SES는 Amazon EC2에 호스팅된 애플리케이션에서 이메일을 발송할 경우 매월 62,000건까지 무료로 전송할 수 있다. 샌드박스 해제 방법은 [Moving out of the Amazon SES sandbox](https://docs.aws.amazon.com/ses/latest/dg/request-production-access.html)을 참고하길 바란다.

이번 포스트에서는 아마존 SES를 활용하는 방법을 작성해보겠다.

## MX 레코드 등록

SES 사용 준비가 끝나면, 도메인에 MX 레코드를 등록한다. MX 레코드는 도메인으로 전송되는 이메일을 수신하기 위해 메일 서버를 지정하는 것이다[^1].

도메인에 설정한 DNS 프로바이더에서 사용할 도메인에 설정값을 추가한다. 타입은 `MX`로 지정하고 값은 다음과 같다.

```text
inbound-smtp.<region>.amazonaws.com
```

&lt;region&gt;은 SES 서비스를 사용할 리전을 선택한다. 나는 오레곤 지역을 사용하고 있으므로 `us-west-2`로 설정했다. 우선순위(Priority)는 `10`으로 설정한다.

![mx-record]({{ site.url }}/assets/images/Amazon-SES-Setup-00/mx.png)  

하는 김에 겸사 SPF도 같이 설정해 주자. TXT 레코드로 다음의 값을 추가한다[^2].

```text
"v=spf1 include:amazonses.com ~all"
```

큰따옴표까지 전부 넣어줘야 함에 주의한다. include가 여럿일 경우 다음과 같이 이어붙여서 설정한다.

```text
"v=spf1 include:example.com include:amazonses.com ~all"
```

## 도메인 인증

이제 본격적으로 시작해 보자. [Amazon SES 콘솔](https://us-west-2.console.aws.amazon.com/ses/home?region=us-west-2#/verified-identities)에서 `Configuration` - `Verified identities`으로 들어간다.

먼저, AWS 콘솔에서 SES(Simple Email Service)를 선택하고 리전을 고른다. 리전은 미국 동부(버지니아 북부), 미국 서부(오레곤), EU(아일랜드) 중에 고를 수 있다. 본인은 미국 서부(오레곤)을 선택했다.

`Create identity`를 눌러서 도메인을 추가한다.

![SES 00]({{ site.url }}/assets/images/Amazon-SES-Setup-00/create-identity.png)  

![SES 01]({{ site.url }}/assets/images/Amazon-SES-Setup-00/create-identity2.png)  

메일 송수신에 사용할 도메인을 입력하고 Easy DKIM을 선택한다. DKIM signing key length는 2048비트를 선택한다.  
나는 Route53이 아닌 Cloudflare를 사용하고 있으므로 `Publish DNS Records to Route53`을 체크하지 않았다. 만약 Route53을 사용하고 있다면 `Enable`에 체크해서 자동을 CNAME을 등록하도록 처리해도 좋다.

![pending]({{ site.url }}/assets/images/Amazon-SES-Setup-00/pending.png)

이후 콘솔을 확인해 보면, `DKIM configuration`이 `Pending`으로 표시되며 CNAME을 등록해달라는 메세지가 나타난다.
내용과 같이 해당하는 도메인 이름에 값을 연결해 주면 된다. Cloudflare를 사용하는 경우, Orange Cloud(프록시)를 사용하면 안 된다. 프록시를 사용할 경우 유효성 검사가 제대로 이루어지지 않을 수 있다[^3].

설정 후 잠시 기다리면 Pending이 `Verified`로 바뀐다. DKIM 셋업이 성공했다고 이메일이 오기도 하니 여유롭게 기다리자.

다음으로 `MAIL FROM`을 설정하자. `MAIL FROM`은 Amazon SES를 통해 전송한 이메일이 아마존의 서브도메인이 아니라 내가 설정한 도메인을 통해 발송하도록 표시를 해준다. `amazonses.com`의 서브도메인으로 메일이 전송되더라도 SPF 인증을 통과하므로 충분하지만, 일부 발신자의 경우 `MAIL FROM` 도메인으로 설정하는 것을 선호한다. `MAIL FROM` 도메인을 설정하면 DMARC를 준수할 수 있고, 이를 통해 해당 도메인에서 발송된 이메일이 하나 이상의 인증 시스템에 의해 보호되고 있음을 나타낼 수 있다[^4].

설정 방법은 똑같이 MX 레코드와 TXT 레코드 설정을 통해 할 수 있다.

## 이메일 수신 처리

아마존 SES는 `Rule Set` 정의를 통해 메일이 수신될 경우의 처리를 제어할 수 있다. 이점을 활용하여 내 도메인의 이메일 주소로 메일이 수신될 경우, gmail로 자동 포워딩하는 세팅을 하려 한다. 꼭 gmail이 아니어도 되지만, 내가 사용하는 메일이 gmail이므로 그쪽으로 보낼 것이다. 이 과정은 [arithmetric/aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder)의 도움을 받을 것이므로 자세한 정보는 해당 레포를 참고 바란다.  
다음과 같은 룰을 만들 것이다.

1. xxx@example.com으로 메일이 수신될 경우, 내 버킷에 이메일을 저장한다.
2. 그리고 Lambda Action을 통해 저장된 email을 콘텐츠로 내 gmail 계정으로 송신한다.

### 버킷 생성

이메일이 저장될 버킷을 생성하고, 다음과 같이 권한을 설정한다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowSESPuts",
            "Effect": "Allow",
            "Principal": {
                "Service": "ses.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<버킷이름>/*",
            "Condition": {
                "StringEquals": {
                    "aws:Referer": "<계정 ID>"
                }
            }
        }
    ]
}
```

`<버킷이름>`에는 본인의 버킷 이름을, `계정 ID`에는 AWS 우상단 이름을 눌렀을 때 나오는 계정 ID를 하이픈`-`없이 숫자만 입력한다.

### 람다 액션 생성

1. [https://github.com/arithmetric/aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder) 의 코드를 내려받거나 클론한다. 혹은 index.js 코드만 복사하여 저장한다. (우리에게 필요한 것은 index.js뿐이다.)
2. `index.js`에 있는 `defaultConfig`값을 본인에게 알맞게 수정한다. 기본값 예시가 잘 되어있어서 설정이 어렵지는 않을 것이다.  
    `fromEmail`: 내 gmail로 전달할 때 활용할 송신자 이메일 주소  
    `emailBucket`: 이메일을 저장할 S3 버킷 이름  
    `emailKeyPrefix`: S3 버킷에서 저장될 위치의 KeyPrefix
    `forwardMapping`: 어느 이메일로 메일이 오면 어디로 전달을 할지에 대한 매핑
3. AWS 람다에서 새로운 함수를 생성한다.
    `새로 작성`을 선택하고, 이름은 "SesForwarder"와 같이 본인이 알아볼 수 있게 적당히 지어준다. 런타임은 `Node.js`을 선택한다.  
    `기본 실행 역할 변경`은 `기본 Lambda 권한을 가진 새 역할 생성`로 설정한다. 이후 역할을 수정할 것이다.
4. Lambda의 함수 코드에는 이전에 내려받고, `defaultConfig`를 수정한 `index.js`를 복사하여 붙여 넣는다. 핸들러가 `index.handler`로 설정되어 있는지 확인한다.
5. `구성` - `일반 구성` 탭에서 편집을 눌러 설정값을 수정한다.
   메모리는 128 MB 그대로 두고, 제한 시간은 10초 정도면 충분하다. 보통 작업은 30 MB 메모리와 몇 초 걸리지 않는다. 혹은 대용량 첨부 파일이 있는 경우, 512 MB 메모리와 30초 정도의 제한 시간이 필요할 수 있다.
6. 설정 저장 후, `구성` - `권한` 탭에서 역할 이름을 눌러 IAM 역할 정책을 수정한다. `<리전이름>`, `<계정ID>`, `<S3버킷이름>`부분은 본인에게 알맞게 수정하여 설정한다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:<리전이름>:<계정ID>:*"
        },
        {
            "Effect": "Allow",
            "Action": "ses:SendRawEmail",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::<S3버킷이름>/*"
        }
    ]
}
```

### 이메일 인증

다시 AWS SES로 돌아와서, 이메일을 전달받을 메일 주소(본인의 경우 Gmail 주소)를 인증한다. (만약 이미 Sandbox 제한이 해제되어 있을 경우, 이 과정은 생략해도 된다.)  
`Amazon SES` - `Configuration: Verified identities` - `Create Identity`로 들어가서, 이메일 주소를 입력한 다음, 인증 메일을 확인하면 바로 처리된다.

### Rule 생성

이제 앞서 설명한 바와 같이 이메일이 수신되면 S3에 저장하고, Lambda를 통해 전달하는 룰을 생성해 보자.  

- `Amazon SES` - `Configuration: Email receiving` - `Create rule set`으로 룰셋을 생성한다. 이미 사용하는 룰셋이 있을 경우 그것을 활용해도 좋다.
- 적당한 이름으로 룰셋을 만들고, 룰셋 안에서 `Create rule`로 새 룰을 생성한다.

![create rule]({{ site.url }}/assets/images/Amazon-SES-Setup-00/rule.png)

- 마찬가지로 이름을 적당히 지어주고, `Next` 버튼을 클릭한다.

![create rule]({{ site.url }}/assets/images/Amazon-SES-Setup-00/rule2.png)

- `Recipient conditions` 에 수신할 이메일의 규칙을 설정한다. `john@example.com`와 같이 특정 이메일을 지정하거나, `example.com`와 같이 도메인으로 오는 이메일에 동작하도록 할 수도 있다. `Guidelines`에 적절한 예시들이 있으니 확인을 해보길 바란다.

![create rule]({{ site.url }}/assets/images/Amazon-SES-Setup-00/rule3.png)

- 1번으로 `Deliver to Amazon S3 bucket`을 선택한다.
  - S3 bucket에는 앞서 만든 버킷을 선택한다.
  - Object key prefix는 버킷 내 이메일을 저장할 위치다. 앞서 lambda 코드에서 설정했다면 똑같이 맞춰서 지정한다.
- `Add new action`을 선택하여 두 번째 액션으로 `Invoke AWS Lambda function`을 선택한다.
  - `Lambda function`에 앞서 생성한 lambda를 지정한다.
- `Next`를 눌러 저장한다.

모든 설정이 끝난 다음, 테스트 이메일을 전송한다.

![receive test]({{ site.url }}/assets/images/Amazon-SES-Setup-00/receive.png)

모든 설정이 정상적으로 이루어졌다면 메일이 잘 수신될 것이다.  
보낸 사람은 lambda에 설정한 `fromEmail`, 답장 주소는 실제로 이메일을 보낸 이메일로 나타난다.  
받는 사람은 내 개인 도메일 메일 주소로, 발송 도메인은 MAIL FROM으로 설정한 서브도메인이다.

여담으로, 이메일 포워딩도 결국 SES, S3, Lambda 같은 아마존 자원을 사용하기에, 비용이 많이 발생하지 않을까 걱정했는데, Github Issue 글을 보니 4만 8천 건당 1달러 정도로 아주 작아 신경 쓰지 않아도 될 정도라고 한다[^5].

## 이메일 발송 처리

이메일 발송 방법은 크게 두 가지가 있다. Gmail 같은 상용 메일 서비스에 중계 서버를 등록해서 전송하는 방법, AWS SDK를 활용해서 서버 등 자신이 만들고 있는 서비스에서 활용하는 방법. 두 가지 모두 순서대로 설명하겠다.
**참고로 AWS SES는 Sandbox 제한 해제 요청을 따로 하지 않을 경우, `Amazon SES` - `Configuration: Verified identities`에 인증해둔 이메일 주소로만 메일을 전송할 수 있다.** 타인에게 이메일을 전송하려면, Sandbox 제한 해제 요청을 꼭 해야 한다.

### Gmail에서 내 도메인 메일 전송하기

앞서 Gmail로 중계하도록 처리했으니, Gmail에서 내 도메인 메일 주소를 발신자로 설정하여 메일을 전송하는 방법을 설정해 보자.

- 먼저, 발신에 사용할 이메일 주소를 인증해야 한다. 샌드박스에서 나왔거나 이미 인증해두었으면 넘어간다. `Amazon SES` - `Configuration: Verified identities`에서 `Create Identity` - `Email Address`로 등록한다.
- `Amazon SES` - `Account dashboard` - `Simple Mail Transfer Protocol (SMTP) settings` - `Create SMTP credentials`를 클릭하여 새 credential을 생성한다.
- SMTP 인증을 위한 새 IAM 사용자를 생성하게 되는데, 적절한 이름을 지정해 준다.

![iam smtp user]({{ site.url }}/assets/images/Amazon-SES-Setup-00/iam-smtp-user.png)

- 이후 SMTP 유저 이름과 비밀번호가 출력된다. 우하단 `자격 증명 다운로드` 버튼을 눌러 SMTP 보안 자격 자격증명의 csv 파일을 내려받을 수 있는데, 자격 증명을 확인하거나 다운로드할 수 있는 유일한 시점으로 꼭 기록하거나 내려받는다.

- Gmail에서 설정 - `계정 및 가져오기` - `다른 주소에서 메일 보내기` - `다른 이메일 주소 추가`를 클릭한다. 이름은 메일 발송 시 타인에게 표시될 기본 이름을 작성하고, 등록할 내 도메인 이메일 주소를 입력한다.

![SES 06]({{ site.url }}/assets/images/Amazon-SES-Setup-00/06.png)

- SMTP 서버와 포트는 `Amazon SES` - `Account dashboard` - `Simple Mail Transfer Protocol (SMTP) settings` 에서도 확인할 수 있다.
- SMTP 서버 : `email-smtp.<SES리전주소>.amazonaws.com`
- 포트 : 25, 587 or 2587
- 사용자 이름과 비밀번호 방금 생성한 SMTP credentials를 입력한다.
- 계정 추가 버튼을 누르면 인증 메일이 발송되는데, 메일이 도착하는 대로 메일에 있는 인증 코드를 입력하거나 인증 링크를 누르면 모든 설정이 끝난다.

![mail send]({{ site.url }}/assets/images/Amazon-SES-Setup-00/mail-send.png)

- 모든 과정이 끝나고 `편지 쓰기`를 눌러보면 `보낸 사람`을 내가 추가한 메일 주소로 할 수 있는 것을 확인할 수 있다.

### AWS SDK를 활용하여 메일 전송하기

`Node.js`에서 `nodemailer`와 `aws-sdk`를 활용해 메일 전송을 해보도록 하자.

1. Node.js에서 사용할 IAM 사용자 계정이 없을 경우, AWS IAM에서 사용자를 추가한다.
    액세스 방식은 `프로그래밍 방식 액세스`를 체크한다.
    정책은 Email 전송과 관련된 권한으로 하나 만들어준다.
    생성한 사용자에 관한 Credentials.csv 파일을 안전한 곳에 보관한다.
2. 프로젝트 디렉터리에서 aws-sdk, nodemailer 모듈을 설치한다.

```bash
npm install aws-sdk nodemailer --save 
```

3. 적절한 이름으로 AWS 설정 파일을 생성한다. `aws_config.json`라는 이름으로 해줬다. 앞서 만든 사용자의 키를 입력하고, 리전 코드는 [AWS Docs](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)에서 확인할 수 있다.

```json
{
  "accessKeyId": "AWS_ACCESS_KEY_ID", 
  "secretAccessKey": "AWS_SECRET_ACCESS_KEY",
  "region": "AWS_REGION"
}
```

4. 그리고 프로젝트 소스를 작성한다.

```javascript
const nodemailer = require('nodemailer');
const AWS = require('aws-sdk');
AWS.config.loadFromPath(__dirname+'/aws_config.json');

const transporter = nodemailer.createTransport({
    SES: new AWS.SES({
        apiVersion: '2010-12-01'
    })
})

transporter.sendMail({
  from: '내-도메인-이메일-주소',
  to: '수신자-이메일-주소',
  subject: 'Node.js에서 발송한 메일',
  text: 'Hello World'
}, (err, info) => {
  if (err) {
    error(err);
  }
  console.log('sendEmail: '+ JSON.stringify(info.envelope));
  console.log(info.messageId);
});
```

메일 폼을 적절히 자신에게 알맞게 수정한다. 그리고 명령어를 통해 Node.js를 실행하면 발송 로그를 출력한다.

```bash
$ node index.js
sendEmail: {"from":"내-도메인-이메일-주소","to":["수신자-이메일-주소"]}
<some-id@us-west-2.amazonses.com>
```

![SES 08]({{ site.url }}/assets/images/Amazon-SES-Setup-00/08.png)  

수신 메일 주소의 메일함에 들어가 보면, 메일이 잘 도착한 것을 확인할 수 있다.

## 참고 자료

- [발송 메일을 스팸으로 분류되지 않도록 개선하는 법 - 아사마루](https://blog.asamaru.net/2015/12/16/how-can-i-prevent-mail-from-being-classified-as-spam/)
- [White Domain 소개 - KISA 불법스팸대응센터](https://www.kisarbl.or.kr/whiteip/whiteip_tutorial.jsp)  
- [Moving out of the Amazon SES sandbox](https://docs.aws.amazon.com/ses/latest/dg/request-production-access.html)
- [github.com/arithmetric/aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder)

## 각주

[^1]: [Publishing an MX record for Amazon SES email receiving](https://docs.aws.amazon.com/ses/latest/dg/receiving-email-mx-record.html)
[^2]: [Authenticating Email with SPF in Amazon SES](https://docs.aws.amazon.com/ses/latest/dg/send-email-authentication-spf.html)
[^3]: [AWS SES DKIM cannot be proxied?](https://community.cloudflare.com/t/aws-ses-dkim-cannot-be-proxied/237431)
[^4]: [Why use a custom MAIL FROM domain?](https://docs.aws.amazon.com/ses/latest/dg/mail-from.html)
[^5]: [Cost?](https://github.com/arithmetric/aws-lambda-ses-forwarder/issues/68#issuecomment-418201013)
