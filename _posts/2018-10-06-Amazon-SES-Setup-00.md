---
title: "아마존 SES 세팅부터 Node.js 활용 메일 전송까지"
date: 2018-10-06 17:59:25 +0900
categories: amazon SES
tags: [AWS, SES, NodeJS]
layout: post
comments: true

---

## Intro
개인 도메인의 이메일 주소를 사용하는 방법은 다양하다. [LINK WORKS](https://line.worksmobile.com)(구 네이버 웍스)나 [Google Apps](https://apps.google.com/user/hub)와 같은 그룹웨어를 이용하는 방법, [mailgun](https://www.mailgun.com/), [Amazon SES](https://aws.amazon.com/ko/ses/)와 같은 이메일 서비스를 이용하는 방법, 직접 SMTP 서버를 구축하는 방법 등이 있다.

라인 웍스나, 구글 앱스 모두 예전엔 무료 요금제도 존재했지만 지금은 모두 유료로 전환되어 개인이 사용하기에 부담감이 있다. 또한 본인이 본인 도메인의 이메일 주소를 얻기 위해서 직접 SMTP 서버를 구축 및 유지관리하는 것은 그다지 효율성 있는 방법은 아니라 생각한다. 

더구나 개인이 집에서 운영하여 유동 IP를 할당받아 지속적으로 IP가 변동되는 사설서버라면 더욱 그렇다. 이메일의 발신자가 위조된 것이 아니라는 것을 검증하기 위하여 DNS에 SPF(Sender Policy Framework) 레코드 등록해야 하는데, 메일 서버 IP 주소가 지속적으로 바뀌면 SPF에 설정된 IP도 계속 바꿔줘야 한다. 또한 화이트 도메인 신청도 해줘야 한다.

mailgun은 한 달에 1만 건까지 이메일을 무료로 전송할 수 있으며, 다양한 내부 서비스를 제공하고 있다. 그리고 SES에 비해 등록이 간편하고 바로 사용할 수 있다

아마존 SES는 등록된 이메일이 아닐 경우 샌드박스 해제 과정을 거쳐야 하는 번거로움이 있다. 대신 Amazon SES는 Amazon EC2에 호스팅된 애플리케이션에서 이메일을 발송할 경우 매월 62,000건까지 무료로 전송할 수 있다.

이번 포스트에서는 아마존 SES를 활용하는 방법을 작성해보겠다.

## 도메인 인증
먼저, AWS 콘솔에서 SES(Simple Email Service)를 선택하고 리전을 고른다. 리전은 미국 동부(버지니아 북부), 미국 서부(오레곤), EU(아일랜드) 중에 고를 수 있다. 본인은 미국 서부(오레곤)을 선택했다.

![SES 00]({{ site.url }}/assets/images/Amazon-SES-Setup-00/00.png)  

그리고 좌측 메뉴에서 `Domains`를 선택하고, `Verify New Domain`을 눌러 도메인을 등록한다. `Generate DKIM Settings`도 체크해서 DKIM 세팅을 생성할 수 있게 한다. DKIM은 `DomainKeys Identified Mail`의 약어로 SPF와 마찬가지로 이메일 발신자가 위조되었는지 확인하기 위한 방법이다.

![SES 01]({{ site.url }}/assets/images/Amazon-SES-Setup-00/01.png)  

다음 단계로 넘어가면 도메인 소유 검증 및 이메일 수신을 하기 위해 다음 레코드를 등록하라고 나타난다. 본인이 사용 중인 DNS 관리 서비스에서 정보를 설정한다.

![SES 02]({{ site.url }}/assets/images/Amazon-SES-Setup-00/02.png)  

DNS 세팅을 끝내고 잠시 후 도메인 리스트에서 새로 고침을 하면 모든 Status가 `verified`로 바뀐 것을 볼 수 있다.

## 이메일 수신 처리
아마존 SES는 `Rule Set` 정의를 통해 메일이 수신될 경우 처리를 제어할 수 있다. 이점을 활용하여 내 도메인의 이메일 주소로 메일이 수신될 경우, gmail로 자동 포워딩하는 세팅을 하려 한다. 꼭 gmail이 아니어도 되지만, 내가 사용하는 메일이 gmail이므로 그쪽으로 보낼 것이다. 이 과정은 [arithmetric/aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder)의 도움을 받을 것이므로 자세한 정보는 해당 레포를 참고 바란다.  
다음과 같은 룰을 만든다.

1. xxx@example.com으로 메일이 수신될 경우, 내 버킷의 특정 prefix에 이메일을 저장한다.
2. Lambda Action을 통해 저장된 email을 콘텐츠로 내 gmail 계정으로 송신한다.

### 세팅
1. [https://github.com/arithmetric/aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder) 의 코드를 내려받거나 클론한다.
2. `index.js`에 있는 `defaultConfig`값을 본인에게 알맞게 수정한다.  
    `fromEmail`: 내 gmail로 전달할 때 활용할 송신자 이메일 주소  
    `emailBucket`: 이메일을 저장할 S3 버킷 이름  
    `emailKeyPrefix`: S3 버킷에서 저장될 위치의 KeyPrefix
    `forwardMapping`: 어느 이메일로 메일이 오면 어디로 보낼지에 대한 매핑
3. AWS 람다에서 새로운 함수를 생성한다.
    `새로 작성`을 선택하고, 이름은 "SesForwarder"와 같이 본인이 알아볼 수 있게 적당히 지어준다. 런타임은 `Node.js 8.10`을 선택한다.  
    역할은 `사용자 지정 새 역할`을 선택하여 새로운 역할을 만든다. 마찬가지로 이름은 `LambdaSesForwarder`와 같이 본인이 적당히 알아볼 수 있게 적어준다.  
    정책은 다음의 코드를 복사하여 사용한다. 당연하지만 `S3-BUCKET-NAME`은 본인의 버킷 이름을 적어주면 된다.
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
         "Resource": "arn:aws:logs:*:*:*"
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
         "Resource": "arn:aws:s3:::S3-BUCKET-NAME/*"
      }
   ]
}
```
4. Lambda의 함수 코드에는 이전에 내려받고, `defaultConfig`를 수정한 `index.js`를 복사하여 붙여 넣는다. 핸들러가 `index.handler`로 설정되어있는지 확인한다.
   메모리는 128 MB 그대로 두고, 제한 시간은 10초로 넉넉하게 잡는다. 보통 작업은 30 MB 메모리와 몇 초 걸리지 않는다.
5. 다시 AWS SES로 돌아와서, 이메일을 전달받을 메일 주소(본인의 경우 Gmail 주소)를 인증한다. (만약 이미 Sandbox 제한이 해제되어있을 경우, 이 과정은 생략해도 된다.)  
    `Identity Management` - `Email Addresses` - `Verify a New Email Address`를 누르고, 이메일 주소를 입력한 다음, 인증 메일을 확인하면 바로 처리된다. 
6. 이제 AWS SES에서 `Email Receiving` - `Rule Sets`로 이동한다. 만들어둔 Rule Set이 있을 경우 해당 Rule Set으로, 없을 경우 새 Rule Set을 만든다. 그리고 `Create Rule`을 눌러 새 룰을 만든다. 
    ![SES 03]({{ site.url }}/assets/images/Amazon-SES-Setup-00/03.png)  
7. `Recipients`에서는 내가 포워딩시키고 싶은 앞서 등록한 도메인의 이메일 주소를 적는다. 여기에 적은 이메일 주소로 메일이 오면 Lambda 코드에 적어둔 주소로 보낸다.
    ![SES 04]({{ site.url }}/assets/images/Amazon-SES-Setup-00/04.png)  
8. 액션 설정 페이지에서, `S3` - `Lambda`순으로 액션을 만들고, 이전에 만들고 설정한 Bucket, Key Prefix, Lambda Function 이름을 설정한다. 
9. Rule 이름을 지어주고, `Enable spam and virus scanning`은 꼭 체크한다.
    만약 SES에서 `lambda:InvokeFunction` 권한 추가를 요청하면, 허용한다.
    만약 "Could not write to bucket"이 나오면, IAM 사용자가 S3 버킷에 읽기, 쓰기 권한이 없다는 뜻이다. `S3 버킷` - `권한` - `버킷 정책`에서 Bucket 정책 설정을 다음과 같이 해준다. `AWS-ACCOUNT-ID`은 AWS에서 `내 계정` - `계정 ID`에 나온다.
```json
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Sid": "GiveSESPermissionToWriteEmail",
         "Effect": "Allow",
         "Principal": {
            "Service": "ses.amazonaws.com"
         },
         "Action": "s3:PutObject",
         "Resource": "arn:aws:s3:::S3-BUCKET-NAME/*",
         "Condition": {
            "StringEquals": {
               "aws:Referer": "AWS-ACCOUNT-ID"
            }
         }
      }
   ]
}
```
10. 추가적으로, S3 lifecycle 정책을 설정하여 버킷에 저장된 이메일을 일정 기간이 지나면 삭제하도록 할 수도 있다.

모든 설정이 끝난 다음, 다른 이메일에서 위에서 설정한 내 도메인을 사용하는 이메일 주소로 테스트 메일을 전송하면 중계가 정상적으로 이루어지는 것을 확인할 수 있다.

![SES 05]({{ site.url }}/assets/images/Amazon-SES-Setup-00/05.png)  

여담으로, 이메일 포워딩도 결국 SES, S3, Lambda 같은 아마존 자원을 사용하기에, 비용이 많이 발생하지 않을까 걱정했는데, Github Issue 글을 보니 4만 8천건당 1달러 정도로 아주 작아 신경 쓰지 않아도 될 정도라고 한다. ([Link](https://github.com/arithmetric/aws-lambda-ses-forwarder/issues/68#issuecomment-418201013))

## 이메일 발송 처리
이메일 발송 방법은 크게 두 가지가 있다. Gmail 같은 상용 메일 서비스에 중계 서버를 등록해서 전송하는 방법, AWS SDK를 활용해서 서버 등 자신이 만들고 있는 서비스에서 활용하는 방법. 두 가지 모두 순서대로 설명하겠다.
**참고로 AWS SES는 Sandbox 제한 해제 요청을 따로 하지 않을 경우, `Identity Management` - `Email Addresses`에 인증해둔 이메일 주소로만 메일을 전송할 수 있다.** 타인에게 이메일을 전송하려면, Sandbox 제한 해제 요청을 꼭 해야 한다.


### Gmail에서 내 도메인 메일 전송하기
앞서 Gmail로 중계하도록 처리했으니, Gmail에서 내 도메인 메일 주소를 발신자로 설정하여 메일을 전송하는 방법을 설명한다.

1. 먼저, 발신에 사용할 이메일 주소를 인증해야 한다. AWS SES - `Identity Management` - `Email Addresses` - `Verify a New Email Address`에서 발신에 사용할 내 도메인 이메일 주소를 추가한다. 내 도메인 이메일 주소로 인증 메일이 전송된다. Gmail로 중계 처리되어있으니 문제없이 인증을 해낸다.
2. AWS SES - `Email Sending` - `SMTP Settings` - `Create My SMTP Credentials`를 눌러 SMT 인증용 게정을 생성한다. Credentials 파일을 다운로드한 다음 안전한 곳에 저장한다.
3. Gmail에서 설정 - `계정 및 가져오기` - `다른 주소에서 메일 보내기` - `다른 이메일 주소 추가`를 클릭한다. 이름은 메일 발송 시 타인에게 표시될 기본 이름을 작성하고, 등록할 내 도메인 이메일 주소를 입력한다.
    ![SES 06]({{ site.url }}/assets/images/Amazon-SES-Setup-00/06.png)  
4. SMTP 서버, 사용자 이름, 비밀번호는 모두 Credentials.csv에 있는 그대로 입력한다.  
    그리고 계정 추가 버튼을 누르면 인증 메일이 발송되는데, 메일이 도착하는 대로 메일에 있는 인증 코드를 입력하거나 인증 링크를 누르면 모든 설정이 끝난다.
    ![SES 07]({{ site.url }}/assets/images/Amazon-SES-Setup-00/07.png)  
5. 모든 과정이 끝나고 `편지 쓰기`를 눌러보면 `보낸 사람`을 내가 추가한 메일 주소로 할 수 있는 것을 확인할 수 있다.

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
let nodemailer = require('nodemailer');
let AWS = require('aws-sdk');
AWS.config.loadFromPath(__dirname+'/aws_config.json');

let transporter = nodemailer.createTransport({
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
[발송 메일을 스팸으로 분류되지 않도록 개선하는 법 - 아사마루](https://blog.asamaru.net/2015/12/16/how-can-i-prevent-mail-from-being-classified-as-spam/)  
[White Domain 소개 - KISA 불법스팸대응센터](https://www.kisarbl.or.kr/whiteip/whiteip_tutorial.jsp)  
[github.com/arithmetric/aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder)