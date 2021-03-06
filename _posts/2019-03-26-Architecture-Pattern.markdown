---
layout: post
title:  안드로이드 앱 아키텍처 패턴
date:   2019-03-26
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: /03_DESIGN_PATTERN/abstract-4022574_1920.jpg
categories: Android
tags: [Android, Architecture, DesignPattern]
author: 우주
---



회사 내부 세미나를 준비하면서 정리했던 아키텍처 패턴(혹은 디자인 패턴)에 대해 블로그로 남기기로 한다.



수많은 자료를 보고 여러 블로그들을 방문하면서 다양한 정보들을 찾을 수 있었지만 아키텍처 패턴이 왜 필요하며 어떻게 등장하게 되었는가에 대한 설명은 조금 부족하다고 느꼈다.



먼저 간단하게 그 부분에 대한 정리를 한 후에 각 패턴에 대한 설명을 하겠다.

<br><br><br><br><br>





# 1. 모바일 앱 아키텍처 개요

------



## 1-1 모바일 앱 사용자 환경
![01_camera_app_process]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/01_camera_app_process.png){: .center}

<center>
  <span style="font-size: small; color: grey">사진공유 앱에서 카메라로 사진을 찍은 뒤 공유하는 프로세스</span>
</center>

<br>

아키텍처 패턴을 살펴보기 전에 가장 먼저 모바일 앱 사용자 환경에 대한 이해가 필요하다. 사진을 공유하는 앱을 예로 들어보자. 앱을 통해 사진을 공유하는 프로세스는 다음과 같다.

<br>

1. 사진 공유 앱을 실행한다.
2. 사진 공유 앱 내에서 카메라앱을 실행한다.
3. 카메라 앱이 실행된다.
4. 사진을 찍은 뒤 사진 공유 앱으로 돌아와 사진을 개시(공유)한다.

<br>

단순 과정으로만 봤을 때 카메라 앱이 실행될 때 실제로는 사진 공유 앱은 중단된 상태이다. 하지만 사용자의 입장에서는 사진을 공유하는 작업 환경(사용자 사용 환경)이 중단된 것이 아니다. 심지어 카메라 앱이 실행 되어 있는 중에도 사용자는 다른 앱을 실행할 수가 있다.

<br><br>

### ￭ 사용 환경 중단

이처럼 모바일 환경에서는 전화, 알림 등에 의해 **사용 환경이 중단**되거나 **다른 앱으로 바꾸는 동작**이 일반적이다. 사용자는 사용 환경 중단에 대응하고 다시 앱으로 돌아가기를 기대하기 때문에 앱에서 이러한 흐름을 올바르게 처리할 필요가 있다.

<br>

### ￭ 휴대기기의 제한적 리소스

또한 스마트폰의 성능이 아무리 좋아졌다고 한들 **휴대기기의 리소스는 제한적**이다. 그렇기 때문에 운영체제에서 새로운 앱을 위한 공간 확보를 위해 언제든 앱 프로세스의 종료가 가능하다.

<br><br>

정리하자면, **앱 구성요소는 개별적이고 비순차적으로 실행이 가능**하고, **사용자나 운영체제에 의해 언제든지 제거**될 수 있다. 그렇기 때문에 앱 구성요소에 앱 데이터나 상태 등을 저장해서는 안되고 앱 구성요소는 서로 종속되면 안되는 것이다.<br>(* 여기서는 앱 구성요소를 액티비티/프래그먼트로 한정해서 생각해도 될 듯 하다.)

<br><br><br>

## 1-2 일반 아키텍처 원칙

### ￭ 관심사 분리 (Separate of Concerns)

안드로이드는 UI 기반 클래스(Activity, Fragment)에 코드를 작성한다. 이러한 클래스는 사실 안드로이드 OS와 앱 사이의 계약을 나타내도록 이어주는 역할에 불과한데 OS는 언제든지 이 클래스들을 제거할 수 있다. 사용자 환경과 수월한 앱 관리 환경을 위해 이러한 클래스의 의존성을 최소화하는 것이 필요하다.

<br>

### ￭ 모델에서 UI 만들기

모델이란 앱의 데이터 처리를 담당하는 구성요소로 View 개체와 앱 구성요소에 독립되어 앱 수명주기에 영향을 받지 않는다. 안드로이드 개발자 가이드에서는 다음과 같은 이유를 들어 지속 모델을 권장한다.

- OS에서 앱을 제거해도 사용자 데이터가 삭제되지 않는다.
- 네트워크 연결이 약하거나 연결되지 않아도 앱이 작동한다.

<br>

뒤에 다시 설명하겠지만 수명주기와 일반 아키텍처 원칙이 적용된 ViewModel(in MVVM)을 간단히 비교해보자.
<br>

