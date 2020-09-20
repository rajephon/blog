---
title: "Node.js 서버에 Google OAuth 2.0 로그인 손쉽게 만들기"
date: 2017-11-28 01:27:54 +0900
categories: NodeJS
layout: post
comments: true

---

구글 계정을 이용한  로그인 구현은 예전에 비하여 정말 손쉬워졌다.
Firebase를 이용하거나, Google Sign-In for Websites를 순차적으로 따라하는 것만으로도 쉽게 만들 수 있다.
본인은 Passport.js라이브러리가 마음에 들어서 그것을 활용해보려고 한다.

# 개발환경
AWS EC2 인스턴스에 express 프로젝트를 구성해둔 상태에서 작업한다.

**Ubuntu 16.04.3 LTS**, **Node v4.2.6**
```bash
npm install --save body-parser express express-session
```

# OAuth 2.0 클라이언트 ID 발급
Google OAuth 2.0을 이용하려면 클라이언트 ID를 발급받아야 한다. 발급은 [Google APIs Console](https://console.developers.google.com/apis)에서 받을 수 있다. 프로젝트가 없으면 생성하고, `OAuth 2.0 클라이언트 ID`항목이 비어있다면 `사용자 인증 정보 만들기`-`OAuth 2.0 클라이언트 ID`를선택하여 생성한다. 그리고 별도의 사용 도메인이 있을 경우 생성한 클라이언트 ID 설정화면에서 URI에 도메인을 입력하고, 리디렉션 URI를 세팅한다. 본인은 리디렉션 경로로 `/auth/google/callback`을 사용할 예정이므로 https://mydomain/auth/google/callback 과 같이 세팅하였다.


![screenshot1]({{ site.url }}/assets/images/Nodejs-Google-OAuth-2-0/screenshot1.jpg)  

# Passport.js 세팅
먼저 `npm install` 명령어를 이용하여 **passport** 라이브러리와 **passport-google-oauth20** 라이브러리를 설치한다. **passport-google-oauth2** 라는 이름의 라이브러리도 존재하는데, 라이브러리 문서를 참고하여 확인하고 자신의 입맛에 맞는 라이브러리를 선택하여 사용하면 된다. 여기서는 **passport-google-oauth20**를 사용하고 그것에 맞춰 설명할 것이다.

```bash
npm install --save passport@0.4.0 passport-google-oauth20@1.0.0
```

그리고 노드에서 기본 세팅을 진행한다.

**GOOGLE_CLIENT_ID**와 **GOOGLE_CLIENT_SECRET**는 상단 진행과정을 통해 발급받은 정보를 입력하고, CALLBACK_URL또한 사전에 구글 API발급을 받을 때 등록해둔 경로로 해야한다.

```javascript
const passport = require('passport');
var GoogleStrategy = require( 'passport-google-oauth20' ).Strategy

passport.serializeUser(function(user, done) {
    done(null, user);
});

passport.deserializeUser(function(obj, done) {
    done(null, obj);
});

passport.use(new GoogleStrategy({
        clientID: 'GOOGLE_CLIENT_ID',
        clientSecret: 'GOOGLE_CLIENT_SECRET',
        callbackURL: 'https://mydomain/auth/google/callback'
    }, function(accessToken, refreshToken, profile, done) {
        process.nextTick(function() {
            user = profile;
            return done(null, user);
        });
    }
));
```

passport는 사전에 Strategy(전략)을 세팅해두고, 인증 요청이 들어올 경우 세팅해둔 인증 프로토콜에 따라 처리하고, 결과를 Strategy callback으로 반환한다. Strategy callback에서 받은 profile은 DB에 기록하거나, DB에 있는 사용자 데이터를 검색하여 처리하는 등의 각자의 서버에 알맞게 세팅할 수 있다.

# Express app 세팅
```javascript
app.get('/auth/google', passport.authenticate('google', { scope:
    ['profile']}));

app.get('/auth/google/callback', passport.authenticate( 'google', { failureRedirect: '/login' }),
    function(req, res) {
            res.redirect('/'); 
});
```
먼저, 임의의 경로로 접속 시 passport에 google 인증 요청을 전달하도록 한다. 본인은 임의로 '/auth/google'라는 경로로 접속시에 시도하도록 세팅하였다.

그리고, google api를 등록할 때 설정해둔 콜백 경로 또한 세팅한다.

```javascript
passport.serializeUser(function(user, done) {
    done(null, user);
});
passport.deserializeUser(function(obj, done) {
    done(null, obj);
});
```
serializeUser는 사용자가 인증을 성공할 경우, deserializeUser는 이후 사용자의 요청 시마다 호출된다.

passport는 google 이외에도 facebook, twitter, github, local login 등 다양한 로그인 처리를 손쉽게 이룰 수 있다. 각 기업의 서비스마다 비슷하지만 다른 인증 방식으로 인하여 다소 번거로운 과정또한 손쉽게 만들어줘 정말 유익하다.