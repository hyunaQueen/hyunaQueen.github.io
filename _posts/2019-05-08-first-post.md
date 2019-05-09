---
title: "안드로이드 프레임워크 공부 5주차"
date: 2019-04-18 21:50:18 -0400
categories: Android Framework
---

안드로이드 서비스 개요
=============
* 안드로이드에서 서비스는 UI 주기적으로 특정한 일을 수행하는 백그라운드 프로세스를 가리킨다.
  - 안드로이드 프로그램을 작성할 때 개발자가 적절한 애플리케이션 서비스를 직접 구현해서 적용한다면 더 반응성이 좋은 애플리케이션을 개발할 수 있다.
* 안드로이드 프레임워크 또한 애플리케이션 개발에 필요한 중요 API를 시스템 서비스 형태로 지원한다.

안드로이드 서비스 분류
-------------
<img src="https://user-images.githubusercontent.com/48199401/57457449-89000f00-72aa-11e9-8b5c-678a55d09a57.png">
안드로이드 서비스는 크게 프레임워크에서 기본제공하는 시스템 서비스와 애플리케이션 개발자가 Service클래스를 상속해서 구현한 애플리케이션 서비스로 구분된다.

안드로이드 애플리케이션 서비스
-------------
애플리케이션 서비스는 안드로이드 SDK의 Service클래스를 확장한 클래스의 인스턴스로 UI없이 주기적으로 특정한 일을 수행하는 백그라운드 프로세스를 가리킨다.
애플리케이션 개발자가 서비스를 사용하는 방법 두가지
* 서비스 시작, 종료 : 특정 기능을 수행하는 서비스를 백그라운드로 실행/종료 시킨다.
* 바인딩을 통한 서비스 원격 제어 : 액티비티처럼 서비스 클라이언트가 서비스에 바인딩을 하게되면 클라이언트는 바인딩이 유지되는 동안 서비스가 제공하는 인터페이스를 통해 서비스의 각종 기능을 제어할 수있다.
애플리케이션이 서비스를 생성하려면 startService(), binService()같은 API를 상황에 맞에 이용하면 된다.단순히 백그라운드에서 특정 동작 서비스를 실행하기만 하면 된다면 startService()를, 서비스에서 바인딩해서 서비스가 제공하는 인터페이스를 통해 서비스를 제어하고 싶다면 bindService()를 통해 서비스를 생성하면 된다.
<img src="https://user-images.githubusercontent.com/48199401/57465138-f61aa100-72b8-11e9-87f0-1b3c4ded4cad.jpg">
startService()와 bindService()로 시작된 서비스 모두 onCreate(), onDeatroy() 콜백 메서드가 호출된다. onCreate()메서드는 처음 서비스 생성시에 초기화를 해주고, onDestroy()메서드는 서비스 종료 직전에 서비스가 사용한 리소스들을 해제해준다.
* onStartCommand(Intent, int, int)는 startSevice()메서드의 첫 번째 인자로 넘어온 인텐트가 onStartCommand()의 첫번째 인자로 그대로 전달된다. 이때 인텐트에는 주로 실행할 서지스에 대한 정보가 포함되어있다. 이후 onStartCommand()에서 일반적으로 인텐트로부터 넘어온 인자가 있다면 이를 처리하거나 실직적인 백그라운드 작업을 처리하는 스레드를 실행한다.
* bindService()서비스에서는 onBind()는 클라이언트가 서비스에 바인딩하려고 할때 호출된다. 서비스의 onBind()콜백에서는 바인딩할 클라이언트를 위해 해당 서비스와 연결 가능한 객체를 제공한다. 따라서 바인딩이 끝나면 클라이언트는 이 객체를 통해 서비스를 원격제어 할 수 있다.

