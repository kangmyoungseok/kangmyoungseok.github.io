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

- 왼쪽 원격탐색기 > 구성(톱니바퀴 설정모양) > .ssh\config 선택

![image](https://user-images.githubusercontent.com/33647663/177084788-d8c0a76b-ff0d-4196-9315-8e6837b82213.png)

- 다음과 같이 작성. 세부 작성 방법은 그 아래 참고

![image](https://user-images.githubusercontent.com/33647663/177085227-752bf143-11ed-411c-bcbd-bd5e91d68373.png)


- Host : vscode에서 표시할 이름. webhell로 작성
- HostName : 클라우드 IP 또는 호스트 이름. 아래에서 퍼블릭 DNS 참고
- User : ubuntu
- IdentityFile : 인스턴스 만들때 저장했던 키 파일. (.ssh\밑 경로에 저장)

---
- HostName 찾는 방법
- 인스턴스 ID 선택 > 연결 > SSH 클라이언트 > 4. 퍼블릭 DNS 부분 복사
![image](https://user-images.githubusercontent.com/33647663/177084924-c68a7f32-8424-474c-ba8a-bed4b12d674a.png)

![image](https://user-images.githubusercontent.com/33647663/177084993-1eb27046-b911-4a2d-9cbd-b088e738641c.png)

![image](https://user-images.githubusercontent.com/33647663/177085019-821a9817-b1db-4aff-8baa-80d06351ca5b.png)


잘 설정했으면 다음과 같이 연결 가능 대상이 생김. 가장 오른쪽 창 추가 버튼을 누르면 다음과 같이 연결된다

![image](https://user-images.githubusercontent.com/33647663/177085467-1d4cde67-7571-41f9-8675-cd5e31665a57.png)


- Linux > 계속  선택

- 기다리고 창이 전부 로드되면
- 터미널 > 새 터미널

![image](https://user-images.githubusercontent.com/33647663/177085634-dc55d546-4283-41e3-b387-5538e551767d.png)

```
여기까지 AWS 설치 및 연동 끝
```
---


# 클라우드에 APM 환경 설정

## APM 설치
APM?
- Apache
- PHP
- Mysql


### 저장소 업데이트
```bash
# 등록된 저장소 내 패키지 정보를 최신으로 업데이트 한다.
$sudo apt update 

# 최신으로 업데이트 된 저장소 내 패키지 정보를 바탕으로 시스템에 설치된 패키지들을 업그레이드 해준다.
$sudo apt upgrade -y

```

### Apache 설치
```sh
# apache2를 설치 한다.
$sudo apt install apache2 -y

# 아파치 서버 시작
$sudo service apache2 start

```

- 설치가 되면 클라우드로 접속해서 확인
- Hostname이나 IP주소로 접근가능

- 접속 주소는 SSH 연결할때 사용한 Hostname
- Hostname에서 IP주소의 추출은 다음과 같이 하면 된다.
- http://ec2-**54-144-125-125**.compute-1.amazonaws.com/ : **54.144.125.125**

![image](https://user-images.githubusercontent.com/33647663/177086297-4b5f25db-cc8e-47bc-a559-c994ee453665.png)


### PHP 설치
- PHP 설치는 버전 정보 꼭 맞춰야 함.
- 버전마다 함수이름이랑, 문법이 바뀌기 때문에 자신이 개발했을 때 PHP 7인지 8인지 확인하고 설치해야 함

- XAMPP로 잘 따라왔으면 아마 PHP 8일 거임. 다음 화면을 보면 우리는 PHP 8.1.6을 설치했으므로 Ubuntu에도 8.1을 설치

![image](https://user-images.githubusercontent.com/33647663/177088005-83685c15-1756-4a68-bd66-9754699bd6d6.png)



```bash
$sudo apt update
$sudo apt install lsb-release ca-certificates apt-transport-https software-properties-common -y
$sudo add-apt-repository ppa:ondrej/php
```

```sh
$sudo apt update
```

```sh
$sudo apt install php8.1 -y
```

- **PHP 설치 확인**

xampp는 htdocs/ 폴더에서 php파일을 처리 했지만, Apahce에서는 /var/www/html/ 폴더에서 php등의 웹사이트 파일을 처리한다.

우선 ubuntu 사용자로 파일을 생성할 수 있도록 다음과 같이 권한 설정을 해준다.

```sh
$cd /var/www/html
$sudo chmod 777 /var/www/html
```

- /var/www/html/index.php 파일 생성

```sh
$vi index.php
```

파일안에 다음과 같이 작성

![image](https://user-images.githubusercontent.com/33647663/177092795-b2cef7d9-d9b4-418c-bdbe-dd61da7e755e.png)

```sh
# 기존 index.html 파일 삭제
$rm -f index.html
```

- IP주소를 이용해서 다시 접근해 보면 다음과 같이 PHP 설치 정보 출력
- PHP 8.1.7 설치 확인

![image](https://user-images.githubusercontent.com/33647663/177093382-bb82814a-136f-4918-a31d-baf1067f879a.png)

### mysql 설치

```sh
# Mysql 설치
$sudo apt install mysql-server -y

$sudo ufw allow mysql
$sudo systemctl start mysql
$sudo systemctl enable mysql

```


```sh
현재 우분투 로그인 계정이 ubuntu인데, 비밀번호 설정이 되어있지 않아서 mysql 접속이 안된다.
우분투 비밀번호를 설정해 준다.

$sudo passwd ubuntu
Enter new UNIX password:  ubuntu 
Retype new UNIX password:  ubuntu
passwd: password updated successfully
```

- mysql 접속 확인
``` sh
$sudo mysql -u root -p
Enter password : ubuntu
```

![image](https://user-images.githubusercontent.com/33647663/177092270-efcd3a79-cc7a-4f37-ad3b-dbb15b44a738.png)


```md
여기까지가 원격 서버에서 웹서버를 호스팅할 준비가 완료됨.
즉, local에서 작업했던 xampp환경을 리눅스에서 그대로 구현한 것
이제 github에 업로드 했던 **htdocs/** 폴더 내용을 우분투에서
**/var/www/html/** 폴더로 git clone하여 확인
```


# 작업했던 웹사이트 파일을 다운
- /var/www/html/ 폴더를 비우기

![image](https://user-images.githubusercontent.com/33647663/177093878-d8032d6f-8757-4c1c-9507-c01cc4fef3f8.png)

## git pull

- 자신의 레포지토리에서
- Code > HTTPS > 복사

![image](https://user-images.githubusercontent.com/33647663/177094477-bc1f4073-40ea-4034-bfea-84696d2e3a4a.png)


```sh
$git pull https://github.com/WebH3ll/myoungseok.git
```


- 다음과 같이 기존에 깃허브에 업로드했던 파일들이 다운로드 된다.

![image](https://user-images.githubusercontent.com/33647663/177099002-379b0090-ce2c-47fc-9aff-31ef37cbd048.png)

