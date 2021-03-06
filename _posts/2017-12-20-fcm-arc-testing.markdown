---
layout: post
title:  Advanced REST client를 이용한 안드로이드 푸시 테스트
date:   2017-12-20
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: /01_FCM/intro.png
categories: Android
tags: [Blog, Android, FCM, ARC]
author: 우주
---

FCM을 이용해 안드로이드 푸시 앱을 구현했다면 이제 테스트를 해 볼 차례입니다. 바로 Firebase Console에서 메시지를 보내 테스트를 할 수도 있습니다.

<br><br><br>![firebase_console_noti]({{site.baseurl}}/assets/img/02_ARC/firebase_console_noti.png)

> 이렇게 푸시 테스트를 할 수도 있습니다.

<br><br>

하지만 실제 앱에서는 어쨌든 서버를 통해 푸시를 받게될 경우가 많을 것이기 때문에 (서버는 못 만들겠고..)<br> 
Advanced REST client(ARC)를 이용해 푸시 테스트를 한 번 해보겠습니다.<br>
(본 테스트는 ARC가 설치되어 있다는 전제하에 진행합니다. +굳이 ARC만 쓸 이유는 없고 Postman 이라던가 다른 REST client를 사용하셔도 됩니다.)
<br><br><br><br><br>

1_ARC 세팅
<br><br>
![arc_01]({{site.baseurl}}/assets/img/02_ARC/arc_01.png)

왼쪽에서 HTTP request를 선택하고 Method는 POST, Request URL은 [https://fcm.googleapis.com/fcm/send]를 입력해줍니다.
<br><br><br><br><br>

2_ARC Header 세팅
<br><br>
이제 헤더 부분을 세팅해주어야 합니다. 헤더는 name과 value로 구성되어 있고 우리는 여기서 두 쌍의 헤더를 추가해줘야 합니다.<br>

1. Header name : content-type, Header value : application/json
2. Header name : authorization, Header value : key=your device token<br>

여기서 중요한 부분이 2번의 Header value 입니다. "key=" 이 뒷부분에 우리가 만들었던 Firebase 프로젝트의 서버 키를 넣어주어야 합니다.
<br><br><br><br><br>

3_Firebase 서버 키 얻기

다시 [Firebase Console]로 이동합니다.

<br><br>
![firebase_console_01]({{site.baseurl}}/assets/img/02_ARC/firebase_console_01.png)

Firebase Console에서 이전에 만들었던 자신의 프로젝트를 클릭합니다.

<br><br><br>![firebase_console_02]({{site.baseurl}}/assets/img/02_ARC/firebase_console_02.png)

여기서 프로젝트 설정으로 들어갑니다.

<br><br><br>![firebase_console_03]({{site.baseurl}}/assets/img/02_ARC/firebase_console_03.png)

두 번째 클라우드 메시징 탭에서 서버 키를 발견할 수 있습니다. 이 서버 키를 복사해서 ARC 두 번째 Header value에 넣어주도록 합니다.
<br><br><br><br><br>

4_ARC Body 세팅
<br><br>
![arc_02]({{site.baseurl}}/assets/img/02_ARC/arc_02.png)

{% highlight java %}
{
    "to": "Your Device Token",
    "data": {
        "title":"My Push Test",
        "content":"Test Message"
    }
}
{% endhighlight %}
"to": 부분에는 안드로이드 앱에서 얻은 토큰값을 넣어줍니다. (우리가 만든 앱에만 푸시를 날릴 것이기 때문) <br>

이렇게 설정하면 MyFirebaseMessagingService의 onMessageReceived()에서 아래처럼 데이터를 받을 수 있게 됩니다.<br>
{% highlight java %}
    Map<String, String> data = remoteMessage.getData();
    String title = data.get("title");
    String messagae = data.get("content");
{% endhighlight %}

<br><br>

설정이 끝난 후 SEND를 눌러보면 정상적으로 푸시를 수신하여 앱의 포그라운드/백그라운드에 상관없이 동일한 노티피케이션이 생성되는 것을 볼 수 있습니다.

<br><br><br>'Advanced REST client를 이용한 안드로이드 푸시 테스트'는 이상으로 마칩니다. 감사합니다.



<br><br><br><br>
안드로이드 푸시앱 소스는 [이곳]에서 확인하실 수 있습니다.

참고 : [Firebase 가이드]

[https://fcm.googleapis.com/fcm/send]: https://fcm.googleapis.com/fcm/send
[Firebase 가이드]: https://firebase.google.com/docs/cloud-messaging/android/client?hl=ko
[Firebase Console]: https://console.firebase.google.com/
[이곳]: https://github.com/stonybean/MyPushTest