### 애플리케이션 서비스의 분류
애플리케이션 서비스는 로컬 서비스와 리모트 서비스로 구분한다. 구분하는 기준은 서비스와 이를 생성한 서비스 클라이언트가 동일한 프로레스에서 동작하고 잇는지 여부이다. 
<img src="https://user-images.githubusercontent.com/48199401/57466587-efd9f400-72bb-11e9-83b1-5ced61ff2bb3.jpg">
* 좌측의 그림처럼 생성된 서비스가 자신과 동일한 프로세스에서 실행되는 경우 해당 서비스를 로컬 서비스라고 부른다. 로컬 서비스는 자신을 생성한 어플리케이션 내에서만 사용 가능하며 애플리케이션이 종료되면 함께 종료된다.
* 로컬서비스와 달리 리모트서비스는 생성한 액티비티와 별개의 독립적인 프로세스 위에서 동작하기 때문에 메인 애플리케이션이 종료하더라도 계속 동작한다. 메인이 종료해도 동작해야되는 프로그램의 경우에는 리모트 서비스를 고려해야 하지만 잘못 구현된 리모트 서비스는 프로그램이 종료하더라도 시스템 자원(배터리 등)을 비효율적으로 소모할 수 있기 때문에 설계에 신중을 기해야 한다.
로컬 서비스와 리모트 서비스의 가장 큰 차이는 서비스 제어를 위한 바인딩 방법이다. 바인딩은 서비스 클라이언트 프로그램이 서비스를 원격 제어하기 위해 상호 연결하는 과정이다.
로컬 서비스는 바인딩할 로컬 서비스의 레퍼런스만 얻으면 되지만 리모트 서비스는 IPC 메커니즘을 이용해야 한다. 


## 로컬 서비스
예제를 통해 Local Service Binding을 살펴보자
<img src="https://user-images.githubusercontent.com/48199401/57468376-485ec080-72bf-11e9-819e-39e501860672.jpg">
(1) LocalService 연결 요청
```
        private OnClickListener mBindListener = new OnClickListener() {
            public void onClick(View v) {
                doBindService();
            }
        };
        void doBindService() {
            if (bindService(new Intent(Binding.this, LocalService.class),
                    mConnection, Context.BIND_AUTO_CREATE)) {
                mShouldUnbind = true;
            } else {
                Log.e("MY_APP_TAG", "Error: The requested service doesn't " +
                        "exist, or this client isn't allowed access to it.");
            }
        }
```
bindService(Intent, ServiceConnection, int)API의 첫 번째 인자는 LocalService를 실행하기 위한 인텐트이고, 두 번째 인자는 서비스 클라이언트 측에서 서비스와의 바인딩 연결을 처리할 객채다. 세번째 인자인 Context.BIND_AUTO_CREATE는 바인딩 서비스가 없을 경우 자동으로 생성하게 하는 플래그다. 따라서 LocalService가 아직 실행되지 않았으므로 바인딩에 앞서 LocalService서비스를 먼저 생성한다.

(2) 서비스와 통신하기 위한 LocalBinder 객체 반환
```
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    
    private final IBinder mBinder = new LocalBinder();
    
    public class LocalBinder extends Binder {
        LocalService getService() {
            return LocalService.this;
        }
    }
```
바인딩할 서비스가 생성됐으므로 바인딩 처리를 위해 서비스의 onBind()콜백 메서드를 호출한다. onBind()메서드는 액티비티가 LocalService 자신과 연결 할 수 있게 Binder 클래스를 확장한 LocalBinder객체를 반환한다.

(3) LocalBinder 객체의 getService()를 호출 -> LocalService 객체의 레퍼런스 반환
```
        private ServiceConnection mConnection = new ServiceConnection() {
            public void onServiceConnected(ComponentName className, IBinder service) {
                mBoundService = ((LocalService.LocalBinder)service).getService();
                Toast.makeText(Binding.this, R.string.local_service_connected,
                        Toast.LENGTH_SHORT).show();
            }
            public void onServiceDisconnected(ComponentName className) {
                mBoundService = null;
                Toast.makeText(Binding.this, R.string.local_service_disconnected,
                        Toast.LENGTH_SHORT).show();
            }
        };
```
프레임워크는 서비스 클라이언트 측 onServiceConnected(ComponentName, IBinder)메서드를 호출한다. 이 메서드의 두 번째 인자에는 onBinder()에서 생성된 LocalBinder 객체의 레퍼런스가 전달된다. 이제 Binding액태비티는 LocalBinder객체의 getService()메서드를 호출해서 바인딩하려고 했던 LocalService객체의 레퍼런스 값을 구한다.

(4) LocalService 객체 연결
이렇게 구한 LocalService 객체의 레퍼런스 값을 액티비티의 mBoundService멤버 필드에 저장하면 서비스 바인딩이 마무리된다.

일단 서비스가 바인딩 되고 나면 액티비티는 항상 서비스가 가진 모든 메서드와 멤버 필드 값을 mBoundService 멤버 필드를 통해 접근할 수 있다.

