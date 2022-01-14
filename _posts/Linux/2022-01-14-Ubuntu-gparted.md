---
title: "[Linux] Ubuntu vmware 용량 늘리기 "
categories:
  - linux
tags:
  - linux
  - ubuntu
  - vmware
  - gparted
toc: true
toc_sticky: true
toc_label: "ubuntu vmware 용량 늘리기"
toc_icon: "bookmark"
author_profile: true
---

💡 가상머신에서 하드디스크 용량을 추가하는 방법에 대한 포스팅입니다.
{: .notice--warning}

# Introduction
 가상머신을 사용하다 보면, 용량이 부족해지는 경우가 생깁니다. 이 때 기존에 사용하던 루트 파티션의 용량을 증설하는 방법에 대해서 포스팅 해보겠습니다.

# [01] Vmware 설정
 가상머신을 종료한 후에 가상머신 설정 값을 변경해 줍니다.

 ![image](https://user-images.githubusercontent.com/33647663/149458509-82ef30f2-7ef8-4110-9d27-67a11e48e2c3.png)

 **Hard Disk**를 선택하고 Expand > 원하는 만큼 하드를 늘려줍니다.

 ![image](https://user-images.githubusercontent.com/33647663/149458434-ac6db743-2717-4af1-b113-fd2434395de4.png)

 이렇게 설정하면 물리적으로는 하드디스크의 크기가 증가되었지만, 실제 운영체제에 적용하기 위해서는 논리적으로 파일시스템 크기를 조정해 줘야 합니다.

# [02] Gparted 설정
 우분투로 접속한 다음 ```df -h```를 입력해서 용량이 변했는지 확인 해 줍니다. 35기가로 용량을 변경해 줬지만 아직 반영되지 않은 것을 볼 수 있습니다.

 ![image](https://user-images.githubusercontent.com/33647663/149458877-9e8ea71f-e261-475d-8890-3491d2312d19.png)

 다음 명령어를 입력해서 gparted를 설치해 줍니다.
 ```bash
 sudo apt-get install -y gparted
 ```

 설치가 되었으면 gparted를 실행해 줍니다.
 ```bash
 sudo gparted
 ```
 
 실행이 되면 다음과 같이 unallocated 파티션이 5기가 늘어난 것을 볼 수 있습니다. 

 ![image](https://user-images.githubusercontent.com/33647663/149459033-f6873bad-d3ca-454d-b6ab-5b2a5c691686.png)

 저는 확장 파티션을 사용하고 있기 때문에 /dev/sda5를 늘려주기 위해서 우선 /dev/sda2 확장 파티션의 크기를 늘려줍니다.

 /dev/sda2에서 마우스 우클릭 후 **Resize/Move**를 눌러줍니다.

 ![image](https://user-images.githubusercontent.com/33647663/149459341-9559bfaa-811d-4010-8a34-a6de12b3f1a7.png)

 다음과 같이 화면이 나오면 마우스로 눌러서 확장 시켜 주면 됩니다.

 ![image](https://user-images.githubusercontent.com/33647663/149459517-e4ce70ac-2850-43d5-8039-1b5fb809c6af.png)

 ![image](https://user-images.githubusercontent.com/33647663/149459561-14cfc314-4895-4baa-b98c-069c670e8e76.png)

 저장을 하면 다음과 같이 할당되지 않은 영역이 /dev/sda2 확장 파티션 안으로 들어간 것을 볼 수 있습니다.

 ![image](https://user-images.githubusercontent.com/33647663/149459609-60497b01-461f-4cba-af5c-69fdd84905b7.png)

 이제 루트 파티션인 /dev/sda5를 같은 방식으로 확장시켜 줍니다.

 ![image](https://user-images.githubusercontent.com/33647663/149459705-f41167c4-3e4f-4244-b227-41e1f4c5ecf7.png)

 저장 버튼을 누른 후, Apply를 선택합니다.

 ![image](https://user-images.githubusercontent.com/33647663/149459811-7602ee7b-f1ac-4fd0-a1b4-0410b3f51d09.png)

 적용이 되었는지 확인을 해보면 성공적으로 적용이 된 것을 볼 수 있습니다.

 ![image](https://user-images.githubusercontent.com/33647663/149459910-0f04eba1-4c6d-46ed-a7d3-53f2476b4225.png)

# Reference
 - https://miiingo.tistory.com/221
 - https://cragy0516.github.io/Expand-Hard-Disk-in-VMWare/