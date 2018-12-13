---
title: "슬랙 날씨 봇 만들기"
date: 2018-05-20 18:21:16 +0900
categories: NodeJS
layout: post
comments: true
pinned: true
---

## 00. 준비
슬랙 메신저를 통해 날씨를 알려주는 봇을 만들어보자.  
목표는 매일 일정 시간에 날씨를 알려주고, 명령어를 통해 날씨 정보를 확인할 수 있는 봇을 개발하는 것이다.

먼저 봇을 동작하게 할 토큰을 생성해보자.  
슬랙 상단에서 Workspace 이름을 클릭, Administaration-Manage Apps-Custom Integrations-Bots 페이지로 들어간다.  
**Add Configuration**을 눌러 생성한다.  
![Add Configuration]({{ site.url }}/assets/images/Slack-Weater-Bot/00.AddConfiguration.png)  

그리고, 여기서 API Token을 얻을 수 있다.
![API Token]({{ site.url }}/assets/images/Slack-Weater-Bot/01.APIToken.png)  


## 01. 코드 작성
{% gist 249a3e2765953b5d1dcc696c106de5ee %}

좋은 라이브러리들이 많이 존재해서 코드는 단 100줄도 되질 않는다.  
`node-schedule`를 활용해서 주기적으로 채널에 날씨정보 메세지를 보내도록 했다. 그리고 `@slack/client`로 슬랙 API와 연동을 시켰고, `weather-js`로 날씨 정보를 가져온다.

코드를 단락별로 분석해보자.  

```javascript
var schedule = require('node-schedule');
const { RTMClient, WebClient } = require('@slack/client');
var weather = require('weather-js');
const token = '<#YOUR-SLACK-TOKEN#>';

const rtm = new RTMClient(token);
const web = new WebClient(token);
rtm.start();
```

모듈들을 로드한다. `@Slack/client`는 `RTMClient`와 `WebClient`로 나뉜다.  
`RTMClient`는 Real Time Message Client의 약자로 이름 그대로 실시간으로 오가는 메세지를 이벤트로 캐치하여 활용할 수 있다. `WebClient`는 파일 업로드를 하는 것 처럼 `RTMClient`에서 못하는 기능의 대용으로 사용한다. 여기서는 채널 리스트를 가져오는 용도로 사용할 것이다.

```javascript
var sendMessage = function(text, ch) {
	if (Array.isArray(ch)) {
		for (var i = 0; i < ch.length; i++) {
			rtm.sendMessage(text, ch[i])
				.then((msg) => console.log(`Message sent to channel ${ch}`))
				.catch(console.error);
		}
	}else {
		rtm.sendMessage(text, ch)
				.then((msg) => console.log(`Message sent to channel ${ch}`))
				.catch(console.error);
	}
}
```

특정 채널에 메세지를 전송하는 것을 편하게 쓰고싶어 함수로 정의했다. 여러 채널로 메세지를 전송하고 싶을 때는 채널을 배열로 넘겨준다.

