---
title: "[AWS] EC2 하드 용량 늘리기"
categories:
  - cloud
tags:
  - aws
  - ec2
  - Ubuntu
  - EBS
  - OpenSSH Client
  - Putty
toc: true
toc_sticky: true
toc_label: "SSH 연결하기"
toc_icon: "bookmark"
author_profile: true
---

💡 AWS EC2에서 하드디스크 용량을 증설하는 방법에 대한 포스팅입니다.
{: .notice--warning}

# Introduction
 AWS로 작업을 하다보면 하드디스크 용량이 부족해 지는 경우가 있습니다. 이때, 볼륨을 늘리는 방법에 대해서 포스팅을 해보겠 습니다.

 볼륨을 늘리는 것에는 2가지 방법이 있습니다. 하나는 기존에 사용하던 볼륨이 8기가 라면 16기가로 볼륨을 수정하는 방법이 있고, 다른 방법은 8기가에 새로운 8기가를 더하는 방법이 있습니다.


## 볼륨 수정
 EC2를 생성하면 다음과 같이 볼륨이 생성되었을 것입니다. 

 ![image](https://user-images.githubusercontent.com/33647663/149194234-ba9ddfab-00b9-4041-ba0d-c29f4170713c.png)

 수정을 원하는 볼륨을 선택 후, 작업 > 볼륨 수정을 눌러 줍니다.

 ![image](https://user-images.githubusercontent.com/33647663/149194480-6d07839c-0ef9-45b0-8306-1947a3d63b3d.png)

 해당 페이지에서 볼륨의 크기를 16기가로 바꿔줍니다

 ![image](https://user-images.githubusercontent.com/33647663/149194567-35b22c28-8128-4f10-bd9d-268830d2ff16.png)

 수정은 EC2를 끄지 않아도 가능하지만 **적용이 되기 위해서는 재부팅이 필요**합니다.

## 볼륨 추가
 이번엔 독립적인 볼륨을 추가하는 방법으로 하드를 늘려보겠습니다. 다시 볼륨 대시보드로 이동합니다.
 이때, **가용 영역** 부분을 기억하고 **볼륨 생성**을 눌러 줍니다. 

 ![image](https://user-images.githubusercontent.com/33647663/149195230-43fcf4b4-ddea-45ff-8ffe-147db27c84e7.png)
 
 추가하고자 하는 용량과, 가용 영역을 맞춰준다음 볼륨을 생성해 줍니다.

 ![image](https://user-images.githubusercontent.com/33647663/149196297-2b53250f-478c-4b40-ac89-284ab869408e.png)

 생성된 볼륨을 EC2와 연결하기 위해서 **볼륨 연결**을 눌러줍니다

 ![image](https://user-images.githubusercontent.com/33647663/149196609-b6dc9bf5-4f28-40d1-a870-734857006bf4.png)


 볼륨을 더하고자 하는 인스턴스를 선택하고 **볼륨 연결**을 눌러 줍니다.

 ![image](https://user-images.githubusercontent.com/33647663/149196754-e33add1e-b842-40df-bdf3-886e6462709e.png)

 ![image](https://user-images.githubusercontent.com/33647663/149196801-01821f21-ed5e-41f5-a5cb-7b9b5ca07527.png)


## 볼륨 적용
 [볼륨 수정](#볼륨-수정)과 [볼륨 추가](#볼륨-추가)에서 바뀐 내용이 EC2에 적용 되도록 EC2를 재부팅 해줍니다.

 재부팅 후 디스크 용량을 확인해 보면 다음과 같습니다.

 ![image](https://user-images.githubusercontent.com/33647663/149197837-f8291161-680e-41f0-ba2f-0f6a39b1f8a1.png)

 [볼륨 수정](#볼륨-수정)으로 16기가로 변경한 부분은 성공적으로 반영되었으나, [볼륨 추가](#볼륨-추가)로 더해준 부분은 반영되지 않았습니다. 그 이유는 볼륨 추가는 아예 새로운 하드디스크를 서버에 하나 더 붙이는 개념으로 이를 리눅스에 반영되게 하기 위해서는 마운트를 해줘야 합니다.

 물리적 디스크를 확인하는 명령어인 ```fdisk -l```을 입력해서 디바이스 정보를 확인합니다.

 ```bash
 fdisk -l
 ```
 
 결과를 보면 /dev/xvda가 현재 사용하고 있는 16기가 장치이고, /dev/xvdf가 새로 추가한 볼륨으로 사용되고 있지 않습니다. 

 ![image](https://user-images.githubusercontent.com/33647663/149198386-f9affb9b-6822-4547-a079-9c6477837f95.png)


## 마운트

### 1. 파티션 설정
 ```bash
 fdisk /dev/xvdf
 n
 [default] 값
 w
 ```

 ![image](https://user-images.githubusercontent.com/33647663/149199193-a64434ea-6b4a-4755-ae8b-0ca43068c528.png)

 다시 fdisk -l 로 확인을 해보면 다음과 같이 /dev/xvdf1파티션이 생겼습니다.

 ![image](https://user-images.githubusercontent.com/33647663/149199478-0c01a5ef-eb1d-4ab5-bea7-541f24a5d739.png)


### 2. 파일시스템 설정

 ext4 방식으로 포맷합니다.

 ```bash
 mkfs -t ext4 /dev/xvdf1
 ```

 ![image](https://user-images.githubusercontent.com/33647663/149199706-ceaf60eb-4f8f-41e0-bacb-1f44058149e9.png)


### 3. 디렉터리로 마운드
 대상 폴더 : /volume

 ```bash
 mkdir /volume
 mount /dev/xvdf1 /volume
 df -h
 ```
 다음과 같이 /volume 디렉터리로 4기가바이트의 장치가 마운트 되었음을 볼 수 있습니다.

 ![image](https://user-images.githubusercontent.com/33647663/149200075-d0e28c2a-bc03-4c19-a2d9-60da32e4cecf.png)

### 4. /etc/fstab
 현재 명령어로의 마운트는 시스템이 재부팅 되면 사라집니다. /etc/fstab 에는 부팅시 마운트 정보를 읽어들여서 자동으로 마운트 해줍니다.

 ```md
 /dev/xvdf1     /volume      defaults    0  0
 ```

 ![image](https://user-images.githubusercontent.com/33647663/149200701-ca5bc5e7-3cd6-4631-abe8-7c8bf05e84e4.png)

 