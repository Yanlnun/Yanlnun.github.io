---
title: Twitter(X) API - 기본정보조회
author: YALN
date: 2024-11-02 00:34:00 +0800
categories: [API, Twitter(X)]
tags: [API, Twitter, NodeJS, VS Code, IntelliJ, HTML, JS]
---
 
## _For What?_
___
&ensp; 간단하게 입력받은 개인 혹은 단체의 트윗계정의 최근 트윗에 대한 리트윗 수를 구하고싶다. 

![img](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/1.png)

&ensp; Twitter가 X.com 으로 ~~타락~~한 뒤, URL 부분에선 채널명만 알 수 있을뿐 더이상 USERID 값( 숫자로만 이뤄진 ID)을 찾아볼순없다. 물론 개발자도구(F12)에서 Element -> Ctrl+F -> "ident" 로 검색 시 손쉽게 ID 넘버값을 확인 할 수 있지만, 그래도 모처럼 유료API 계정을 경험 할 수 있게 됐으니 한번 API요청으로 이것까지 받아와보자.  

![img](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/2.png)

&ensp; Twitter API 는 기본적으로 트윗ID ( 넘버 아이디 )값을 기준으로 이뤄지기 때문에 API 활용함에 꼭 필요한 최소한의 정보이다. 

<br>
<br>
<br>

## API 사용법 
___
&ensp; 조회만 할거라면 기본적인 형태는 다음과 같다. 
```js
const BEARER_TOKEN = ""; //TwitterAPI 앱등록시 발급받는 BEARER_TOKEN 기입
const APIurl = "" // 사용하려는 Twitter API
fetch(`https://api.twitter.com/${APIurl}`, {
  method: "GET",
  headers: {
    Authorization: `Bearer ${BEARER_TOKEN}`,
  },
})
  .then((response) => response.json())
  .then((data) => {
    
    // API로 부터 받은 data 가공
    
  })
    .catch((error) => console.error("Error:", error));
```

<br>
<br>
<br>

## STEP1 - 트윗계정 조회 : /2/users/by/username/${userName}
___
<br>

트위터 이름 가져와 해당 채널의 ID 넘버값을 확인해보자 
현 시각 기준 국민게임 롤. 대회중이니 롤스포츠 채널로 테스트해보겠다.

```js
const BEARER_TOKEN = ""; //TwitterAPI 앱등록시 발급받는 BEARER_TOKEN 기입
const username = "lolesports"; // 확인하려는 계정의 사용자 이름
let userId = ""; //api로 부터 받을 lolesports 의 Id 값.

fetch(`https://api.twitter.com/2/users/by/username/${username}`, {
  method: "GET",
  headers: {
    Authorization: `Bearer ${BEARER_TOKEN}`,
  },
})
  .then((response) => response.json())
  .then((userData) => {
    userId = userData.data.id;
    console.log( userData.data); // API호출로 받을수있는 data 값들 
    console.log("id : " + userId);
  })
  .catch((error) => console.error("Error:", error));
```

![img](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/3.png)

확인해보니 전체적으로는 id, name, username 값들을 받을수있고, 지금 필요한 id값만 받아주고 실제로 맞는지 확인해보자.

![img](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/4.png)

정확하게 맞아 떨어지는것을 확인했다면, 다음 스텝으로 가보자.

<br>
<br>
<br>

## STEP2 - 해당 계정의 최신 트윗(게시글) 조회 : /2/users/${userId}/tweets 
___
<br>

우선 트위터의 게시글(트윗)은 각 트윗마다 포스트넘버(id)가 붙게되고, API에 특정 트윗의 정보를 요청할때 이 ID값이 필요하게 되므로 위 코드에서 다음과 같이 수정 및 추가 해준다.
```js
const BEARER_TOKEN = ""; //TwitterAPI 앱등록시 발급받는 BEARER_TOKEN 기입
const username = "lolesports"; // 확인하려는 계정의 사용자 이름
let userId = ""; //api로 부터 받을 lolesports 의 Id 값.
let tweetId = ""; // api로 부터 받을 lolesports의 트윗(게시글) Id 값.

