---
title: "[pythonchallenge] 12번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - Hdx
  - png
  - jpg
  - signature


toc: true
toc_sticky: true
toc_label: "12번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 12번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/return/evil.html](http://www.pythonchallenge.com/pc/return/evil.html){:target="_blank"}
	+ id: huge
	+ pw: file

# 문제
![image](https://user-images.githubusercontent.com/33647663/156872611-fac2936f-ca80-498c-9a7e-0ec55d4f6c47.png)

소스코드를 보면 이미지 파일의 이름이 `evil1.jpg`입니다.

![image](https://user-images.githubusercontent.com/33647663/156872630-b3158fc2-65c8-48d1-a4e7-11960243dcaf.png)

evil2,evil3 등도 있는지 확인해보면 다음과 같이 나옵니다.


**[evil2]**
![image](https://user-images.githubusercontent.com/33647663/156872652-024fb4e5-7a12-471f-ba1f-6383f2a3a55f.png)

**[evil3]**
![image](https://user-images.githubusercontent.com/33647663/156873196-71c55ca9-b727-4df3-a8b2-d10e79f1ebfb.png)

**[evil4]**
![image](https://user-images.githubusercontent.com/33647663/156873219-f19dcd3a-d39c-4d3a-915d-328ab68a8750.png)

evil4는 이미지가 깨져서 보이길래 소스코드를 보니 다음과 같은 문자열이 숨어 있었습니다.

```Bert is evil! go back!```



# 문제 풀이
`evil2.jpg`로 접근을 해보면 jpg가 아닌, gfx라고 합니다. `evil2.gfx`로 접근을 해보면 gfx파일이 다운로드 됩니다.

gfx 파일은 처음보는 확장자여서 이것저것 구글링도 해보고 관련 프로그램을 다운받아서 열어봤지만, 되지 않았습니다. 그래서 다르게 접근을 해보기 위해서 `Hxd` 프로그램을 이용해서 바이트 값을 봤습니다.

![image](https://user-images.githubusercontent.com/33647663/156872839-c3bfcfc2-e541-4458-a749-6639863ed5af.png)

처음 열어 봤을때는 눈치를 못챘는데, 자세히 보면 특이한게 눈에 띕니다. 파일 시그니처를 잘 모르시는 분은 모르겠지만, jpg나 png 파일을 많이 열어보신 분들은 보일 겁니다. 저는 파일 중간에 있는 JF IF,,, IHDR 과 같은 jpg 파일과 png 파일에서 나오는 시그니처가 눈에 띄었습니다.

![image](https://user-images.githubusercontent.com/33647663/156872902-c1ab90da-b644-4c5b-ac53-0e835937b9ad.png)

파일의 종류를 구분지을 수 있는 파일의 시그니처 정보는 대부분 파일 가장 앞쪽에 나옵니다. 그리고 가장 많이 볼 수 있는 jpg 파일과 png 파일의 시그니처는 다음과 같습니다.

jpg : FF D8 FF E0 / FF D8 FF E1

png :  89 50 4E 47 0D 0A 1A 0A 

위의 정보를 바탕으로 파일을 잘 보면, 5바이트씩 띄어져 있으며, 해당 파일은 5개의 서로다른 확장자의 파일이 저장되어 있음을 알 수 있습니다.

전체 바이트를 5바이트 단위로 5개 파일로 나누는 코드 입니다.

```py
from PIL import Image
from io import BytesIO

s = open("evil2.gfx", "rb").read()
for i in range(5):
    piece = s[i::5]  # every fifth byte, starting at i
    im = Image.open(BytesIO(piece))
    f = open("%d.%s" % (i, im.format.lower()), "wb")
    f.write(piece)
    f.close()
```

총 5개의 파일이 생성되었으며, 다음과 같습니다.

![image](https://user-images.githubusercontent.com/33647663/156873833-ac5734b5-b2b4-43d3-a494-4ac4372b4113.png)

4번째 파일이 잘 안보이는데, 다른 프로그램에서 열어보면 다음과 같이 `ional`이 잘 보입니다.

![image](https://user-images.githubusercontent.com/33647663/156873868-c3cccef2-065c-4578-b3ef-21104fadfa00.png)


정답은 **disproportional** 입니다.


# 다음 문제
[http://www.pythonchallenge.com/pc/return/disproportional.html](http://www.pythonchallenge.com/pc/return/disproportional.html){:target="_blank"}
