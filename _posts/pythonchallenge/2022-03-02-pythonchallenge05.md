---
title: "[pythonchallenge] 5번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - python pickle
  - pickle
  - pickle load

toc: true
toc_sticky: true
toc_label: "5번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 5번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/peak.html](http://www.pythonchallenge.com/pc/def/peak.html){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/156313780-e95e4fb6-1014-42b2-bb85-d0bc86f8ae95.png)


소스 코드

```html
<html>
<head>
  <title>peak hell</title>
  <link rel="stylesheet" type="text/css" href="../style.css">
</head>
<body>
<center>
<img src="peakhell.jpg"/>
<br><font color="#c0c0ff">
pronounce it
<br>
<peakhell src="banner.p"/>
</body>
</html>

<!-- peak hell sounds familiar ? -->

```


# 문제 풀이
첫 문제화면을 보면 `pronounce it` 이 힌트로 주어져 있습니다. 그리고 소스코드 주석을 보면 peak hell이 발음상 무엇과 비슷한지를 묻고 있습니다. 

정답은 `pickle`로, 파이썬에서 자료를 저장할때 사용하는 라이브러리 입니다. 따라서 이번 문제는 `pickle` 라이브러리를 활용하여 푸는 것임을 알 수 있습니다. 소스코드를 보면 `banner.p`파일을 주고 있습니다. 이 파일을 내려받아 pickle 라이브러리를 이용하여 load 해봅니다.

```html
<peakhell src="banner.p"/>
```

```python
import requests
import pickle

response = requests.get('http://www.pythonchallenge.com/pc/def/banner.p')
data = pickle.loads(response.content)
```

성공적으로 data에 저장되었으며, 그 구조를 보면 다음과 같습니다.

```sh
>>> type(data)
<class 'list'>
>>> data[0]
[(' ', 95)]
>>> data[1]
[(' ', 14), ('#', 5), (' ', 70), ('#', 5), (' ', 1)]
>>> data[2]
[(' ', 15), ('#', 4), (' ', 71), ('#', 4), (' ', 1)]
```

data는 list 형태로 구성되어 있으며, 하나씩 원소를 보면 튜플들의 집합으로 구성되어 있습니다.

또한 튜플을 보면 뒤의 숫자를 더하면 모두 95개가 나오는 것으로 보아 한 줄씩, 앞에 있는 문자들을 출력하면 될것이라고 추론이 가능합니다.

```py
for raw in data:
    for item in raw:
        print(item[0] * item[1], end='')
    print()

# 아래는 더 깔끔한 코드
for raw in datas:
    print("".join([word * repeat for word,repeat in raw]))
```

그러면 다음과 같이 `channel`이라는 문자가 출력됩니다.

![image](https://user-images.githubusercontent.com/33647663/156315458-edf590ef-374b-42b6-b0cb-e46da1dec458.png)

# 전체 코드
```py
# http://www.pythonchallenge.com/pc/def/peak.html

import requests
import pickle

response = requests.get('http://www.pythonchallenge.com/pc/def/banner.p')
datas = pickle.loads(response.content)

for raw in datas:
    print("".join([word * repeat for word,repeat in raw]))

```

# 다음 문제
[http://www.pythonchallenge.com/pc/def/channel.html](http://www.pythonchallenge.com/pc/def/channel.html){:target="_blank"}