![02_life_cycle_viewmodel]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/02_life_cycle_viewmodel.png){: .small-wrapper}{: .center}

<center>
  <span style="font-size: small; color: grey">수명주기와 ViewModel (그림 왼쪽은 액티비티의 생성, 화면 전환, 종료까지의 수명주기를 나타낸다.)</span>
</center>
<br>

뷰모델은 액티비티와 프래그먼트에서 사용되는 UI 관련 데이터를 보관, 관리하기 위해 디자인 되었는데, 그림처럼 액티비티가 여러번 재생성되는 상황에서도 **뷰모델은 인스턴스를 유지함으로써 데이터를 안전**하게 다룰 수 있다.
또한 데이터의 소유권을 액티비티와 프래그먼트로부터 분리시킴으로써 **관심사 분리도 가능해지고 앱 수명주기에도 영향을 받지 않는다.**

<br>

![03_basic_activity]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/03_basic_activity.png){: .center}

<center>
  <span style="font-size: small; color: grey">일반적 코드 작성법</span>
</center>
<br>

다음은 가장 일반적인 방식으로 안드로이드 코드를 작성한 것이다. 뷰, 이벤트 리스너, 데이터 처리 등 모든 것이 한 액티비티에 존재하는 것을 확인할 수 있다. 이렇게 코드를 작성하게 되면 시간이 지날수록 복잡, 비대해져 문제가 생길 수 있고, 유지보수에 어려움이 있을 수 있다. 또한 액티비티 특성상 수명주기에 따른 영향도 있을 것이며 데이터도 안전하게 다루지 못하게 된다.

<br>

결국, 모바일 앱 사용 환경, 안드로이드 클래스의 특징, 앱 수명주기의 영향, 코드의 복잡성 등 다양한 이유와 문제점을 바탕으로 좀 더 안전하고 깔끔한 개발을 위해 아키텍처 패턴이 생겨나게 된 것이다.

<br><br><br><br>

# 2. MVC

------



## 2-1 구조 및 동작

MVC 패턴에서 사용자 입력은 컨트롤러를 통해 들어오며 컨트롤러는 모델과 상호작용을 통해 뷰를 업데이트한다. 이때 뷰는 모델을 참조하게 된다. 이를 그림으로 나타내면 아래와 같다.

<br>

![04_mvc_pattern]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/04_mvc_pattern.png){: .center}

<center>
  <span style="font-size: small; color: grey">MVC 구조 및 동작</span>
</center>


<br><br><br>

## 2-2 특징

### ￭ Model

- 데이터 + 상태 + 비즈니스 로직
- 뷰나 컨트롤러에 묶이지 않아 재사용 가능

<br>

### ￭ View

- 사용자에게 제공되는 UI (모델의 표현)
- UI, 앱과의 상호작용에서 컨트롤러와 통신
- 사용자가 어떤 액션을 하든 무엇을 해야할지 모름

<br>

### ￭ Controller

- 앱을 묶어주는 접착제 (액티비티/프래그먼트)
- 사용자에게 입력을 받아 해당하는 모델 선택 후 처리
- 모델의 데이터 변화에 따라 뷰를 선택

<br><br>

우리가 일반적으로 알고 있는 MVC 패턴과 동일하지만 안드로이드는 UI 기반의 클래스(액티비티)에 코드를 작성하게 된다. 그렇기 때문에 액티비티가 뷰와 컨트롤러의 특성을 모두 가지게 되어 뷰와 컨트롤러의 구분이 애매하다는 점에서 차이가 있다.

<br><br><br>

## 2-3 장/단점

### ￭ 장점

- 모델과 뷰를 분리
- 모델의 비종속성으로 재사용 가능
- 구현하기 가장 쉬움

<br>

### ￭ 단점

- 컨트롤러가 뷰에 단단히 결합되며, 뷰의 확장일 수 있음
- 뷰의 화면 업데이트를 위해 모델을 직/간접적으로 참조(서로간의 의존성 완전히 없앨 수 없음)
- 시간이 지날수록 컨트롤러에 많은 코드가 쌓여 비대화, 문제 발생 가능

<br><br>

장점은 MVC 패턴을 적용해 모델과 뷰를 분리를 하게 되었다는 점이다. 모델은 다른 부분에 종속되지 않으므로 어디서든 재사용이 가능하다. 그리고 다른 패턴들에 비해 구현하기가 가장 쉽다는 것도 장점이다. 하지만 앞서 말한 것처럼 안드로이드는 컨트롤러와 뷰의 구분이 쉽지 않다. 그렇기 때문에 컨트롤러가 뷰에 단단히 결합되고 이는 뷰의 확장이 될 수도 있다. 또한 뷰의 화면 업데이트를 위해서 모델을 직/간접적으로 참조하기 때문에 서로 간의 의존성이 완전히 사라진 것도 아니다. 모델을 제외한 한 컨트롤러(액티비티) 안에 코드를 작성하게 되므로 시간이 지날수록 코드량은 점점 많아지고 문제가 발생할 가능성도 늘어나게 된다.

