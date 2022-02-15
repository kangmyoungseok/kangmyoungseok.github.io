---
title: "[webhacking.kr] Challenge g00gle2 문제 풀이"
categories:
  - webhacking
tags:
  - webhacking.kr
  - wargame
toc: true
toc_sticky: true
toc_label: "g00gle2 문제 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/webhacking.jpg
---

💡 Webhacking.kr challenge g00gle2 문제에 대한 풀이입니다.
{: .notice--warning}

# 문제
  ![image](https://user-images.githubusercontent.com/33647663/152630424-281e44ad-4495-4aed-90fb-0d11350cc626.png)

  구글 스프레드 시트에 다음과 같이 함수가 호출되어서 화면에는 ```FLAG{??????}```만 출력되어 있습니다.


# 문제 풀이
  현재 구글 시트에는 **읽기 전용** 기능이 켜져 있으며, 제 계정으로는 아무런 권한이 없기 때문에 수정을 할 수가 없습니다. A1셀에 적혀있는 함수는 다음과 같습니다.

  ```python
  =REPLACE(IMPORTRANGE("https://docs.google.com/spreadsheets/d/1x7b8Wwp3jJAr6sgEG8RmuZOh-d7gpSmYvH9tmXWDq_c/edit","A1"),6,19,REPT("?",19-6))
  ```

  IMPORTRANGE 함수는 (URL,CELL) 과 같이 부르면 외부의 URL에 있는 데이터를 불러 올 수 있는 기능으로 현재 해당함수를 통해서 ```https://docs.google.com/spreadsheets/d/1x7b8Wwp3jJAr6sgEG8RmuZOh-d7gpSmYvH9tmXWDq_c/edit```경로에 있는 스프레드 시트의 **A1**셀의 데이터를 불러온 것입니다. 

  그 이후 REPLACE 함수를 통해서 6-19번째 문자열들을 **?**로 바꿔서 플래그가 숨겨져 있는 상태입니다.

  물론 저 URL경로를 직접 접근하면 권한이 없기 때문에 들어갈 수 없습니다.

  ![image](https://user-images.githubusercontent.com/33647663/152630527-a8faf26e-2dad-43d8-9491-81741db6eba9.png)

  제가 접근한 방법은 우선 처음 IMPORTRANGE를 호출 한 뒤에, 현재 스프레드시트에서 문자열을 교체했으니, 네트워크 통신 상에서는 분명히 A1셀의 데이터가 평문으로 전달되었을 것이라 생각했습니다. 그래서 네트워크 패킷을 분석했습니다.

  개발자 모드에서 네트워크 패킷을 잡은 다음, A1셀에 들어있는 주소중 일부를 입력하여, 해당 주소로 접근하는 패킷을 먼저 찾았습니다.

  ![image](https://user-images.githubusercontent.com/33647663/152631279-936a75ac-b76d-473a-b7aa-63b19b1eea40.png)

  위의 그림처럼 3개의 패킷에서 발견되었으며, 각각의 패킷을 본 결과 다음과 같이 FetchData 에서 데이터를 평문으로 받아와서 플래그를 찾을 수 있었습니다.

  ![image](https://user-images.githubusercontent.com/33647663/152631323-ee20764c-01ac-429b-b8d7-39cf01dd3603.png)






  