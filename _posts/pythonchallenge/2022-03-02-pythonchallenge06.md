---
title: "[pythonchallenge] 6번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - zip
  - zipfile
  - re

toc: true
toc_sticky: true
toc_label: "6번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 6번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/channel.html](http://www.pythonchallenge.com/pc/def/channel.html){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/156322049-59907125-289c-4d20-b6ce-4a63cb13132a.png)


소스 코드

```html
<html> <!-- <-- zip -->
<head>
  <title>now there are pairs</title>
  <link rel="stylesheet" type="text/css" href="../style.css">
</head>
<body>
<center>
<img src="channel.jpg">
```


# 문제 풀이
소스코드를 보면 `<!--> <-- zip -->` 로 zip이라는 주석이 표시되어 있습니다. 해당 힌트를 이용하여 페이지 주소를 `channel.zip`으로 접속하면 zip 파일이 받아집니다.

zip 파일을 열어보면 다음과 같이 많은 텍스트 파일이 들어 있습니다. 그리고 주석에 어떠한 문자들이 들어가 있습니다.

![image](https://user-images.githubusercontent.com/33647663/156322488-648162ea-34c9-439f-a111-3a7f35991f14.png)

readme.txt파일을 보면, 두가지 힌트가 있습니다.

![image](https://user-images.githubusercontent.com/33647663/156322671-b5a85842-b0f3-4ffd-b813-f061371d9fbe.png)

1. 90052번 부터 시작
2. zip파일 내에 답이 있다.


하나씩 파일을 읽어가면서, 링크를 따라가도록 코드를 작성해줍니다.

```py
from zipfile import ZipFile
import re

zip_obj = ZipFile('channel.zip')
print(zip_obj.read("readme.txt").decode("ascii"))

next = 90052

while True:
    result = zip_obj.read(str(next)+'.txt')
    if ('Next nothing is' in str(result)):
        next = re.sub(r'[^0-9]','',str(result))        
    else:   
        print(result.decode("ascii"))
        break

```
위의 코드를 실행결과 마지막 파일에서는 다음과 같이 또다른 힌트가 있습니다.

```
Collect the comments.
```

따라서 파일들을 탐색할 때, 힌트를 모아주는 코드도 추가해 줍니다.

```py
next = 90052

answer = ''
# 그다음은 끝까지 반복문, comment 수집
while True:
    result = zip_obj.read(str(next)+'.txt')
    if ('Next nothing is' in str(result)):
        answer += zip_obj.getinfo(str(next)+'.txt').comment.decode('ascii')
        next = re.sub(r'[^0-9]','',str(result))        
    else:
        print(result.decode("ascii"))
        break

print(answer)

```

![image](https://user-images.githubusercontent.com/33647663/156323305-5b049859-232e-433c-8e26-339462f0772e.png)

출력해보니 `hockey`가 나옵니다. 해당 url로 접속해줍니다.

![image](https://user-images.githubusercontent.com/33647663/156323514-9c09eead-3759-4863-a5c1-2d83936eb52f.png)

`공기에 있다. 글자를 봐라.` 라고 나옵니다. 다시 출력 문자를 자세히 보면 글자를 구성하는 알파벳들이 있습니다. 나열해보면 `oxygen`입니다. oxygen.html로 접속해보면 문제가 풀립니다.

# 전체 코드
```py
# http://www.pythonchallenge.com/pc/def/channel.html
# http://www.pythonchallenge.com/pc/def/channel.zip

from zipfile import ZipFile
import re


zip_obj = ZipFile('channel.zip')
print(zip_obj.read("readme.txt").decode("ascii"))


next = 90052

answer = ''
# 그다음은 끝까지 반복문, comment 수집
while True:
    result = zip_obj.read(str(next)+'.txt')
    if ('Next nothing is' in str(result)):
        answer += zip_obj.getinfo(str(next)+'.txt').comment.decode('ascii')
        next = re.sub(r'[^0-9]','',str(result))        
    else:
        print(result.decode("ascii"))
        break

print(answer)

# 답은 hockey가 아니라 OXYGEN 이란다 ^^
```

# 다음 문제
[http://www.pythonchallenge.com/pc/def/oxygen.html](http://www.pythonchallenge.com/pc/def/oxygen.html){:target="_blank"}

