---
title: "[Linux] Ubuntu 20.04 swap 용량 늘리기 "
categories:
  - linux
tags:
  - linux
  - ubuntu
  - vmware
toc: true
toc_sticky: true
toc_label: "swap 영역 용량 늘리기"
toc_icon: "bookmark"
author_profile: true
---

💡 우분투 20.04 에서 swap 영역을 늘리는 방법에 대한 포스팅입니다.
{: .notice--warning}

# Introduction
 가상머신으로 ubuntu를 사용하면, 메모리가 부족해서 팅기는 경우가 많이 생깁니다. 현재 제 컴퓨터는 가상머신에 메모리를 4기가 정도로 할당 했는데, 메모리가 부족한 경우가 많아서 swap 영역을 사용하려고 합니다.

# [01] 현재 설정된 swap 영역 확인
 ```bash
 free -h
 ```

 기존에는 스왑 영역이 1기가 정도로 설정되어 있었습니다. 이를 8기가로 확장하겠습니다.

 ![image](https://user-images.githubusercontent.com/33647663/152498024-9c35b073-a0c4-4201-a5ac-1b84aaabfb2d.png)

<br><br>

# [02] 현재 스왑영역 삭제 및 새로운 크기로 재 할당
  ```bash
  sudo swapoff -v /swapfile
  sudo rm -f /swapfile
  sudo fallocate -l 8G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  ```

<br><br>

# [03] /etc/fstab 파일 수정
  ```bash
  sudo vi /etc/fstab
  ```

  아래의 내용 파일에 추가

  ```md
  swapfile none swap sw 0 0
  ```

<br><br>

# [04] 변경 사항 확인
  ```bash
  free -h
  ```

  ![image](https://user-images.githubusercontent.com/33647663/152498575-b8aee7cc-1f80-4f83-8655-284d4e4ae38a.png)

