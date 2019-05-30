---
title: "안드로이드 프레임워크 공부 4주차"
date: 2019-04-18 11:59:49 -0400
categories: Android Framework
---

Zygote
=============

Zygote란 무엇인가? 
-------------
* Zygote 프로세스는 애플리케이션이 실행되기 전에 실행된 가상 머신의 코드 및 메모리 정보를 공유함으로써 애플리케이션이 실행되는 시간을 단축시킬 수 있다.
* 안드로이드 프레임워크에서 동작하는 애플리케이션이 사용할 클래스와 자원을 미리 메모리에 구동해 두고 이러한 자원에 대한 연결 정보를 구성한다.

### Zygote를 통한 프로세스의 생성
Init 프로세스 실행 -> Daemon (Native Services)실행 -> Zygote 프로세스 실행

### 리눅스와 안드로이드에서 새로운 프로세스 생성시 차이점
(1) 리눅스 시스템에서 새로운 애플리케이션을 실행하는 과정
* 부모 프로세스 A는 fork() 시스템 콜을 호출하여 새로운 자식 프로세스 A'를 생성
  - 프로세스 A'는 부모 프로세스인 프로세스 A의 메모리 구성 정보 및 공유 라이브러리에 대한 링크 정보를 공유한 상태
* 자식프로세스 A'는 exec('B')시스템 콜을 호출해 새로운 프로세스 B의 코드를 메모리로 로딩
  - 부모 프로세스 A의 메모리 정보는 지워지고 로딩된 B를 실행하는데 필요한 메로리를 새롭게 구성한 후 프로세스 B가 사용할 공유 라이브러리가 메모리에 이미 로딩돼 있는 상태라면 이에 대한 링크 정보만 새롭게 구성하지만 그렇지 않은 경우에는 스토리지에서 해당 라이브러리를 메모리로 로딩하는 과정이 추가로 필요
* 이러한 과정이 새로운 프로세스를 실행할 때마다 일어나게 된다.

(2) 안드로이드 시스템에서 새로운 애플리케이션 실행하는 과정
* Zygote 프로세스는 fork()시스템 콜을 호출해 자식 프로세스인 Zygote'프로세스 생성
  - 생성된 Zygote'프로세스는 부모인 Zygote 프로세스의 코드 영역과 링크 정보를 공유
  - 새로운 안드로이드 어플리케이션 A는 복제된 달빅 가상 머신 위에 동적으로 로딩
* Zygote' 프로세스는 애플리케이션 A클래스의 메서드로 실행 흐름을 넘겨 애플리케이션이 동작하게 된다. 애플리케이션 A는 기존의 Zygote 프로세스가 구성해 놓은 라이브러리 및 리소스에 대한 링크 정보를 그대로 사용하기에 빠르게 실행 가능
<img src="https://user-images.githubusercontent.com/48199401/56362282-68e9ac80-6224-11e9-8583-51e647c59e40.jpg">

app_process로부터 ZygoteInit class실행
-------------
* Zygote는 자바로 작성되어 있기 때문에 다른 네이티브 서비스나 데몬과 같이 init프로세스에서 바로실행 불가능
* 자바로 작성돼 있는 Zygote 클래스가 동작하려면 달빅 가상 머신이 생성돼야 하고, 생성된 가상머신 위에서 ZygoteInit클래스를 로딩하고 실행해야 한다.
  - 이러한 작업을 수행하는 프로세스가 바로**app_process**

### AppRuntime 객체 생성
* **app_process**실행 규칙
> app_process [java-options] cmd-dir start-class-name [options]
* [java-options] : 가상 머신으로 전달되는 옵션, 반드시 '-'로 시작되어야 한다.
* cmd-dir : 프로젝트가 실행될 디렉터리
* start-class-name : 가상머신에서 생설할 클래스의 이름. app_process는 전달받은 클래스를 가상 머신으로 로딩한 후 해당 클래스의 main()메서드를 호출
* [options] : 실행될 클래스로 전달될 옵션

