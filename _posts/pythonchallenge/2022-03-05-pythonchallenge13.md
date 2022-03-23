---
title: "[pythonchallenge] 13번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon

toc: true
toc_sticky: true
toc_label: "13번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 13번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/return/disproportional.html](http://www.pythonchallenge.com/pc/return/disproportional.html){:target="_blank"}
	+ id: huge
	+ pw: file



# 문제
![image](https://user-images.githubusercontent.com/33647663/159636049-544a8e97-e1f5-4e10-936e-91bcebe9954a.png)

**evil에게 전화를 하라**는 문제입니다. 그림에서 5번을 눌러보면 다음과 같은 오류 코드가 나옵니다.

![image](https://user-images.githubusercontent.com/33647663/159636140-9525a4d1-d6c3-4a1c-9692-d7d0f174d27b.png)

처음보는 유형의 오류이기 때문에 구글에 해당 코드를 붙여서 검색해보니 xmlrpc에서 발생하는 오류라고 합니다.

따라서 파이썬에서 제공하는 xmlrpc를 이용하여 연결해 봅니다.

# 문제풀이

xmlrpc 모듈을 이용하여 연결을 해본 결과 다음과 같이 나옵니다.

```python
# http://www.pythonchallenge.com/pc/return/disproportional.html

import xmlrpc.client

# XML 서버와 프록시 연결
server = xmlrpc.client.ServerProxy("http://www.pythonchallenge.com/pc/phonebook.php")

# XML 서버에서 사용가능한 Method들 
server.system.listMethods()
# ['phone', 'system.listMethods', 'system.methodHelp', 'system.methodSignature', 'system.multicall', 'system.getCapabilities']

```

프록시 서버로 연결 후, 사용할 수 있는 명령어들을 출력시켜 보니 'phone'이라는 명령어가 있습니다. 이 메소드가 무슨 기능을 하는지 알아보기 위해 system.methodHelp()를 실행시켜 봅니다.

```python
# Method에 대한 설명
>>> server.system.methodHelp('phone')
'Returns the phone of a person'
```

설명에 따르면 **phone**메소드는 사람이름에 따른 전화번호를 주는 것 같습니다.

숫자 **1**을 입력해서 phone을 메소드를 호출하면 다음과 같은 결과가 나옵니다.

```python
>>> server.phone("1")
'He is not the evil'
```

evil의 이름을 알아서 phone 메소드를 호출 하는 것 같습니다. evil의 이름에 대한 힌트는 13번 문제가 아닌, 12번 문제에 있습니다. 

12번 문제에서 evil1.jpg, evil2.jpg, evil3.jpg, evil4.jpg 까지의 파일이 있었으며 마지막 evil4.jpg 파일은 정상적으로 그림이 출력되지 않았었습니다.

이를 확인해보면 문자열이 들어있는 파일입니다.

![image](https://user-images.githubusercontent.com/33647663/159637084-3a1468c2-decf-4eae-9dd8-2195a1dc8e82.png)

evil의 이름은 Bert 라고 합니다.!

```python
>>> server.phone("Bert")
'555-ITALY'
```

정답인 555-ITALY 가 출력되었으며 실제 정답은 **ITALY**입니다.

# 다음 문제
[http://www.pythonchallenge.com/pc/return/italy.html](http://www.pythonchallenge.com/pc/return/italy.html){:target="_blank"}


