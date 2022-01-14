---
title: "[VSCode] C++ 라이브러리 링킹 오류 처리 "
categories:
  - error
tags:
  - vscode
  - library
  - tasks.json
---


# Introduction
 VSCode로 C++ 코딩을 하다가 라이브러리 링크를 해야 되는데 어디에서 해줘야 하는지 몰랐다.

 ![image](https://user-images.githubusercontent.com/33647663/149487983-52117a71-f640-4709-827f-11a12da52a03.png)
 
 오류를 보니 pthread 라이브러리를 못찾는 것인데, pthread 라이브러리는 컴파일 때 링킹을 해주면 된다.

 터미널로 하면 
 ```bash
 g++ thread1.cpp -o thread1 -l pthread
 ```

 VScode에서는 컴파일 시 옵션을 tasks.json에서 넣어줄 수 있어서 그 부분을 봤더니 아래 빨간 부분이 컴파일 옵션인거 같아서 아래에 같은 문법으로 추가해 줬다.

 ![image](https://user-images.githubusercontent.com/33647663/149488554-bd58a411-16eb-46ce-9adb-b858c50dc5d8.png)

 ![image](https://user-images.githubusercontent.com/33647663/149488688-5240fce7-670a-4204-89ce-085e35b44a08.png)

 추가한 후 빌드를 해줬더니 의도한 대로 ```-l pthread``` 옵션이 더해지면서 성공적으로 빌드가 되었다.

 ![image](https://user-images.githubusercontent.com/33647663/149488778-6e76d67f-e671-4c32-be2e-4e15f156420b.png)