## 리모트 서비스 
예제를 통해 Remote Service Binding을 살펴보자. Local Service Binding과 다르게 ISecondary.aidl이라는 ADIL파일과 이 파일에 의해 자동 생성된 ISecondary.java파일이 추가 되어있다.
* RemoteService.java : RemoteService 서비스 외에 RemoteService를 이용하는 두 액티비티인 Controller와 Binding이 각각 내부 클래스로 선언돼 있다.
* ISecondary.aidl : 액티비티와 서비스 통신을 위한 인터페이스 정의
* ISecondary.java : ISecondary.aidl파일을 참조해 안드로이드가 자동으로 생생해주는 파일. ISecondary 인터페이스를 기반으로 액티비티와 서비스가 서로 통신할 수 있게끔 마샬링/언마샬링 수행

<img src="https://user-images.githubusercontent.com/48199401/57470626-3d5a5f00-72c4-11e9-9921-fc17f5a3796c.jpg">
(1) Bingding 액티비티 : RemoteService와의 연결 요청
```
        private OnClickListener mBindListener = new OnClickListener() {
            public void onClick(View v) {
                Intent intent = new Intent(Binding.this, RemoteService.class);
                intent.setAction(IRemoteService.class.getName());
                bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
                intent.setAction(ISecondary.class.getName());
                bindService(intent, mSecondaryConnection, Context.BIND_AUTO_CREATE);
                mIsBound = true;
                mCallbackText.setText("Binding.");
            }
        };
```
로컬 서비스와 비슷하게 bindService()에서 첫 번째 인자에 리모트 서비스를 실행하기 위한 인텐트가 전달된다.

(2) RemoteService 서비스 : 실제 서비스 메서드 기능 구현 및 서비스와 통신하기 위한 바인더 객체 제공
```
    private final ISecondary.Stub mSecondaryBinder = new ISecondary.Stub() {
        public int getPid() {
            return Process.myPid();
        }
        public void basicTypes(int anInt, long aLong, boolean aBoolean,
                float aFloat, double aDouble, String aString) {
        }
    };
    
    public IBinder onBind(Intent intent) {
        if (IRemoteService.class.getName().equals(intent.getAction())) {
            return mBinder;
        }
        if (ISecondary.class.getName().equals(intent.getAction())) {
            return mSecondaryBinder;
        }
        return null;
    }
```
서비스 바인더 객체는 자동으로 생성된 ISecondary.java의 ISecondary.Stub클래스를 통해 생성한다. 이때 ISecondary 인터페이스에 정의된 getPid()메서드의 실제 코드를 구현한다. getPid()의 구현에서는 서비스의 프로세스 ID를 반환한다.

(3) Bingding 액티비티 : 서비스와 바인더 IPC를 수행하기 위한 프록시 객체 생성
```
        private ServiceConnection mSecondaryConnection = new ServiceConnection() {
            public void onServiceConnected(ComponentName className, IBinder service) {
                mSecondaryService = ISecondary.Stub.asInterface(service);
            }
        };
```
프레임워크는 bindService()의 두 번째 인자로 넘긴 연결 객체의 onServiceConnected()콜백 메서드를 호출. 이때 두 번째로 인자로 전달되는 IBinder 타입의 service객체를 ISecondary.Stub.asInterface()함수의 인자로 전달해서 호출 함으로서 ISecond.Stub.Prox 서비스 프록시 객체가 생성되고 mSecondaryService 멤버 변수에 저장된다.

(4) Bingding 액티비티 : 서비스 프록시 객체를 이용해서 RemoteService 서비스의 getPid() 서비스 프록시 메서드 호출
```
        private OnClickListener mKillListener = new OnClickListener() {
            public void onClick(View v) {
                int pid = mSecondaryService.getPid();
                Process.killProcess(pid);
            }
        };
```
'Kill Process'버튼을 누르면 서비스 프록시 객체의 getPid()서비스 프록시 메서드가 호출된다. 액티비티는 서비스 프로세스를 Process.killProcess()메서드를 통해 강제 종료한다.

(5) 바인더 IPC : 서비스 프록시 객체(ISecond.Stub.Proxy)에서 서비스 바인더 객체(ISecond.Stub)로 바인더 IPC 데이터 전달
(4)에서 호출된 getPid()프록시 메서드의 처리를 위해 데이터를 ISecond.Stub.Proxy리모트 프록시 객체에서 바인더 IPC를 통해 ISecond.Stub서비스 바인더 객체로 전달된다.

(6) RemoteService 서비스 : RemoteService 서비스의 getPid() 스텁 메서드 호출
바인더 IPC데이터를 수신한 ISecond.Stub서비스 바인더 객체는 (2)에서 구현한 실제 getPid()스텁 메서드를 호출함으로써 서비스의 프로세스 ID를 액티비티에 반환한다.

