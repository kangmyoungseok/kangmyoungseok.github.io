---
title: "[pythonchallenge] 4번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - python requests
  - requests
  - import requests
  - 4번
toc: true
toc_sticky: true
toc_label: "4번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 4번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/linkedlist.php](http://www.pythonchallenge.com/pc/def/linkedlist.php){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/154649177-3ade1c5c-4205-4acd-bda1-e291c3097769.png)

이전 문제와 같이 `ctrl + u` 를 눌러서 소스 코드를 봅니다.

![image](https://user-images.githubusercontent.com/33647663/154649412-0212ad1e-9815-41b1-96b8-c396c2a0d557.png)



# 문제 풀이
코드를 보니, 문제 이미지에 ```<a href="linkedlist.php?nothing=12345">``` 형태로 링크가 걸려 있습니다. 이미지를 누르면 다음과 같은 화면이 나옵니다.

![image](https://user-images.githubusercontent.com/33647663/154649639-cfa2d060-55c9-4312-8a2b-569089f4be36.png)


이렇게 계속 링크를 타고 가면 되는 문제입니다. 주석에 나와있듯이, 파이썬 requests 라이브러리를 이용하여 요청을 하고, 응답값에서 숫자만 추출하여 다음 요청을 반복하도록 해줍니다.


```python
import requests
import re

url = "http://www.pythonchallenge.com/pc/def/linkedlist.php"

params ={'nothing' : 12345}
response = requests.get(url,params=params)

while True:
    params['nothing']= re.sub(r'[^0-9]','',response.text)
    response = requests.get(url,params = params)
    if('and the next nothing' not in response.text):
        print(response.text)
        break
    else:
        print(response.text[-5:])

# 중간에 한번 수동작업 해줘야 함
params = {'nothing' : 8022}
response = requests.get(url,params=params)

while True:
    params['nothing']= re.sub(r'[^0-9]','',response.text)
    response = requests.get(url,params = params)
    if('and the next nothing' not in response.text):
        print(response.text)
        break
    else:
        print(re.sub(r'[^0-9]','',response.text))
```

![image](https://user-images.githubusercontent.com/33647663/154737224-3ee2e026-bf87-4578-a1f3-fd178ac7afc8.png)

# 다음 문제

[http://www.pythonchallenge.com/pc/def/peak.html](http://www.pythonchallenge.com/pc/def/peak.html){:target="_blank"}

