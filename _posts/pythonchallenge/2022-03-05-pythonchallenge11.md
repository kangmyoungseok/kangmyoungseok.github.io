---
title: "[pythonchallenge] 11번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - PIL
  - Image

toc: true
toc_sticky: true
toc_label: "11번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 11번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/return/5808.html](http://www.pythonchallenge.com/pc/return/5808.html){:target="_blank"}
	+ id: huge
	+ pw: file

# 문제
![image](https://user-images.githubusercontent.com/33647663/156870008-f2fa9901-9286-4394-a430-458c390e87ce.png)

문제를 보면 그림 이외에 힌트가 되는 부분은 제목이 `odd even` 입니다. 홀수 짝수라는 의미인데, 혹시 픽셀 단위에서 이미지가 홀짝으로 분리되지 않을지 접근을 해봅니다.

# 문제 풀이
이미지를 열고, y 값을 1씩 증가시켜보며, y가 홀,짝 일때 값이 차이가 나는지 확인해 봅니다.
```py
img = Image.open("cave.jpg")

# (x,y) 에서 y가 짝수인 애들과 홀수인애들 차이
for i in range(0,10,2):
    print([0,i],img.getpixel((0,i)),[0,i+1],img.getpixel((0,i+1)))
```

실행 결과는 다음과 같습니다. 결과를 보니 확실히 다릅니다. 

```py
[0, 0] (0, 20, 0) [0, 1] (148, 186, 111)
[0, 2] (0, 20, 0) [0, 3] (145, 182, 105)
[0, 4] (0, 24, 0) [0, 5] (156, 190, 114)
[0, 6] (0, 19, 0) [0, 7] (153, 187, 110)
[0, 8] (0, 21, 0) [0, 9] (154, 193, 112)
```

이제 홀 짝이 y 좌표랑만 연관있는지 확인해 보기 위해서 x를 1로 바꾼다음에 확인해 봅니다.

```py
for i in range(0,10,2):
    print([1,i],img.getpixel((1,i)),[1,i+1],img.getpixel((1,i+1)))

# 실행 결과
[1, 0] (142, 180, 105) [1, 1] (0, 20, 0)
[1, 2] (158, 195, 118) [1, 3] (0, 22, 0)
[1, 4] (150, 184, 108) [1, 5] (0, 19, 0)
[1, 6] (155, 189, 112) [1, 7] (0, 18, 0)
[1, 8] (158, 197, 116) [1, 9] (0, 21, 0)
```

실행 결과를 보면 y 값의 홀짝이 아닌, x+y의 값이 홀수 짝수인 경우로 이미지를 분리하면 될 것 같습니다. 다음과 같이 이미지를 분리해 줍니다.

```py
odd_img = Image.new("RGB",(641,481))
even_img = Image.new("RGB",(641,481))
for x in range(width): 
    for y in range(height):
        if( (x+y) % 2 == 0 ):
            even_img.putpixel((x,y), img.getpixel((x,y)))
            even_img.putpixel((x,y+1),img.getpixel((x,y)))
        else:
            odd_img.putpixel((x,y), img.getpixel((x,y)))
            odd_img.putpixel((x,y+1),img.getpixel((x,y)))
```

Image 객체를 만들어 준 뒤, x+y 의 값이 홀 짝인 경우에 대해서 다르게 픽셀을 넣어 줬습니다. 이때, 크기를 맞추기 위해서 2번씩 넣어줬습니다.

실행 결과 이미지를 보면 다음과 같습니다.

[홀수 이미지]
![image](https://user-images.githubusercontent.com/33647663/156871199-da6a6258-aa61-4c8f-8fb2-1543f61e4549.png)

[짝수 이미지]
![image](https://user-images.githubusercontent.com/33647663/156871213-a080d357-d318-4d32-878a-61a89b6b3ce9.png)

짝수 이미지를 보면 오른쪽 상단에 `evil`이라는 글자가 보입니다. 정답은 `evil`입니다.

# 전체 코드
```py
import requests
from PIL import Image

url = "http://www.pythonchallenge.com/pc/return/cave.jpg"
response = requests.get(url,auth=("huge","file"))

with open("cave.jpg","wb") as fw:
    fw.write(response.content)

img = Image.open("cave.jpg")

# (x,y) 에서 y가 짝수인 애들과 홀수인애들 차이
for i in range(0,10,2):
    print([0,i],img.getpixel((0,i)),[0,i+1],img.getpixel((0,i+1)))

for i in range(0,10,2):
    print([1,i],img.getpixel((1,i)),[1,i+1],img.getpixel((1,i+1)))

# x,y 의 합이 홀수인 픽셀과 짝수인 픽셀을 모아보자.

width,height = img.size
# (640,480)

odd_img = Image.new("RGB",(641,481))
even_img = Image.new("RGB",(641,481))
for x in range(width): 
    for y in range(height):
        if( (x+y) % 2 == 0 ):
            even_img.putpixel((x,y), img.getpixel((x,y)))
            even_img.putpixel((x,y+1),img.getpixel((x,y)))
        else:
            odd_img.putpixel((x,y), img.getpixel((x,y)))
            odd_img.putpixel((x,y+1),img.getpixel((x,y)))
odd_img.show()
even_img.show()    
```

# 다음 문제
[http://www.pythonchallenge.com/pc/return/evil.html](http://www.pythonchallenge.com/pc/return/evil.html){:target="_blank"}
