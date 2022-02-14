---
title: "[OWASP-MSTG] Level1 문제 풀이"
categories:
  - andriod
tags:
  - reverse engineering
  - Frida
  - OWASP-MSTG
  - level1
  - ADB
  - Android
  - Nox
  - jadx
toc: true
toc_sticky: true
toc_label: "Level1 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/owasp.jpg
---

💡 OWASP-MSTG Level1 문제에 대한 풀이입니다.
{: .notice--warning}

# 개요
문제 파일은 다음 깃허브 페이지에서 다운받으실 수 있습니다.

문제 링크 : [https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/Android](https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/Android){:target="_blank"}



# 문제 풀이
앱을 실행하면 다음과 같이 `Root detected!` 메시지와 함께 바로 꺼집니다.

![image](https://user-images.githubusercontent.com/33647663/153807007-c7c35d47-4106-4656-a97a-268fae294027.png){: .align-center}

jadx로 디컴파일하여 소스코드를 분석합니다.

```java
public class MainActivity extends Activity {
    private void a(String str) {
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

    protected void onCreate(Bundle bundle) {
        if (c.a() || c.b() || c.c()) {
            a("Root detected!");
        }
        if (b.a(getApplicationContext())) {
            a("App is debuggable!");
        }
        super.onCreate(bundle);
        setContentView(R.layout.activity_main);
    }
```

소스코드를 보면, 생성자에서 `c.a() || c.b() || c.c()` 조건이 모두 참이면 `a("Root detected!");` 함수를 호출합니다.

먼저 c Class를 디컴파일하여 분석하면 다음과 같습니다. a,b,c 함수는 모두 루팅을 검사하는 로직입니다. 

```java
public class c {
    public static boolean a() {
      // 환경변수 PATH를 파싱한 다음, su가 경로에 존재하면 true
        for (String str : System.getenv("PATH").split(":")) {
            if (new File(str, "su").exists()) {
                return true;
            }
        }
        return false;
    }

    public static boolean b() {
      // Build.Tags에 test-keys가 있는지 검사
        String str = Build.TAGS;
        return str != null && str.contains("test-keys");
    }

    public static boolean c() {
        // 다음의 파일이 있는지 검사
        for (String str : new String[]{"/system/app/Superuser.apk", "/system/xbin/daemonsu", "/system/etc/init.d/99SuperSUDaemon", "/system/bin/.ext/.su", "/system/etc/.has_su_daemon", "/system/etc/.installed_su_daemon", "/dev/com.koushikdutta.superuser.daemon/"}) {
            if (new File(str).exists()) {
                return true;
            }
        }
        return false;
    }
}

```

현재 a,b,c 조건중 루팅을 탐지하는 로직에 걸려서 `MainActivity.a(str)` 함수가 호출되어 `AlertDialog`를 만들고 사용자의 onclick() 이벤트를 기다리고 있는 상태입니다. 사용자가 버튼을 누르면, `system.exit(0);` 함수를 호출하여 프로그램을 종료합니다.

따라서 우선 프로그램이 종료되지 않도록 system.exit()함수를 후킹하겠습니다.

```python
import frida,sys

def on_message(message,data):
    print(message)
    
    
jscode = """
Java.perform(function(){
    console.log("[+] hooking System API");
    var SystemClass = Java.use("java.lang.System");
    console.log(SystemClass);
    SystemClass.exit.overload('int').implementation = function(param1){
        console.log("[+] System.exit called");
    }
})
"""

session = frida.get_usb_device(timeout=1).attach("Uncrackable1")
script = session.create_script(jscode)
script.on("message",on_message)
script.load()
sys.stdin.read()
```

위와 같이 system.exit() 함수를 오버로딩하여, 사용자가 버튼을 클릭하여 system.exit() 함수를 호출해도 프로그램이 종료되지 않도록 처리해 줍니다.

후킹을 한 뒤, 버튼을 클릭하면 다음처럼 Secret String을 입력하는 칸이 나옵니다.

![image](https://user-images.githubusercontent.com/33647663/153804950-1c55a395-e961-445a-99a7-0234afcbf442.png){: .align-center}

해당 `Secret String`을 검증하는 `Verify` 함수의 코드는 다음과 같습니다. `a.a(obj)` 함수를 호출하여 true를 반환하면 성공하는 로직으로 되어있습니다. obj에는 사용자가 입력하는 문자열이 들어가 있습니다.

```java
public void verify(View view) {
        String str;
        // 사용자의 입력값을 obj에 저장
        String obj = ((EditText) findViewById(R.id.edit_text)).getText().toString();
        AlertDialog create = new AlertDialog.Builder(this).create();
        if (a.a(obj)) {
          //a.a(obj)가 참이면 성공
            create.setTitle("Success!");
            str = "This is the correct secret.";
        } else {
            create.setTitle("Nope...");
            str = "That's not it. Try again.";
        }
        create.setMessage(str);
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() { // from class: sg.vantagepoint.uncrackable1.MainActivity.2
            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int i) {
                dialogInterface.dismiss();
            }
        });
        create.show();
    }

```

a.a(obj) 함수를 보기위해서 a Class를 분석합니다. 아래에 a클래스 내부를 보면, obj를 **bArr**과 비교를 해서 같으면 true를 반환합니다. 

```java
public class a {
    public static boolean a(String str) {
        byte[] bArr = new byte[0];
        try {
            bArr = sg.vantagepoint.a.a.a(b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0));
        } catch (Exception e) {
            Log.d("CodeCheck", "AES error:" + e.getMessage());
        }
        // bArr와 obj가 같으면 true를 반환
        return str.equals(new String(bArr));
    }

    public static byte[] b(String str) {
        int length = str.length();
        byte[] bArr = new byte[length / 2];
        for (int i = 0; i < length; i += 2) {
            bArr[i / 2] = (byte) ((Character.digit(str.charAt(i), 16) << 4) + Character.digit(str.charAt(i + 1), 16));
        }
        return bArr;
    }
}

```

따라서 bArr가 우리가 찾는 secret 키 이며, 이는 다른 클래스의 함수와 Base64인코딩을 처리하여 평문으로는 읽히지 않도록 처리하고 있습니다.

우리는 함수를 후킹하여 사용할 수 있기 때문에 이 이상은 코드를 분석할 필요 없이 bArr 를 만드는 **sg.vantagepoint.a.a.a** 함수를 호출하며, 매개변수로 **b("8d127684cbc37c17616d806cf50473cc")**와 **Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0)**를 동일하게 설정하여 주면 secret 키를 알아낼 수 있습니다.


다음은 위의 내용을 코드로 구현한 내용입니다. 
secret1,2에 각각 bArr를 구하기 위해 필요한 값들을 그대로 만들어 주고, secret1,2를 이용해서 bArray를 다시 생성해 준다음, String으로 형 변환을 하여 출력해 줬습니다.

```javascript
// secret1 생성
var SecretClass = Java.use("sg.vantagepoint.uncrackable1.a");
var secret1 = SecretClass.b("8d127684cbc37c17616d806cf50473cc");

// secret2 생성
var Base64Class = Java.use("android.util.Base64");
var secret2 = Base64Class.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0);
    
// secret1,secret2를 이용해서 bArray 생성
var EncryptClass = Java.use("sg.vantagepoint.a.a");
var bArry = EncryptClass.a(secret1,secret2);

// bArray를 String 형으로 출력하여 secret 값 출력
var StringClass = Java.use("java.lang.String");
console.log(StringClass.$new(bArry));
```

실행하면 다음과 같이 secret 값이 출력됩니다.

![image](https://user-images.githubusercontent.com/33647663/153806595-0d9defd0-2757-417b-81ef-d61438c07c14.png)



# 전체 코드
<script src="https://gist.github.com/kangmyoungseok/3ff7117ab969339a60e695165ee23f06.js"></script>