```javascript
var sendWeatherMessage = function(location, channel){
	weather.find({search: location, degreeType: 'C'}, function(err, result) {
		if(err) {
			console.log(err);
			sendMessage('에러! '+err, channel);
			return;
		}
		if (!result.length) {
			sendMessage('정보가 없습니다.', channel);
			return;
		}

		var now = new Date();
		var tz = now.getTime() + (now.getTimezoneOffset() * 60000) + (9 * 3600000);
		now.setTime(tz);

		var weekToday = now.toLocaleString('en-us', { weekday: 'short' });
		var weekTommorow = new Date(now.getTime() + 24 * 60 * 60 * 1000).toLocaleString('en-us', { weekday: 'short' });
		var forecastToday = { low:'', high:'', skytextday:''};
		var forecastTomorrow = { low:'', high:'', skytextday:''};
		for (var i = 0; i < result[0].forecast.length; i++) {
			if (result[0].forecast[i].shortday == weekToday) {
				forecastToday = result[0].forecast[i];
			}else if (result[0].forecast[i].shortday == weekTommorow) {
				forecastTomorrow = result[0].forecast[i];
			}
		}

var text = result[0].location.name+'을 기반으로 한 정보입니다.\n\
현재 기온: '+result[0].current.temperature+'\'C, '+result[0].current.skytext+'\n\
오늘예상: '+forecastToday.skytextday+', 최저 '+forecastToday.low +'\'C, 최고 '+forecastToday.high+'\'C\n\
내일예상: '+forecastToday.skytextday+', 최저 '+forecastTomorrow.low +'\'C, 최고 '+forecastTomorrow.high+'\'C\n';
		sendMessage(text, channel);
	});
};
```
날씨 정보 메세지를 보내기 위한 메소드다.  
`weather.find()`로 원하는 지역의 날씨 정보를 받아온다. 실패할 경우 실패 로그를 슬랙으로 보낸다.  
성공할 경우, 내가 원하는 일자(오늘과 내일)의 날씨 정보만 추려낸다. 그리고 슬랙으로 보내기 위한 메세지 포맷을 짜고 메세지를 전송한다.

```javascript
rtm.on('message', (message) => {
	if (message.subtype || (!message.subtype && message.user === rtm.activeUserId)) {
		return;
	}
	console.log(message);
	// Log the message
	console.log(`(channel:${message.channel}) ${message.user} says: ${message.text}`);
	if (message.text.includes('!날씨 ')) {
		var location = message.text.split('!날씨 ')[1];
		if (!location.length) {
			return;
		}
		sendWeatherMessage(location, message.channel);
	}
});
```
슬랙에서 사용자가 날씨 정보를 요청할 경우, 응답하기 위한 이벤트 리스너다. 사용자가 `!날씨 {지역명}` 형태로 정보를 요청하면, 캐치하고 해당 지역의 날씨 정보 메세지를 전송한다.

```javascript
var scheduleRule = new schedule.RecurrenceRule();
scheduleRule.hour = 21;
scheduleRule.minute = 0;
var scheduleJob = schedule.scheduleJob(scheduleRule, function(){
	console.log('scheduleJob');
	web.channels.list().then((res) => {
	  // Take any channel for which the bot is a member
	  var channels = [];
	  for (var i = 0; i < res.channels.length; i++) {
		  if (res.channels[i].is_member) {
			  channels.push(res.channels[i].id);
		  }
	  }
	  if (!channels) {
		  console.log('가입된 채널이 없습니다.');
		  return;
	  }
	  sendWeatherMessage('Seoul', channels);
  });
});
```
매일 아침 6시에 날씨 정보를 날씨봇이 초대된 채널에 보내기 위한 ScheduleJob이다.
시간은 GMT 기준이므로, 전송을 원하는 한국 시간 -9시간을 지정한다. 채널 정보를 가져오고, 날씨봇이 초대된 방이면 메세지를 전송한다.

## 02. Deploy
로컬에서 돌려도 되고, 자신이 사용하는 서버가 있다면 그곳에서 동작시켜도 된다.  
개인적으로는 날씨봇 하나를 위해 서버를 호스팅 받기엔 아깝고, 24시간 켜두는 컴퓨터도 없다보니 Heroku를 활용했다.  
굉장히 작은 규모의 앱에 IO도 작고 중요도도 작으므로 Node 무료 호스팅 정도로도 충분하다고 생각했다.

Heroku 호스팅 방법은 [devcenter.heroku.com](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction)에 매우 친절하게 적혀져 있으므로 참고바란다. 정말 자세하게 적어둬서 누구나 손쉽게 따라할 수 있을 것이다.  

![Slack Weater Bot]({{ site.url }}/assets/images/Slack-Weater-Bot/02.SlackWeaterBot.png)  

100줄도 안되는 코드로 금방 봇을 만들 수 있었다.