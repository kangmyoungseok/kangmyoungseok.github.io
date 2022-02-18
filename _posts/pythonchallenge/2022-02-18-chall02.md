---
title: "[pythonchallenge] 2번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - 2번
toc: true
toc_sticky: true
toc_label: "2번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 2번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/ocr.html](http://www.pythonchallenge.com/pc/def/ocr.html){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/154646593-527ec323-21e9-418e-8db5-19e05bb37a5a.png)


# 문제 풀이
문제 화면에서 페이지 소스를 보라고 나옵니다. `Ctrl + u` 를 눌러서 페이지 소스를 봅니다.

![image](https://user-images.githubusercontent.com/33647663/154646718-729830a7-4182-4a9c-af53-ad8f55ec0db3.png)

주석에서 여러 문자들 중에서 글자를 찾으라고 합니다. 


```python
string = "주석에 있는 문자열 복사"

answer = ''
for character in string:
    if(character.isalpha()):
        answer += character

print(answer)
# equality
```


# 다음 문제

[http://www.pythonchallenge.com/pc/def/equality.html](http://www.pythonchallenge.com/pc/def/equality.html){:target="_blank"}


