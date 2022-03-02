---
title: "[pythonchallenge] 1번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - 1번
toc: true
toc_sticky: true
toc_label: "1번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 1번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/map.html](http://www.pythonchallenge.com/pc/def/map.html){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/154644171-c7eb6f54-bf51-4b71-a198-3c2dce5fd8e0.png)


# 문제 풀이
이번 문제는 카이사르 암호(시저 암호)를 이용하는 문제입니다. 카이사르 암호는 문자를 몇 칸씩 옆으로 밀어서 암호화를 하는 방식입니다.

해당 사진에서는 각각의 문자들을 오른쪽으로 2칸씩 밀어준 것을 알 수 있습니다. 문자들에 대해서 오른쪽으로 2칸씩 밀어주어서 문제에 주어진 문자열을 읽습니다.

```python
string = "g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj."


def shift_right(text,num):
    answer = ''
    for character in text:
        if(character.isalpha()):
            next_char = chr(ord(character)+num)
            if(next_char.isalpha() == False):
                next_char = chr(ord(next_char)-26)
            answer += next_char
        else:
            answer += character
        
    return answer

print(shift_right(string,2))
# i hope you didnt translate it by hand. thats what computers are for. doing it in by hand is inefficient and that's why this text is so long. using string.maketrans() is recommended. now apply on the url.
```

문자를 해독하면 평문이 나오며, maketrans() 함수를 이용하는 것을 추천한다는 내용과, url에 대해서도 적용하라고 나옵니다. 

maketrans() 함수를 이용해서 동일한 기능을 구현하여 url에 대해서 암호화 해줍니다.

```python
before_string = 'abcdefghijklmnopqrstuvwxyz'
after_string = 'cdefghijklmnopqrstuvwxyzab'
transtab = string.maketrans(before_string,after_string)

string.translate(transtab)

string = 'map'
shift_right(string,2)
# ocr
```

문제의 url에서 `map` 을 `ocr` 로 바꿔서 접속해 줍니다.




# 다음 문제

[http://www.pythonchallenge.com/pc/def/ocr.html](http://www.pythonchallenge.com/pc/def/ocr.html){:target="_blank"}