<br><br><br>

## 2-4 샘플코드

![05_mvc_activity]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/05_mvc_activity.png){: .center}

<center>
  <span style="font-size: small; color: grey">MVC 패턴을 적용한 코드 (뷰와 컨트롤러의 구분이 쉽지 않다.)</span>
</center>
<br>

코드를 살펴보면 클릭 이벤트와 뷰에 대한 처리가 함께 있는 것을 확인할 수 있다. 안드로이드 액티비티의 특성 때문에 이러한 현상이 발생하게 되는데 이를 개선하기 위해 MVP와 MVVM 패턴이 등장하게 되었다.

<br><br><br><br>

# 3. MVP

------



## 3-1 구조 및 동작

사용자 입력은 이제 뷰를 통해 들어오게 된다. 뷰는 이러한 이벤트를 프리젠터로 전달하고 프리젠터는 모델과의 상호작용을 통해 뷰에게 업데이트 할 내용을 전달한다. 그리고 이 내용을 받은 뷰가 최종적으로 업데이트 된다.

<br>

![06_mvp_pattern]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/06_mvp_pattern.png){: .center}

<center>
<span style="font-size: small; color: grey">MVP 구조 및 동작</span>
</center>
<br><br><br>

## 3-2 특징

### ￭ Model

- MVC와 동일

<br>

### ￭ View

- 사용자에게 제공되는 UI
- 액티비티/프래그먼트가 뷰의 일부로 간주
- 사용자의 입력을 받고 이벤트를 프리젠터로 전달

<br>

### ￭ Presenter

- 모델과 뷰 상호작용 관리
- 컨트롤러와 본질적 동일 but 뷰에 연결되지 않는 단순 인터페이스
- 뷰에게 표시할 내용만 전달 (표시 방법 지시 X)

<br><br>

MVC와의 차이점은 액티비티와 프래그먼트가 이제 뷰의 일부로 간주된다는 점이다. 뷰는 사용자의 입력 이벤트를 프리젠터로 전달하는 역할을 한다. 프리젠터는 모델과 뷰의 가운데서 상호작용 관리를 한다. 본질적으로는 MVC의 컨트롤러와 동일하지만 뷰에 연결되지 않는 단순 인터페이스라는 점에서 차이가 있다. 그렇기 때문에 뷰에게 표시할 방법을 지시하지 않고 단순히 표시할 내용만 뷰에게 전달하게 된다. 그리고 이 내용을 바탕으로 뷰는 스스로 업데이트를 하게 된다.

<br><br><br>

## 3-3 장/단점

### ￭ 장점

- 모델과 뷰의 의존성 X
- 모델은 프리젠터의 요청만 수행 (다른 상호작용 신경 X)

<br>

### ￭ 단점

- 시간이 지남에 따라 프리젠터도 추가 비즈니스 로직이 모여 비대화
- MVC에 비해 필요한 클래수 수 증가
- 뷰와 프리젠터 1:1 관계로 인한 의존성 증가

<br><br>

MVP 패턴에서 모델과 뷰는 MVC 패턴에서와는 달리 더이상 서로 간의 의존성이 존재하지 않는다. 모델은 프리젠터의 요청만 수행하면 되므로 다른 부분과의 상호작용은 전혀 신경쓰지 않아도 된다. 하지만 프리젠터도 시간이 지날수록 코드가 쌓여 비대해지게 되어 유지보수에 어려움이 있을 수 있다. 그리고 프리젠터를 구현하기 위해 인터페이스와 인터페이스 구현체를 구현해야 하는 등 MVC에 비해서 필요한 클래수 수가 증가하게 된다. 또한 뷰와 프리젠터의 1:1 관계로 인해 서로 간 의존성이 커지게 된다는 단점이 있다.

<br><br><br>

## 3-4 샘플코드

![07_mvp_interface]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/07_mvp_interface.png){: .center}

<center>
  <span style="font-size: small; color: grey">프리젠터 인터페이스 (베이스 프리젠터)</span>
</center>
<br>

베이스가 되는 프리젠터는 단순한 인터페이스에 불과하다.

<br>

![08_mvp_activity]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/08_mvp_activity.png){: .center}

<center>
	<span style="font-size: small; color: grey">뷰의 처리</span>
</center>
<br>

뷰는 베이스 프리젠터의 뷰 인터페이스를 implement 했고(이후 프리젠터로부터 업데이트할 내용을 받은 후 뷰 업데이트) 사용자 터치 이벤트가 들어왔을 때 프리젠터로 해당 이벤트를 전달한다.