### AppRuntime 객체 실행
```
int main(int argc, const char* const argv[])
{
  ...
  if(i < argc) {
    arg = argv[i++];
    if (0 == strcmp("--zygote",arg)) {      -------(1)
      bool startSystemServer = (i < argc) ?
        strcmp(argv[i], "--start-system-server") == 0 : false;
      setArgv0(argv0, "zygote");
      set_process_name("zygote");           -------(2)
      runtime.start("com.android.internal.os.ZygoteInit",
                    startServer);           -------(3)
    } else {
      ...
    }
    ...
  }
}
```
(1) 에서 실행할 클래스의 이름을 확인한다. 실행할 클래스의 이름이 '--zygote'이냐에 따라 처리 과정이 조금 달라지지만 가상 머신상에서 클래스를 로딩한다는 점은 같다.
(3) 에서 AppRuntime의 start()멤버 함수를 호출하면 비로서 가상 머신이 생성되고 초기화된다. 생성된 가상머신에 ZygoteInit클래스를 로딩하고 main()메서드로 실행흐름이 바뀐다.

### 달빅 가상 머신의 생성
* 가상 머신의 실행 옵션을 설정하기 위해 property_get()함수 호출
  - int property_get(const char *key, char *value, const char *default_value)
* JNI_CreateJavaVM() 함수를 통해 달빌 가상 머신을 생성하고 실행
  - jint JNI_CreateJavaVM(JavaNM** p_vm, JNIEnv** p_env, void* vm_args)
    + JavVM **pVm : 생성된 JavaVM 클래스의 인스턴스에 대한 포인터
    + JNIEnv **p_env : 가상 머신에 접근하기 위한 JNIEnv 클래스의 인스턴스에 대한 포인터
    + void *vm_args : 지금까지 설정한 가상 머신의 옵션
    
ZygoteInit 클래스의 기능
-------------
```
public static void main(String argv[]){
  try {
    //새로운 안드로이드 애플리케이션의 실행 요청을 받기 위한 소켓 바인딩
    registerZygoteSocket();
    ...
    //안드로이드 애플리케이션 프레임워크에서 사용할 클래스와 리소스를 로딩
    preloadClasses();
    preloadResources();
    ...
    //SystemServer 실행
    if (argv[1].equals("true")) {
      startSystemServer();
    }
    ...
    if (ZYGOTE_FORK_MODE){
      startSystemServer();
    } else {
      //새로운 안드로이드 애플리케이션 실행 요청을 처리
      runSelectLoopMode();
    }
    closeServerSocket();
  }catch(MethodAndArgsCaller caller){
    caller.run();
  } catch(RuntimeException ex){
    Log.e(TAG, "Zygote died with exception", ex);
    closeServerSocket();
    throw ex;
  }
}
```

### /dev/socket/zygote 소켓 바인딩
Zygote클래스의 main()메서드는 가장 먼저 registerZygoteSocket()메서드를 호출한다.
* System.getenv() 메서드를 호출해서 환경변수로 등록한 소켓의 파일 디스크립터를 가져온다. 
* 파일 디스크립터 값을 이용해 LocalServerSocket클래스의 인스턴스를 생성하고 /dev/socket/zygote와 바인딩

### 애플리케이션 프레임워크에 속한 클래스와 플랫폼 자원의 로딩
***preloadClasses***
* 미리 메모리에 클래스들을 로딩해 놓아 새로운 애플리케이션이 실행될때 시작속도를 빠르게 해준다.
***preloadResources***
* 문자열, 색, 이미지 파일, 사운드 파일등을 리소스 파일로 관리된다.
* 리소스는 리스템 리소스와 애플리케이션 리소스로 나뉘는데 시스템 리소스에 접근하려면 getSystem() 정적 메서드에서 반환하는 객체 

### SystemServer 실행
Zygote에서 달빅 가상 머신을 구동한 이후, 시스템 서버(SystemServer)라는 자바 서비스를 실행하기 위해 새로운 달빅 가상 머신 인스턴스를 생성한다. 
* startSystemServer()메소드에서 새로운 프로세스 실행
* forkSystemServer()메소드에서 생성한 시스템 서버 프로세스의 동작 여부 확인

### 새로운 안드로이드 애플리케이션 실행
시스템 서버가 실행되고 나면 앞서 바인딩한 소켓으로 들어오는 요청을 처리하기 위한 루프가 실행된다. ZYGOTE_FORK_MODE가 false로 되어있으므로 ***runSelectLoopMode()***메서드 호출 이 메서드는 zygote프로세스가 종료 될때까지 반환되지 않는다.
