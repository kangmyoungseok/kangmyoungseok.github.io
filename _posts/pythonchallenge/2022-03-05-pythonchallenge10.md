---
title: "[pythonchallenge] 10번 문제 풀이"
categories:
  - pythonchallenge
tags:
  - pythonchallenge
  - pyhon
  - Look and say sequence
  - 1, 11, 21, 1211, 111221, 


toc: true
toc_sticky: true
toc_label: "10번 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/pythonchallenge.jpg
---

💡 pythonchallenge.com 10번 문제에 대한 풀이입니다.
{: .notice--warning}


- Link : [http://www.pythonchallenge.com/pc/return/bull.html](http://www.pythonchallenge.com/pc/return/bull.html){:target="_blank"}
	+ id: huge
	+ pw: file

# 문제
![image](https://user-images.githubusercontent.com/33647663/156869713-d98bb4d5-837e-4a91-9cc7-6b0c444740c9.png)

화면 하단의 힌트를 보면 `len(a[30])`이 얼마인지 묻고 있습니다.

소를 클릭하면 다음과 같이 수열의 형태가 나옵니다.

![image](https://user-images.githubusercontent.com/33647663/156869734-62774f50-d60a-40d1-a0d3-37452968f5ad.png)

이번 문제는 해당 수열의 a[30] 원소를 구한다음 길이를 구하는 문제라고 추측됩니다.

# 문제 풀이
우선 바로 풀어도 되지만, 수열 같은건 구글에 검색하면 바로 답이 나오기 때문에 ^^... 구글의 힘을 얻어봅니다.

![image](https://user-images.githubusercontent.com/33647663/156869793-32a7f2b5-f05a-488d-b189-a666bc289b7e.png)

해당 수열은 읽고 말하기 수열이라고 합니다. 그리고 [위키](https://ko.wikipedia.org/wiki/%EC%9D%BD%EA%B3%A0_%EB%A7%90%ED%95%98%EA%B8%B0_%EC%88%98%EC%97%B4)에 들어가서, 확인해본 결과 규칙을 알 수 있었습니다.

해당 수열은 다음과 같이 생성됩니다.

1      -> 1개의 1 (11)<br>
11     -> 2개의 1 (21)<br>
21     -> 1개의 2, 1개의 1 (1211)<br>
1211   -> 1개의 1, 1개의 2, 2개의 1 (111221)<br>
111221 -> 3개의 1, 2개의 2, 1개의 1 (312211)<br>
...

이제 로직을 알았으니, 해당 로직을 바탕으로 수열을 생성하는 파이썬 코드를 작성합니다. 코드는 아래에 있습니다.

코드 실행결과 a[30]의 길이는 5808 입니다.

# 전체 코드
```py
def ls_sequence(n):
    sequence = [1]
    r = 0 # rotate
    while r < n-1:
        string = str(sequence[r])        
        idx = 0
        result = ''
        while idx < len(string):
            # idx가 가장 마지막인 경우 비교할 대상 없이 끝낸다.
            if(idx) == len(string)-1:
                result = result +  str(1) + string[idx] 
                break
            # 먼저 현재 인덱스와 같은 숫자의 개수 구한다.
            count = 1
            while string[idx] == string[idx + 1]:
                count += 1
                idx +=1
                if(idx == len(string)-1):
                    break
            # 갯수 먼저 쓰고, 뒤에 숫자 쓴다.
            result = result +  (str(count) + string[idx])
            idx +=1
        sequence.append(int(result))
        r = r+1
    
    return sequence

print(len(str(ls_sequence(31)[-1])))
#5808
```

# 다음 문제
[http://www.pythonchallenge.com/pc/return/5808.html](http://www.pythonchallenge.com/pc/return/5808.html){:target="_blank"}
