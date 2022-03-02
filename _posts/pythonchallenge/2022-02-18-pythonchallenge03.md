---
title: "[pythonchallenge] 3번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - python re
  - re
  - import re
  - regular expression
  - 3번
toc: true
toc_sticky: true
toc_label: "3번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 3번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/equality.html](http://www.pythonchallenge.com/pc/def/equality.html){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/154647344-82a09cbb-56c2-4ec7-b5e7-de8750adbda8.png)

이전 문제와 같이 `ctrl + u` 를 눌러서 소스 코드를 봅니다.

![image](https://user-images.githubusercontent.com/33647663/154647729-52abd96e-c79a-4115-a065-283dd6bf1fb3.png)

문제를 다시보면 다음과 같이 해석됩니다.

```
**정확히** 3개의 대문자가 양옆에 있는 소문자.
```



# 문제 풀이
정규 표현식으로 위의 조건을 표현해 줍니다. 

```python
import re

# 문제의 내용을 정규 표현식으로 표현
regular_expression = '[a-z][A-Z]{3}[a-z][A-Z]{3}[a-z]'
string = '''
주석에 있는 문자
'''

result  = re.findall(regular_expression,string)
print(result)
# ['qIQNlQSLi', 'eOEKiVEYj', 'aZADnMCZq', 'bZUTkLYNg', 'uCNDeHSBj', 'kOIXdKBFh', 'dXJVlGZVm', 'gZAGiLQZx', 'vCJAsACFl', 'qKWGtIDCj']

# 찾은 문자열들에서 4번째에 있는 소문자들만 추출
answer = ''
for res in result:
    answer += res[4:5]

print(answer)
# linkedlist
```



# 다음 문제



[http://www.pythonchallenge.com/pc/def/linkedlist.html](http://www.pythonchallenge.com/pc/def/linkedlist.html){:target="_blank"}