fetch(`https://api.twitter.com/2/users/by/username/${username}`, {
  method: "GET",
  headers: {
    Authorization: `Bearer ${BEARER_TOKEN}`,
  },
})
  .then((response) => response.json())
  .then((userData) => {
    const userId = userData.data.id;
    console.log(userData.data); // API호출로 받을수있는 data 값들 
    console.log("id : " + userId);
    // tweetId값을 요청할 API호출문 추가.
    return fetch(`https://api.twitter.com/2/users/${userId}/tweets`, {
      method: "GET",
      headers: {
        Authorization: `Bearer ${BEARER_TOKEN}`,
      },
    });
  })
  .then((response) => response.json())
  .then((tweetData) => {
    tweetId = tweetData.data[0].id;
    console.log("tweetData:", tweetData.data); //API호출로 받을수있는 data값들.
    console.log("tweetId:", tweetId);
  })
  .catch((error) => console.error("Error:", error));
```
![실행결과](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/5.png)

받아올수있는 정보는 최근 10개의 트윗과 각 트윗의 텍스트내용 및 트윗id 값들로 확인되고, 기본적으로 최신순으로 정렬되는듯하여 data[0]으로 접근했는데 meta 쪽에서 newest_id 값을 직접 가져와도 될것같다.

여기서 한번 눈여겨 봐야할건, 우리가 필요하건 가장 최근 트윗에 대한 정보인데 자동으로 10개의 트윗 정보가 가져와진다는것이다. 불필요한 데이터를 보내고 받는것은 뭐랄까 썩 기분이 좋지 않기에 API 문서를 뒤져봤지만, 명쾌한 해답은 없었다.

그나마 최선이라 볼수있는 방법은 url 파라미터값으로 /tweets?max_results=5 를 추가해주는것이다. 이렇게하면 가져와지는 트윗은 5개로 제한되는데, 공식적으로 지정한 최솟값은 5라고한다.. 코드를 수정하고 실행하면 다음과 같게 나온다.

```js
// 생략
return fetch(`https://api.twitter.com/2/users/${userId}/tweets?max_results=5`, {
// 생략
})
// 생략

```
![실행결과](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/6.png)
<br>
<br>
<br>

## STEP3 - 해당 트윗(게시글)의 정보조회 : /2/tweets?ids=${tweetId}
___
<br>

```js
const BEARER_TOKEN = ""; //TwitterAPI 앱등록시 발급받는 BEARER_TOKEN 기입
const username = "lolesports"; // 확인하려는 계정의 사용자 이름
let userId = ""; //api로 부터 받을 lolesports 의 Id 값.
let tweetId = ""; // api로 부터 받을 lolesports의 트윗(게시글) Id 값.

fetch(`https://api.twitter.com/2/users/by/username/${username}`, {
  method: "GET",
  headers: {
    Authorization: `Bearer ${BEARER_TOKEN}`,
  },
})
  .then((response) => response.json())
  .then((userData) => {
    userId = userData.data.id;
    console.log( userData.data); // API호출로 받을수있는 data 값들 
    console.log("id : " + userId);
    // tweetId값을 요청할 API호출문 추가.
    return fetch(`https://api.twitter.com/2/users/${userId}/tweets`, {
      method: "GET",
      headers: {
        Authorization: `Bearer ${BEARER_TOKEN}`,
      },
    });
  })
  .then((response) => response.json())
  .then((tweetData) => {
    tweetId = tweetData.data[0].id;
    console.log("tweetData:", tweetData.data); //API호출로 받을수있는 data값들.
    console.log("tweetId:", tweetId);

    // 최근tweet의 정보제공을 요청할 API호출문 추가.
    return fetch(`https://api.twitter.com/2/tweets?ids=${tweetId}&tweet.fields=public_metrics`, {
      method: "GET",
      headers: {
        Authorization: `Bearer ${BEARER_TOKEN}`,
      },
    });
  })
  .then((response) => response.json())
  .then((tweetInfo) => {
    console.log("tweetInfo", tweetInfo.data);
  })
  .catch((error) => console.error("Error:", error));