<br>

![09_mvp_presenter]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/09_mvp_presenter.png){: .center}

<center>
	<span style="font-size: small; color: grey">프리젠터 인터페이스 구현체</span>
</center>
<br>

뷰로부터 이벤트가 들어오면 프리젠터는 모델로부터 데이터를 받아 뷰에게 해당 데이터를 전달한다.

<br><br><br><br>



# 4. MVVM

------



## 4-1 구조 및 동작

사용자 입력은 뷰를 통해 들어오게 된다. 뷰는 이러한 이벤트를 뷰모델로 전달하고, 뷰모델과 모델의 상호작용 후 모델이 변경되면 관련된 뷰모델을 사용하는 뷰가 자동으로 업데이트 된다.

<br>

![10_mvvm_pattern]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/10_mvvm_pattern.png){: .center}

<center>
	<span style="font-size: small; color: grey">MVVM 구조 및 동작</span>
</center>

<br><br><br>

## 4-2 특징

### ￭ Model

- MVC와 동일

<br>

### ￭ View

- 사용자에게 제공되는 UI
- 사용자의 입력을 받고 이벤트를 자신이 사용할 뷰모델로 전달

<br>

### ￭ ViewModel

- 뷰를 나타내주기 위한 모델 + 뷰의 표현 로직 담당
- 뷰와 독립적
- UI 관련 데이터 보관, 관리
- 모델이 변경되면 관련된 뷰모델을 사용하는 뷰가 자동 업데이트

<br><br><br>

## 4-3 장/단점

### ￭ 장점

- 뷰에 대한 의존성이 전혀 없으므로 유닛 테스트 용이

<br>

### ￭ 단점

- 뷰에 대한 처리가 복잡해 질수록 뷰모델이 거대해짐
- 상대적으로 뷰는 아무 역할도 하지 않음
- 뷰모델이 또다른 형태의 액티비티 클래스 구현으로 변질

<br><br>

뷰모델은 뷰와 완전히 독립적이므로 유닛 테스트에 용이하다는 장점이 있다. 하지만 뷰모델 역시 로직이 쌓이게 되어 복잡해질 수 있다. 또 상대적으로 뷰가 아무 역할도 하지 않게 되어 뷰모델이 또다른 형태의 액티비티 클래스를 구현한 꼴이 되어버릴 수도 있다.

<br><br><br>

## 4-4 샘플코드

![11_mvvm_activity]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/11_mvvm_activity.png){: .center}

<center>
<span style="color:grey; font-size: small; color: grey">뷰 (so simple..)</span>
</center>
<br>

뷰가 더이상 큰 역할을 하지 않는다. 뷰의 역할을 이제 뷰모델이 담당하게 된다. 뷰모델의 인스턴스는 보통 뷰가 onCreate() 될 때 요청된다.

<br>

![12_mvvm_viewmodel]({{site.baseurl}}/assets/img/03_DESIGN_PATTERN/12_mvvm_viewmodel.png){: .center}

<center>
<span style="color:grey; font-size: small; color: grey">뷰모델</span>
</center>
<br>

initView() 부분을 잘보면 어디서 많이 보던 코드들 아닌가? 뷰에서 일어나던 일들을 이제는 뷰모델이 대신한다. 뷰(액티비티)의 수명주기의 영향을 받지 않고 뷰모델 인스턴스가 유지되면서 데이터를 안전하게 다룰 수 있게 되었다. 하지만 앞서 언급한 것처럼 뷰가 할 일을 뷰모델이 대신하기에 뷰모델에 로직들이 모이게 되고 또다른 뷰 클래스를 생성한 꼴이 되어버릴 수 있음에 유의하여야 한다.

<br><br><br><br><br>

마치며,

<br>

아키텍처 패턴에 대해 여러 자료를 찾아보고 검토해봤지만 사실 아직도 완벽하게 아키텍처 패턴들을 이해한 것은 아니다. 하지만 어느정도 이해한 부분을 이렇게나마 정리해 남겨두고 앞으로 더 공부를 해야겠다. 또 이 패턴들을 조금씩 실제 업무에도 적용해 어느 부분이 좋고, 어느 부분이 나쁜지 직접 느껴봐야겠다.

<br><br>
<br><br><br>

(샘플 코드는 [GitHub]()에서 볼 수 있습니다.)

<br><br>
<br><br><br><br>

참고 : 

https://developer.android.com/jetpack/docs/guide?hl=ko

https://academy.realm.io/kr/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/

https://medium.com/@jungil.han/%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-viewmodel-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-2e4d136d28d2

https://thdev.tech/androiddev/2016/10/12/Android-MVP-Intro/



[GitHub]: https://github.com/stonybean/DesignPatterns
