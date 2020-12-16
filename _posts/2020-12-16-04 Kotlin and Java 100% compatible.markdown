---
layout: post
title:  Android| 안드로이드에서 코틀린과 자바는 어떻게 100% 호환되는가?
date:   2020-12-16
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: /04_KOTLIN_JAVA/shaking-hands.jpg
tags: [Android, Kotlin, Java, JVM, DVM]
author: 우주
---

<br><br>

구글은 지난 I/O 2017 KeyNote에서 코틀린을 안드로이드 앱의 공식 개발 언어로 추가했다고 발표했다.

이후 점점 더 많은 개발자들이 코틀린을 사용하기 시작했고, (Goodbye Java 👋🏻)

현재 [Stack Overflow](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-languages) 기준 2020년 개발자들에게 가장 사랑받은 언어 4위에 랭크되어 있다. (나도 열심히 배우고 쓰고 있다 🤯)

<br><br>

![rank_stackoverflow]({{site.baseurl}}/assets/img/04_KOTLIN_JAVA/rank_stackoverflow.png){: .center}

<center>
  <span style="font-size: small; color: grey">당당하게(?) 4위에 랭크된 코틀린</span>
</center>
<br><br>

자바와 코틀린. 완전히 다르게 생긴 것 같은 두 언어가 어떻게 서로 100% 호환이 가능한 것일까?

(100% 호환이 가능하다는 것은 같은 프로젝트 안에 자바와 코틀린 파일이 바로 옆에 있다면 서로가 서로를 불러서 쓸 수 있다는 것이다.)

<br><br>

설명하기에 앞서 먼저 코틀린이 만들어지게 된 배경을 알고 넘어가면 좋을 듯 하다.

<br><br>

지금은 많은 개발자들이 코틀린을 사용하고 있지만, 코틀린은 원래 판매를 위해 만들어진 것은 아니었다.

코틀린을 만든 곳은 **JetBrains** 인데, IntelliJ IDEA 같은 IDE(Integrated Development Environment)를 만드는 회사로 유명하다.

(Android Studio도 IntelliJ를 기반으로 구글과 협력해 만들어짐)

<br>

원래 이 JetBrains 사의 제품들이 70% 이상 자바로 작성되어 있었는데, 

자바를 사용한다는 것은 그만큼 쓸데없는 Boiler-plate 코드가 많아진다는 것이고, 시간이 갈수록 코드를 유지/보수하고 읽고 쓰기가 점점 더 어려워진다는 **고질적인 문제**가 있었다.

그래서 자바가 아닌 더 좋고 모던한 프로그래밍 언어로 바꾸고 싶었는데 그러기에는 이미 자바로 쓴 코드가 너무 많았고, 완전히 새로운 언어로 작성할 수는 없었다.

이런 상황에서 자바와 호환이 가능한 새로운 언어로 작성하면서, 동시에 이전 시스템을 관리할 수 있는 그런 언어가 필요했고, 그렇게 코틀린이 만들어지게 된 것이다. (자바가 어지간히 싫었나보다..)

<br><br><br><br>

# 1. Java와 Android

------

<br>

자바와 코틀린의 호환성을 이해하기 위해서는 먼저 자바 프로그램의 특징을 알아야한다.

<br><br>

![java_program]({{site.baseurl}}/assets/img/04_KOTLIN_JAVA/java_program.png){: .center}

<center>
  <span style="font-size: small; color: grey">자바 프로그램의 실행 과정 (compiler + interpreter)</span>
</center>
<br><br>


위 그림처럼 **java code(.java)**는 바로 기계어로 컴파일 되는 것이 아니라 먼저 **byte code(.class)**로 컴파일 된다. (Compilation)

그리고 이 byte code가 **JVM(Java Virtual Machine)**에 의해 의해 해석된다. (Interpretation)

<br>

애초에 구글은 안드로이드 앱 개발 언어로 자바를 선택했기 때문에 안드로이드는 이러한 자바의 특징을 그대로 가지고 있다.

하지만 모바일 기기 특성상 데스크탑에 비해 환경이 열악하기도 했고, 또 JVM의 라이센스 문제 등 여러가지 이유로 JVM이 아닌 모바일 환경에 최적화 된 가상머신 DVM(Dalvik Virtual Machine)을 가지고 출발하게 되었다. (물론 현재는 ART(Android Runtime)가 기본)

<br><br>

![dvm_jvm]({{site.baseurl}}/assets/img/04_KOTLIN_JAVA/dvm_jvm.png){: .center}

<center>
  <span style="font-size: small; color: grey">DVM & JVM</span>
</center>
<br><br>


아무튼 여기서는 JVM이냐, DVM이냐가 중요한 것은 아니고 안드로이드가 이런 자바의 특징을 가지고 있다는 사실이 중요하다. (나중에 이 둘의 비교글도 올려봐야겠다.)

결국 이런 특징 때문에 코틀린 또한 안드로이드에서 사용할 수 있게 되었고, 어떻게 그것이 가능한지는 안드로이드의 빌드 프로세스를 살펴보면 알 수 있다.

<br><br><br><br>

# 2. Android Build Process

------

<br>

