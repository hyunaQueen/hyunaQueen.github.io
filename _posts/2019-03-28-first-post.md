---
title: "안드로이드 프레임워크 공부 2일차"
date: 2019-03-28 15:22:05 -0400
categories: Android Framework
---

JNI와 NDK
=============


4.1 안드로이드와 JNI
-------------
### 4.1.1왜 안드로이드에서 JNI를 알아야 하는가?
<img src="https://user-images.githubusercontent.com/48199401/55168989-47fae200-51b7-11e9-830c-571c35fbb2be.jpg">
안드로이드 프레임 워크는 자바와 C/C++기반 모듈이 계층별로 구성되어있고 계층별 모듈 대부분은 서로 밀접하게 관련이 있다.
그림을 보고 예시를 들면 애플리케이션에서 GPS정보를 얻기 위해서는 프레임워크의 Location Manager가 제공하는 자바API를 호출하면 된다.
이 호출은 프레임워크 내부의 GPS라이브러리를 통해 GPS디바이스 드라이버에 연결되서 정보값을 전달해주는 동작 구조이다. 
C/C++레이어와 자바 레이어를 넘나들면서 동작한다.
C/C++레이어와 자바 레이어가 상호 작용(인터페이스)을 하게 해주는것이 바로 **JNI(Java Native Interface)**이다.

JNI를 사용하면 자바 클래스에서 C로 작성된 라이브러리를 불러올수도 있고, C클래스에서 자바로 작성된 라이브러리를 사용할수도 있다.

### JNI를 사용하는 경우
* 빠른 처리 속도를 요구하는 루틴 작성
  - 자바는 네이티브 코드에 비해 느리기 떄문에 빠른 처리 속도가 필요한 경우에는 C나 C++로 작성을 하고JNI를 통해서 자바에서 사용하는 방법이 있다.
* 하드웨어 제어
  - 하드웨어 제어 코드를 C로 작성한 다음 JNI를 통해 자바 레이어와 연결하면 자바에서도 하드웨어 제어가 가능하다.
* 기존 C/C++프로그램의 재사용
  - 이미 기존 코드가 대부분 C/C++로 작성되어 있다면 자바로 옮겨 작성하지 않고 JNI를 사용해서 그대로 사용한다.

안드로이드 프레임 워크는 JNI를 사용해서 자바와 C/C++의 장점을 동시에 활용하고 있다.

4.2 JNI의 기본 원리 이해
-------------
### 4.2.1자바에서 C 라이브러리 함수 호출하기
* 자바에서 C함수 호출하는 순서
1. 자바 코드 작성
  **네이티브 메서드 vs 네이티브 함수**
  * 네이티브 메서드 : 자바에서 선언된 메서드
  * 네이티브 함수 : c에서 실제 구현된 함수
  <img src="https://user-images.githubusercontent.com/48199401/55176391-66b3a580-51c4-11e9-9914-3198da633e7f.jpg">
    + 1에서 native로 선언된 메서드는 실제 자바가 아닌 다른 언어로 구현되어 있음을 알려준다. printHello()나 printString()은 자바 구현부가 없음.
    native를 생략하게 되면 구현부가 없기때문에 컴파일 오류가 난다.
    + 2에서는 실제 구현되어있는 c라이브러리는 호출해서 로딩한다.
    + 3 c로 구현된 함수를 호출할 수 있는 자바클래스가 완성되었기 때문에 main()함수에서 객체를 만들고 네이티브 메소드를 호출하면 JNI네이티브 함수가 호출된다.
  
2. 자바 코드 컴파일
>1단계에서 작성한 소스 코드를 컴파일을 하게 된다.
  ```javac HelloJni.java```명령을 입력한다.
  컴파일을 하게되면 **HelloJNI.class**파일이 생긴다. 이상태에서 바로 실행을 하게 되면 오류가 생긴다. 자바 코드에서 로딩할 hellojni.dll파일을 아직 만들지 않았기 때문에 로딩할 C라이브러리를 찾지 못했다!
3. C헤더 파일 생성
4. C코드 작성
5. C공유 라이브러리 생성
6. 자바 프로그램 