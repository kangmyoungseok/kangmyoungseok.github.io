---
title: "[OWASP-MSTG] Level2 문제 풀이"
categories:
  - andriod
tags:
  - reverse engineering
  - Frida
  - OWASP-MSTG
  - level2
  - ADB
  - Android
  - Nox
  - jadx
toc: true
toc_sticky: true
toc_label: "Level2 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/owasp.jpg
---

💡 OWASP-MSTG Level2 문제에 대한 풀이입니다.
{: .notice--warning}

# 개요
문제 파일은 다음 깃허브 페이지에서 다운받으실 수 있습니다.

문제 링크 : [https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/Android](https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/Android){:target="_blank"}



# 문제 풀이
앱을 실행하면 다음과 같이 `Root detected!` 메시지와 함께 바로 꺼집니다.

![image](https://user-images.githubusercontent.com/33647663/155748074-ddbb6339-bac6-4877-8bc3-0ff5edcc76e8.png){: .align-center}

jadx로 디컴파일하여 소스코드를 분석합니다.

```java
public class MainActivity extends Activity {
    // 후킹해야 할 함수. 주어진 문자열을 출력하고, 프로그램을 종료
    public void a(String str) {
        AlertDialog create = new AlertDialog.Builder(this).create();
        create.setTitle(str);
        create.setMessage("This is unacceptable. The app is now going to exit.");
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() { 
            public void onClick(DialogInterface dialogInterface, int i) {
                System.exit(0);
            }
        });
        create.setCancelable(false);
        create.show();
    }

    // 생성자. Root Check, Debugger check를 수행한다.
    public void onCreate(Bundle bundle) {
        init();
        if (b.a() || b.b() || b.c()) {
            a("Root detected!");
        }
        if (a.a(getApplicationContext())) {
            a("App is debuggable!");
        }
        new AsyncTask<Void, String, String>() { 
            public String doInBackground(Void... voidArr) {
                while (!Debug.isDebuggerConnected()) {
                    SystemClock.sleep(100L);
                }
                return null;
            }
            public void onPostExecute(String str) {
                MainActivity.this.a("Debugger detected!");
            }
        }.execute(null, null, null);
        this.m = new CodeCheck();
        super.onCreate(bundle);
        setContentView(R.layout.activity_main);
    }
```

MainActivity 소스코드에서 생성자인 onCreate()함수를 보면 Rooting 검사와 디버거 검사를 수행합니다. 우선 접속하자마자 루팅검사에 의해서 앱이 종료되므로, 실질적으로 함수를 종료시키는 exit() 함수를 후킹하거나, exit 함수를 호출하는 MainActivity.a 함수를 후킹하면 됩니다.

```js
// MainActivity.a 함수를 후킹하는 방법
var MainActivity = Java.use('sg.vantagepoint.uncrackable2.MainActivity');
    MainActivity.a.overload('java.lang.String').implementation = function(param1){
        console.log("[+] hooking Mainactivity.a :" + param1);
    }
```

```js
// System.exit() 함수를 후킹하는 방법
var System = Java.use('java.lang.System');
    System.exit.overload('int').implementation = function(param1){
        console.log("hooking exit function ");
    }
```

위의 방법으로 후킹을 하면 다음과 같이 a함수를 후킹했다는 로그와 함께, Secret String을 입력하라고 나옵니다.

![image](https://user-images.githubusercontent.com/33647663/155749658-96b99721-afda-412d-9d83-23de323a4873.png){: .align-center}

![image](https://user-images.githubusercontent.com/33647663/155749722-45cee604-706c-432e-91ac-f32858a37796.png){: .align-center}

Secret String을 검증하는 Verify() 함수는 다음과 같습니다.

```java
public void verify(View view) {
        String str;
        String obj = ((EditText) findViewById(R.id.edit_text)).getText().toString();
        AlertDialog create = new AlertDialog.Builder(this).create();
        if (this.m.a(obj)) {    // 이부분이 True면 성공
            create.setTitle("Success!");
            str = "This is the correct secret.";
        } else {
            create.setTitle("Nope...");
            str = "That's not it. Try again.";
        }
    }
```

코드를 보면 this.m.a(obj) 함수가 True를 반환하면 `Success`가 출력되도록 되어 있습니다. 이때 **m** 은 onCreate함수를 보면 다음과 같이 CodeCheck()객체임을 알 수 있습니다. 따라서 CodeCheck.a() 함수를 분석해 줍니다.

```java
    public void onCreate(Bundle bundle) {
        .. 생략
        this.m = new CodeCheck();
        .. 
    }
```

CodeCheck 클래스는 다음과 같이 선언되어 있습니다. a 함수는 native function인 bar 함수의 결과를 그대로 리턴하고 있습니다. native function인 bar를 분석하기 위해서, ida를 이용해서 모듈 분석을 해야 합니다. 

```java
public class CodeCheck {
    private native boolean bar(byte[] bArr);

    public boolean a(String str) {
        return bar(str.getBytes());
    }
}
```

이때 분석해야 할 모듈은 MainActivity에 보면 다음과 같이 "foo" 모듈임을 알 수 있습니다.

```java
public class MainActivity extends c {
    static {
        System.loadLibrary("foo");
    }
}
```

**UnCrackable-Level2.apk** 파일을 압축 해제한 뒤, **/lib/x86/libfoo.so**모듈을 찾아서 ida로 열어 줍니다. ida를 연 뒤 exports 탭을 보면 다음과 같이 CodeCheck_bar 함수가 보입니다.

![image](https://user-images.githubusercontent.com/33647663/155751010-436a816b-c26b-43f4-81a0-9467106d07bd.png)


CodeCheck_bar의 디컴파일된 소스코드는 다음과 같습니다. 코드가 복잡해 보이지만 쉽게 생각해서 우리의 목표인 리턴값 부분부터 확인해 보면 됩니다. 

```c
signed int __cdecl Java_sg_vantagepoint_uncrackable2_CodeCheck_bar(int a1, int a2, int a3)
{
  const char *v3; // esi
  signed int result; // eax
  int s2; // [esp+0h] [ebp-2Ch]
  int v6; // [esp+4h] [ebp-28h]
  int v7; // [esp+8h] [ebp-24h]
  int v8; // [esp+Ch] [ebp-20h]
  __int16 v9; // [esp+10h] [ebp-1Ch]
  int v10; // [esp+12h] [ebp-1Ah]
  __int16 v11; // [esp+16h] [ebp-16h]
  unsigned int v12; // [esp+18h] [ebp-14h]

  v12 = __readgsdword(0x14u);
  if ( byte_4008 != 1 )
    goto LABEL_9;
  s2 = 1851877460;
  v6 = 1713402731;
  v7 = 1629516399;
  v8 = 1948281964;
  v9 = 25960;
  v10 = 1936287264;
  v11 = 104;
  v3 = (const char *)(*(int (__cdecl **)(int, int, _DWORD))(*(_DWORD *)a1 + 736))(a1, a3, 0);
  if ( (*(int (__cdecl **)(int, int))(*(_DWORD *)a1 + 684))(a1, a3) != 23 )
    goto LABEL_9;
  if ( !strncmp(v3, (const char *)&s2, 0x17u) )
    result = 1;
  else
LABEL_9:
    result = 0;
  return result;
}

```

코드를 보면 다음의 조건을 만족하면 **result=1** 이 되고, 마지막에 return result를 하기 때문에 해당 조건에 따라서 bar함수의 리턴 값이 정해집니다.

```c
if ( !strncmp(v3, (const char *)&s2, 0x17u) )
    result = 1;
```

위의 코드는, v3 와 s2주소에 있는 23자리 문자열을 비교하여, 같으면 1을 리턴하는 것입니다. 

따라서 우리는 strncmp 함수를 Interceptor를 이용해서 후킹하면 됩니다. 다음과 같이 Module.findExportByname() 혹은 Module.getExportByName() 함수를 이용해서 함수의 메모리 주소값을 가져온다음 Interceptor를 붙여줍니다.

그다음, 우리가 secret String으로 입력할 값인 `01234567890123456789012` 가 strncmp 함수의 첫번째 인자로 들어오는지 확인합니다. strncmp 함수의 첫번째 값에서 `01234567890123456789012` 값이 있다면, 첫번째 인자와 두번째 인자를 출력하는 코드입니다. 이때, 주의해야 할 점은 param1 == '01234567890123456789012' 처럼 한다면  결과가 제대로 나오지 않는 다는 것입니다. 메모리에서 문자열이 있는지 검사할 떄는 정확한 일치인 **==** 보다, 메모리에 포함되어 있는지 검사하는 **indexOf**를 사용하는게 훨씬 정확합니다.  

```js
    Interceptor.attach(Module.findExportByName("libfoo.so","strncmp"),{
        onEnter: function(args){
            var param1 = Memory.readUtf8String(args[0]);
            var param2 = Memory.readUtf8String(args[1]);
            if(param1.indexOf('01234567890123456789012')!== -1){
                console.log(param1);
                console.log(param2);
            }
        },
        onLeave: function(){}
    })
```

이제 Frida를 이용해서 전체 코드를 실행한 뒤, 앱에서 다음과 같이 **01234567890123456789012**를 입력한 뒤 확인해 보면 Secret String값인 **Thanks for all the fish**가 출력됩니다. 

추가적으로 param1의 결과를 보면 끝에 `able2_Co` 이렇게 다른 문자가 붙어있는 것을 볼 수있습니다. 그렇기 때문에 **==**으로 조건을 두면 제대로 출력되지 않습니다.

![image](https://user-images.githubusercontent.com/33647663/155754965-df77e000-02ff-45a3-9fc3-9000162fbd24.png){: .align-center}

![image](https://user-images.githubusercontent.com/33647663/155753013-a59d21fd-6e71-45cb-82d7-dd1d41c792f7.png){: .align-center}

## 추가 사항
MainActivity를 보면 다음과 같이 AsyncTask가 실행되며 디버거가 붙는지 검사하고 있습니다.

```java
new AsyncTask<Void, String, String>() { 
            public String doInBackground(Void... voidArr) {
                while (!Debug.isDebuggerConnected()) {
                    SystemClock.sleep(100L);
                }
                return null;
            }
            public void onPostExecute(String str) {
                MainActivity.this.a("Debugger detected!");
            }
        }.execute(null, null, null);

```

AsyncTask는 선언되면 다음 그림과 같은 process로 동작하게 됩니다.

![image](https://user-images.githubusercontent.com/33647663/155753483-f15c7550-bdc2-462f-b5fe-ab97e98426ce.png)

이번 문제에서는 doInBackground가 while문 안에서 0.1초 마다 sleep을 하면서 Debug.isDebuggerConnected()를 검사하고 있습니다. 그리고, isDebuggerConnected()함수가 **true**를 반환하는 순간 while문의 조건이 탈출하게 되면서 onPosetExecute()함수가 실행되어 MainActivity.a함수가 실행되는 구조입니다.

이번 문제에서는 이미 a함수나 exit함수를 후킹해 놨기 때문에, 추가적인 작업이 필요하진 않았지만 어떻게 동작하는지 알아놔야 하며, 이 함수를 후킹한다고 했을때 다음과 같이 후킹을 할 수도 있습니다.

```js
var Debug = Java.use("android.os.Debug");
Debug.isDebuggerConnected.implementation = function(){
    //console.log("[+] hooking Debugger check")
    return false;
}
```

이때 console.log를 찍으면 0.1초마다 계속 로그가 출력되기 떄문에 로그 출력은 하지 않습니다.


# 전체 소스 코드

```js
Java.perform(function(){
    var MainActivity = Java.use('sg.vantagepoint.uncrackable2.MainActivity');
    MainActivity.a.overload('java.lang.String').implementation = function(param1){
        console.log("[+] hooking Mainactivity.a :" + param1);
    }


    // 현재 Async task에서 Debug.isDebuggerConnected()를 계속 탐지하고 있다.
    // While문을 계속 탐지하면서 isDebuggerConnected()가 return true이면, a 함수를 호출해서 
    // Debugger detected를 띄워줌. 현재 a 함수를 후킹했기 때문에 아래의 내용은 할 필요가 없다.
    // return false로 하면, [+] hooking Debugger check가 무한루프에서 계속 반복되기 때문에 엄청 뜨고,
    // return true로 하면, 바로 while문을 끝내고, 나오기 때문에 한번만 호출된다.
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function(){
        console.log("[+] hooking Debugger check")
        return false;
    }

    Interceptor.attach(Module.findExportByName("libfoo.so","strncmp"),{
        onEnter: function(args){
            var param1 = Memory.readUtf8String(args[0]);
            var param2 = Memory.readUtf8String(args[1]);
            if(param1.indexOf('01234567890123456789012')!== -1){
                console.log(param1);
                console.log(param2);
            }
        },
        onLeave: function(){}
    })
})

```