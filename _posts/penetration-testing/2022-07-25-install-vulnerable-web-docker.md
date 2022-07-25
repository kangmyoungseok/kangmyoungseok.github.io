---
title: "[Penetraion-Testing] Install Vulnerable WEB with Docker"
categories:
  - penetration-testing
tags:
  - penetration-testing
  - docker
  - dvwa
  - bwapp
  - dvws
  - webgoat
toc: true
toc_sticky: true
toc_label: "Install Vulnerable WEB"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/Penetration-Testing.jpg
---

💡 웹 모의해킹 연습을 위한 취약한 웹 어플리케이션 설치 방법에 대한 포스팅입니다.
{: .notice--warning}

# DVWA

```sh
docker run -d -p 8080:80 --name dvwa citizenstig/dvwa
```

**localhost:8080**로 접속

![image](https://user-images.githubusercontent.com/33647663/180678479-6b3ec1ef-084d-4667-ae24-e4f895b5b854.png)

- Create / Reset Database 클릭
- admin / password로 로그인 

![image](https://user-images.githubusercontent.com/33647663/180680269-600b9221-a75e-49e0-8192-270c9bb72f76.png)

# bwapp

```sh
docker run -d -p 8081:80 --name bwapp raesene/bwapp
```

**localhost:8081/install.php**로 접속

![image](https://user-images.githubusercontent.com/33647663/180680039-a4438bc5-48b7-49e7-b19f-a62fe163b959.png)

- click here을 눌러서 설치

**localhost:8081**로 접속

- bee/bug 로 로그인

![image](https://user-images.githubusercontent.com/33647663/180680164-f0d4d26e-cb73-4bd9-b5b5-941a0891778e.png)


# dvws

```sh
docker run -d -p 8082:80 --name dvws cyrivs89/web-dvws
```

**localhost:8082/dvws/instructions.php**로 접속

![image](https://user-images.githubusercontent.com/33647663/180680825-6cca49fd-2225-4e96-afab-25796e841dc6.png)

- Reset Database

# webgoat

```sh
docker run -d -p 8083:8080 --name webgoat webgoat/webgoat-7.1
```

**localhost:8083/WebGoat/login.mvc**로 접속

![image](https://user-images.githubusercontent.com/33647663/180692987-2ca38af9-3d3f-464a-b356-cfa3366e1090.png)