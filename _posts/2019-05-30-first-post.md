---
title: "안드로이드 프레임워크 공부 8주차"
date: 2019-05-30 16:01:08 -0400
categories: Android Framework
---

자바 시스템 서비스 동작 분석
=============
실제 자바 시스템 서비스 코드인 **액티비티 매니서 서비스**를 통해 동작 과정을 알아보자

액티비티 매니저 서비스 
-------------

## 액티비티 매니저 서비스란?
* 자바 시스템 서비스의 일종인 코어 플랫폼 서비스
* 안드로이드 애플리케이션 컴포넌트인 액티비티, 서비스, 브로드캐스트 리시버 등을 **생성**하고, 이들의 **생명주기**를 관리

 > 이 장에서는 시스템 서비스와 SDK기반의 애플리케이션 서비스 용어가 혼용되고 있다. 별도의 내용이 없으면 '서비스' 용어는 애플리케이션 컴포넌트의 요소인 애플리케이션 서비스를 의미

11장에서는 ApiDemos 예제 코드에 있는 Remote service Controller 애플리케이션을 토대로 액티비티 매니저 서비스가 안드로이드의 애플리케이션 서비스를 어떻게 생성하고, 서비스의 생명주기를 어떻게 제어하는지 알아보자

<img src="https://user-images.githubusercontent.com/48199401/58638325-80da4300-832f-11e9-83a2-f613038cc18a.jpg">
'Start Service'버튼을 눌렀을 때 액티비티 매니저 서비스에 의해 애플리케이션 서비스인 RemoteService가 어떻게 생성되는지 구체적으로 알아보자
1. 안드로이드 어플리케이션은 **startService()** 나 **bindService()**  API를 통해 애플리케이션 서비스를 생성한다. 그중에 startService()를 통해 서비스를 생성한다.
2. startService()를 통해 서비스 실행 요청을 받은 액티비티 매니저 서비스는 요청받은 서비스 클래스(RemoteService.class)를 바로 로드하는 것이 아니라 Zygote 에게 ActivityThread생성을 요청한다. 이는 RemoteService가 별도의 프로세스에서 동작하기 때문이다. ActivityThread는 모든 안드로이드 애플리케이션의 메인 쓰레드로, 액티비티 및 서비스의 생성 및 스케줄링을 담당한다.
3. 액티비티 매니저 서비스로부터 ActivityThread 실행을 요청받은 Zygote는 새로운 프로세스를 생성한 다음 그 위에 ActivityThread 클래스를 로딩한다.
4. 액티비티 매니저 서비스는 3번에서 생성된 ActivityThread에게 RemoteService 서비스의 생성을 요청한다.
5. ActivityThread는 RemoteService를 실행한다.

액티비티 매니저 서비스를 통한 서비스 생성 코드 분석
-------------
위의 과정을 소스 코드를 통해서 자세히 살펴보자

### Controller 액티비티 - startService() 메서드 호출
```
        private OnClickListener mStartListener = new OnClickListener() {
            public void onClick(View v) {
                startService(new Intent(Controller.this, RemoteService.class));
                //책에서 startService를 부를때, 암시적인 인텐트
                startService(new Intent("com.example.android.apis.app.REMOTE_SERVICE");
            }
        };
```
* 안드로이드는 인텐트를 이용해 실행하고자 하는 서비스의 클래스명을 명시적으로 지정해서 원하는 컴포넌트(여기서는 서비스)를 실행할 수 있다.
* com.example.android.apis.app.REMOTE_SERVICE 액션을 지정한 암시적인 인턴테를 이용해 RemoteService라는 서비스를 실행할 것을 요청한 예제

### 액티비티 매니저 서비스의 startService() 메서드 호출 과정(바인더 RPC활용)
* 액티비티에서 호출한 startService() API는 서비스 생성 및 실행과 관련된 내용을 액티비티 매니저 서비스에 요청하는 기능만 수행
* 실제 구현은 액티비티 매니저 서비스에 속한, 동일한 이름을 가진 **startService() 스텁 메서드**에 있다.
즉, 액티비티에서 호출한 startServie() API는 자바 서비스 프레임워크 기반에서 바인더 RPC형태로 액티비티 매니저 서비스에 제공하는 startService() 스텁 메소드를 호출한다.
<img src="https://user-images.githubusercontent.com/48199401/58648983-f3eeb400-8345-11e9-8fbc-23fe6e3c4baa.jpg">
1. Controller 액티비티(ActivityManagerProxy 객체의 startService()프록시 메서드를 호출)
 a) ContextWrapper 클래스 - ContextImpl 객체의 startService() 메서드 

정리
-------------