```
![실행결과](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/7.png)

추가된 API 호출문을 보면 파라미터값으로 ids와 tweet.fields 값을 같이 보냈는데, 중요한건 tweet.fields 이다.

API url 앞부분은 그냥 트윗의 기본정보를 조회하는 부분이고, 저 부분은 해당필드를 추가해서 보여달라는 부분인데 public_metrics라는 필드가 바로 찾고자했던 좋아요, 리트윗 수 등 을 확인할수있는 부분이다. 

> bookmark_count : 북마크된 수 <br>
> impression_count : view 수 <br>
> like_count : 좋아요 수<br>
> quote_count : 인용된 수 ★<br>
> reply_count : 댓글 수 <br>
> retweet_count : 리트윗 수 ★ <br>

 
> 트위터에 접속해서 보는 리트윗 수 = retweet + quote ( 순수 리트윗 수 + 인용된 수 )
{: .prompt-tip }

<br>
<br>
<br>

## 정리
___
<br>

사실 [STEP3](#step3---해당-트윗게시글의-정보조회--2tweetsidstweetid)에서 알게된 tweet.fields 파라미터는 [STEP2](#step2---해당-계정의-최신-트윗게시글-조회--2usersuseridtweets-) 단계에서 붙여도 된다. 

```js
const BEARER_TOKEN = ""; //TwitterAPI 앱등록시 발급받는 BEARER_TOKEN 기입
const username = "lolesports"; // 확인하려는 계정의 사용자 이름
let userId = ""; //api로 부터 받을 lolesports 의 Id 값.
// let tweetId = ""; // api로 부터 받을 lolesports의 트윗(게시글) Id 값. - 필요없어짐

fetch(`https://api.twitter.com/2/users/by/username/${username}`, {
  method: "GET",
  headers: {
    Authorization: `Bearer ${BEARER_TOKEN}`,
  },
})
  .then((response) => response.json())
  .then((userData) => {
    const userId = userData.data.id;
    console.log(userData.data); // API호출로 받을수있는 data 값들 
    console.log("id : " + userId);
    // tweetId값을 요청할 API호출문 추가.
    return fetch(`https://api.twitter.com/2/users/${userId}/tweets?tweet.fields=public_metrics`, {
      method: "GET",
      headers: {
        Authorization: `Bearer ${BEARER_TOKEN}`,
      },
    });
  })
  .then((response) => response.json())
  .then((tweetData) => {
    tweetId = tweetData.data[0].id;
    console.log("tweetData:", tweetData.data); //API호출로 받을수있는 data값들.
    // console.log("tweetId:", tweetId); // 필요없어짐
  })
  .catch((error) => console.error("Error:", error));
```
![실행결과](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/8.png)

이렇게 하면 API 요청도 적어지고, 코드도 짧아지니 실제로 써야한다면 이게 더 좋을것같긴하다. 물론 목적에 따라 달라질수도 있겠지만.. 

이상 트위터API - 기본정보조회 편은 테스트하면서 겪었던 문제들 나열한뒤 마치도록 하겠다.

<br>
<br>
<br>

## 발생했던 문제들
___
<br>

> CASE1 - Too Many Request
{: .prompt-danger }

![실행결과](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/9.png)

이게 제일 곤란했다. 트위터API는 과부화 방지용으로 일정시간동안 제한된 횟수만큼 API 보내게끔 설계되어 있다고한다. 그래서 수정-> 코드실행(API재요청) 과정이 반복될경우 "Too Many Request" 라는 오류가 콘솔창에 출력되며, data가 제공되지않는다... 

이것 또한 명쾌한 해결책이 없고, 그냥 15분정도 다른거 하다오면 알아서 풀려있다고 하더라. ㅎㅎ.. ~~망할 트위터~~

<br>

> CASE2 - CORS policy
{: .prompt-danger }

![실행결과](assets/img/post_imgs/2024-11-02-TwitterAPI-Get-TweetInfo/10.png)

너무 유명한 CORS 문제. 트위터API라서 생긴 문제가 아니라 로컬환경에서 별다른 조치없이 API 쓰게되면 거의 반드시 겪게 되는 문제라 기재할까 했으나, 혹시나 필요한 사람이 있을경우 간단한 조치 방법이라도 알려주기위해 글을 남긴다. 

윈도우환경이라면 win+r ( 실행 ) 창에서 다음을 입력후 엔터
```shell
chrome.exe --user-data-dir="C:/Chrome dev session" --disable-web-security
```

맥 환경이라면 터미널에서 
```shell
open -n -a "Google Chrome" --args --user-data-dir="/tmp/Chrome_dev_session" --disable-web-security
```

흔한 말로는 크롬을 웹 보안을 무시한채 요청보내고 요청받겠다는 편법이다. 오로지 로컬환경에서 api 테스트 할때만 사용하길 바란다. 처음 말했듯이 '보안을 무시한채' 이기 때문에 테스트용이 아니라 일상용으로 쓴다면 위험한 일들이 꽤나 있으리라 감히 예상해본다.  
<br>




