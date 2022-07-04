---
title: "[AWS] EC2에서 Ubuntu 18.04 설치"
categories:
  - cloud
tags:
  - aws
  - ec2
  - Ubuntu
toc: true
toc_sticky: true
toc_label: "EC2 설치하기"
toc_icon: "bookmark"
author_profile: true
---

💡 AWS EC2에서 Ubuntu 18.04 LTS 설치하는 방법에 대한 포스팅입니다.
{: .notice--warning}

<br>

# EC2 생성

- EC2 대시보드 -> 인스턴스 시작

![image](https://user-images.githubusercontent.com/33647663/177082270-82a1ebff-9266-4562-ab35-02d4335645b1.png)


## 이름 및 태그
- 알맞게 이름 넣어주기

![image](https://user-images.githubusercontent.com/33647663/177082369-3213c131-edf4-4677-ba82-ca48aa8c134d.png)

## 애플리케이션 및 OS 이미지
- Ubuntu -> 18.04 LTS(프리티어) 선택

![image](https://user-images.githubusercontent.com/33647663/177082490-af7e7475-4120-4491-a1ee-fe0f4b2ead1f.png)

## 인스턴스 유형
- t2.micro

![image](https://user-images.githubusercontent.com/33647663/177082575-231fd1a0-b481-4ca4-9a5b-6fc69f3f879c.png)


## 키페어 생성

![image](https://user-images.githubusercontent.com/33647663/177082633-bbd23b12-62ad-4714-85ae-c463e321a8dd.png)

webhell.pem 키 파일이 다운받아지면, 해당 파일을 다음의 경로에 위치시켜 둔다.

- 폴더가 없으면 .ssh 폴더 생성해서 위치시켜 주기
- C:\Users\CAU\\.ssh\

![image](https://user-images.githubusercontent.com/33647663/177082816-dcab72ec-12ba-474c-aafe-98086444a9db.png)

## 네트워크 설정

- 인터넷에서 HTTP 트래픽 허용 체크

![image](https://user-images.githubusercontent.com/33647663/177082924-a3293098-fb6b-4818-9c86-508b58ddcaae.png)

## 스토리지 구성
- 기존 8GB 에서 16GB로 추가

![image](https://user-images.githubusercontent.com/33647663/177083013-89a0df6f-f372-43dd-bd3b-641400053db1.png)


# VSCode 연동

ec2를 실행하면 다음과 같이 대시보드에 실행 중으로 표시

![image](https://user-images.githubusercontent.com/33647663/177084118-d3a78b11-a73f-4eb3-a0fd-1c070c56fbbc.png)

우리는 개발할 때 해당 인스턴스로의 접근을 vscode를 이용해서 할 것임

우선 VSCode 열기

- 확장 > remote 검색 > Remote - SSH 설치

![image](https://user-images.githubusercontent.com/33647663/177084289-b234f593-e6a3-4373-aff4-e886e1de7822.png)