먼저 안드로이드 개발자 공식 홈페이지에 나와있는 [빌드 프로세스](https://developer.android.com/studio/build?hl=ko#build-process)를 살펴보면 다음과 같다.

<br><br>

![build-process]({{site.baseurl}}/assets/img/04_KOTLIN_JAVA/build-process.png){: .center}

<center>
  <span style="font-size: small; color: grey">일반적인 Android 앱 모듈의 빌드 프로세스</span>
</center>
<br><br>

1. 컴파일러는 소스 코드를 DEX(Dalvik Executable) 파일로 변환하고 그 외 모든 것은 컴파일된 리소스로 변환합니다. 이 DEX 파일에는 Android 기기에서 실행되는 바이트 코드가 포함됩니다.

2. APK Packager는 DEX 파일과 컴파일된 리소스를 단일 APK로 결합합니다. 그러나, 앱을 Android 기기에 설치하고 배포할 수 있으려면 먼저 APK에 서명해야 합니다.

3. APK Packager는 디버그 또는 출시 키 저장소를 사용하여 APK에 서명합니다.

   a. 디버그 버전의 앱(즉, 테스트 및 프로파일링 전용 앱)을 빌드 중인 경우에는 패키저가 디버그 키 저장소로 앱에 서명합니다. Android 스튜디오는 디버그 키 저장소로 새 프로젝트를 자동으로 구성합니다.

   b. 출시 버전의 앱(즉, 외부에 출시할 앱)을 빌드 중인 경우에는 패키저가 출시 키 저장소로 앱에 서명합니다. 출시 키 저장소를 생성하려면 [Android 스튜디오의 앱 서명](https://developer.android.com/studio/publish/app-signing?hl=ko#studio)을 참조하세요.

4. 최종 APK를 생성하기 전에, 패키저는 앱이 기기에서 실행될 때 더 적은 메모리를 사용하도록 앱을 최적화하기 위해 [zipalign](https://developer.android.com/studio/command-line/zipalign?hl=ko) 도구를 사용합니다.

<br><br>

여기서 빌드 프로세스의 첫 번째 단계가 코틀린이 자바와 100% 호환이 가능하다는 것을 보여주는 부분이다.

<br>

앞서 말한 것처럼 자바 코드(.java)는 컴파일러에 의해 바이트 코드(.class)로 변환되는데, 코틀린 코드(.kt) 또한 마찬가지이다.

**안드로이드**에서 소스 코드는(자바 파일이든 코틀린 파일이든) 컴파일러에 의해 우선 **DEX 파일**로 변환되는데, 이 DEX 파일은 바이트 코드(.class 파일)들을 포함하고 있다.

<br><br>

![code_to_dex]({{site.baseurl}}/assets/img/04_KOTLIN_JAVA/code_to_dex.png){: .center}

<center>
  <span style="font-size: small; color: grey">Java(.java) / Kotlin(.kt) -> Byte Code(.class) -> DEX File(s)(.dex)</span>
</center>
<br><br>

<u>결국 컴파일되는 부분 있어서 자바와 코틀린 모두 바이트 코드(.class)로 컴파일되기 때문에</u> 서로 간에 100% 호환이 가능하게 되는 것이다.

그리고 DEX파일로 변환된 이후부터의 진행 과정은 모두 동일하기 때문에 따로 설명하지는 않겠다.

<br><br><br>

<h3>￭ 요약</h3>

1. Java code(.java)/Kotlin code(.kt) -> byte code(.class)로 컴파일 (= 상호 호환 가능)
2. .class파일들이 다시 DEX 파일로 변환
3. 이후 나머지 빌드 프로세스 동일

<br><br><br><br>

---

<br>

처음에 코틀린이 자바를 대체할 수 있다고 해서 그게 어떻게 가능한가 의문을 가졌었는데,

코틀린이 만들어지게 된 배경이나 여러가지 것들을 찾아보다보니 궁금증을 해소할 수 있었다.

또 이걸 찾아보면서 안드로이드의 빌드 프로세스, JVM, DVM, ART, JIT, AOT 등등 다양한 것들 또한 다시 한 번 학습하게 된 것 같다.

역시 하나를 알려고 하면 꼬리를 물고 다른 것들도 알아야 하는 IT의 학습 환경...(?) 🤯

<br><br><br>자바에 비해 확 줄어드는 코드의 양, 속도, 생산성 등 여러가지 면에서 더 좋은 성능을 보여주기 때문에 앞으로는 코틀린을 훨씬 많이 사용하게 될 것 같다. (구글에서도 자바보다는 코틀린을 확실히 밀고 있기도 하기에)

이제는 완전 대세 반열에 올라선 코틀린을 좀 더 깊이 있게 공부하고 제대로 사용해야겠다. 끝.

<br><br><br><br>

(* 틀린 부분이 있으면 댓글로 알려주세요.)

<br>

참고 : 

[https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-languages](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-languages)

[https://dev.to/jay_tillu/why-jetbrains-create-kotlin-the-inside-story-of-kotlin-creation-1135](https://dev.to/jay_tillu/why-jetbrains-create-kotlin-the-inside-story-of-kotlin-creation-1135)

[http://math.hws.edu/eck/cs124/javanotes6/c1/s3.html](http://math.hws.edu/eck/cs124/javanotes6/c1/s3.html)

<br><br>