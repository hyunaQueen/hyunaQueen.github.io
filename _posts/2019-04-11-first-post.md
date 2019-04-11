---
title: "안드로이드 프레임워크 공부 3일차"
date: 2019-04-11 20:44:05 -0400
categories: Android Framework
---

JNI와 NDK -2
=============


JniFuncMain 클래스 
-------------
<img src="https://user-images.githubusercontent.com/48199401/55968832-b9f32100-5cb7-11e9-8744-4050fe657888.PNG">
예제의 메인 코드인 JniFuncMain 클래스 이다.

JniTest 클래스
-------------
<img src="https://user-images.githubusercontent.com/48199401/55969127-3be34a00-5cb8-11e9-8a16-0404eacd48aa.PNG">
테스트 소스로 객체를 생성하고 callByNativa()메소드를 호출할 것이다. 

JniFuncMain.h 헤더 파일
-------------
```javah jniFuncMain```을 통해 헤더파일을 생성한다.
<img src="https://user-images.githubusercontent.com/48199401/55969477-d2177000-5cb8-11e9-89f2-ff0d700c8ffa.PNG">
<img src="https://user-images.githubusercontent.com/48199401/55969520-e3f91300-5cb8-11e9-8338-1fc17cd15a06.PNG">
헤더파일이 생성된 모습

jnifunc.cpp 파일
-------------
헤더파일 정보를 바탕으로 jnifunc.cpp을 만들어 보자. 
<imp src="https://user-images.githubusercontent.com/48199401/55976232-a69b8200-5cc6-11e9-90bd-58c85e776ffe.PNG">
