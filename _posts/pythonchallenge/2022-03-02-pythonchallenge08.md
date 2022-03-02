---
title: "[pythonchallenge] 8번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - bzip2
  - python bz2
  - import bz2


toc: true
toc_sticky: true
toc_label: "8번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 8번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/integrity.html](http://www.pythonchallenge.com/pc/def/integrity.html){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/156386524-63197ac8-90e8-4b50-861b-46b7ffa046ab.png){: .align-center}

소스 코드

```html
<html>
<head>
  <title>working hard?</title>
  <link rel="stylesheet" type="text/css" href="../style.css">
</head>
<body>
	<br><br>
	<center>
	<img src="integrity.jpg" width="640" height="480" border="0" usemap="#notinsect"/>
	<map name="notinsect">
	<area shape="poly" 
		coords="179,284,214,311,255,320,281,226,319,224,363,309,339,222,371,225,411,229,404,242,415,252,428,233,428,214,394,207,383,205,390,195,423,192,439,193,442,209,440,215,450,221,457,226,469,202,475,187,494,188,494,169,498,147,491,121,477,136,481,96,471,94,458,98,444,91,420,87,405,92,391,88,376,82,350,79,330,82,314,85,305,90,299,96,290,103,276,110,262,114,225,123,212,125,185,133,138,144,118,160,97,168,87,176,110,180,145,176,153,176,150,182,137,190,126,194,121,198,126,203,151,205,160,195,168,217,169,234,170,260,174,282" 
		href="../return/good.html" />
	</map>
	<br><br>
	<font color="#303030" size="+2">Where is the missing link?</font>
</body>
</html>

<!--
un: 'BZh91AY&SYA\xaf\x82\r\x00\x00\x01\x01\x80\x02\xc0\x02\x00 \x00!\x9ah3M\x07<]\xc9\x14\xe1BA\x06\xbe\x084'
pw: 'BZh91AY&SY\x94$|\x0e\x00\x00\x00\x81\x00\x03$ \x00!\x9ah3M\x13<]\xc9\x14\xe1BBP\x91\xf08'
-->

```
# 문제 풀이
html 소스코드를 보면, poly 타입으로 좌표를 주어서 다차원 도형을 만들고 있습니다. 사진을 보면, 파리처럼 생긴 그림을 다각형 형태로 표현한 내용이고 그림을 클릭하면, `return/good.html`로 이동시켜줍니다.

그리고 맨 아래에 `un`,`pw`로 주석정보가 되어 있는데, 맨 앞글자를 보니 bzip2로 압축된 내용으로 추측이 가능합니다.

파이썬을 이용해서 이를 압축해제 해보겠습니다.

```py
import bz2

un = b'BZh91AY&SYA\xaf\x82\r\x00\x00\x01\x01\x80\x02\xc0\x02\x00 \x00!\x9ah3M\x07<]\xc9\x14\xe1BA\x06\xbe\x084'
pw = b'BZh91AY&SY\x94$|\x0e\x00\x00\x00\x81\x00\x03$ \x00!\x9ah3M\x13<]\xc9\x14\xe1BBP\x91\xf08'

bz2.decompress(un)  #huge
bz2.decompress(pw)  #file
```

이제 위의 id/pw를 이용해서 링크로 이동하여 로그인 하면, 성공적으로 문제가 풀립니다.


# 다음 문제
[http://www.pythonchallenge.com/pc/return/good.html](http://www.pythonchallenge.com/pc/return/good.html){:target="_blank"}

