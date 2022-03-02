---
title: "[pythonchallenge] 7번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - PIL


toc: true
toc_sticky: true
toc_label: "7번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 7번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/def/oxygen.html](http://www.pythonchallenge.com/pc/def/oxygen.html){:target="_blank"}

# 문제
![image](https://user-images.githubusercontent.com/33647663/156351262-042356c8-8137-45b0-bd21-a51aa8b010e9.png){: .align-center}

# 문제 풀이
이번 문제는 소스코드에 힌트도 없이 문제에 있는 사진이 전부입니다. 사진을 보면, 검정과 회색줄로 쭉 그어져 있는 부분이 있습니다. 이미지 분석 라이브러리인 PIL(Python Imaging Library)을 이용하여 분석해 봅니다.

우선 그림에서 회색으로 칠해져 있는 부분을 찾기 위해서 이미지의 첫 열들을 모두 출력해보면 다른 픽셀들과 다르게 연속적으로(155,155,155)인 부분이 있습니다.

```py
img = Image.open("oxygen.png")

width,height = img.size

# img[x][y]
for y in range(height):
        print(y,img.getpixel((0,y)))
```

![image](https://user-images.githubusercontent.com/33647663/156354689-a13c5be9-3e66-4ee8-a7f6-7a1c1ad1b80d.png){: .align-center}

회색이 있는 부분은 43-51번째 행인것을 알 수 있습니다. 그 중 45번째 행을 모두 출력해봅니다.

```py
for x in range(width):
    print(x,img.getpixel((x,45)))
```

![image](https://user-images.githubusercontent.com/33647663/156354962-4ef52961-4b95-484d-b7ed-8bcf0f7ffa5c.png){: .align-center}

그림을 보면 7번씩 동일한 값이 반복되어 값이 출력됨을 볼 수 있습니다. 숫자의 범위를 보면 ascii로 출력이 가능한 범위임을 알 수 있습니다. 따라서 7번째 숫자마다 ascii로 출력을 해봅니다.


```py
for x in range(0,width,7):
    print(chr(img.getpixel((x,45))[0]),end='')
```

![image](https://user-images.githubusercontent.com/33647663/156355877-45a3b73b-9ad8-41c6-a259-0b0919cc4c27.png){: .align-center}

`smart guy, you made it. the next level is [105, 110, 116, 101, 103, 114, 105, 116, 121]vse`

끝에 있는 `vse` 같은 경우는 이미지 파일을 보면 회색선이 그림의 끝까지 있는게 아닌 마지막부분에서 조금 부족한 것을 알 수 있습니다. 따라서 해당 부분은 이미지의 진짜 픽셀임을 알 수 있습니다. 해당부분을 제외한 메시지에 나와있는 값들도 아스키로 출력을 해보면 답이 나옵니다.

```py
message = [105, 110, 116, 101, 103, 114, 105, 116, 121]
for character in message:
    print(chr(character),end='')
# Integrity
```

다음문제의 답은 `integrity`입니다.

# 전체 코드
```py
from PIL import Image
import requests

url = "http://www.pythonchallenge.com/pc/def/oxygen.png"
response = requests.get(url)

with open("oxygen.png","wb") as fw:
    fw.write(response.content)

img = Image.open("oxygen.png")
#img.show()

width,height = img.size
# (629,95)

# img[x][y]
for y in range(height):
        print(y,img.getpixel((0,y)))


for x in range(width):
    print(x,img.getpixel((x,45)))

for x in range(0,width,7):
    print(chr(img.getpixel((x,45))[0]),end='')
# smart guy, you made it. the next level is [105, 110, 116, 101, 103, 114, 105, 116, 121]vse


message = [105, 110, 116, 101, 103, 114, 105, 116, 121]
for character in message:
    print(chr(character),end='')

# Integrity
```

# 다음 문제
[http://www.pythonchallenge.com/pc/def/integrity.html](http://www.pythonchallenge.com/pc/def/integrity.html){:target="_blank"}

