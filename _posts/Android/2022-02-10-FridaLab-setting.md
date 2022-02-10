---
title: "[Frida] FridaLab 실습을 위한 환경 설치"
categories:
  - andriod
tags:
  - reverse engineering
  - Frida
  - Frida labs
  - ADB
  - Android
  - Nox
  - jadx
toc: true
toc_sticky: true
toc_label: "환경 설치"
toc_icon: "bookmark"
author_profile: true
header:
  teaser: /assets/images/frida.png
---

💡 Frida 실습을 위한 환경 설치에 대한 포스팅입니다.
{: .notice--warning}

# 개요
 이번 포스팅에서는 FridaLab의 문제들을 풀어보기 위해서, 필요한 프로그램들을 설치하는 방법에 대해서 포스팅 해보겠습니다. FridaLab은 Frida에 익숙하지 않은 사람들이 Frida의 사용법을 익히기 위해서 만든 연습문제 입니다. 해당 문제는 다음 링크에서 다운 받으실 수 있습니다.

  **다운로드 링크 : [https://rossmarks.uk/blog/fridalab/](https://rossmarks.uk/blog/fridalab/){:target="_blank"}**

# 1. Nox
## 1. 설치
 우선 해당 앱 파일을 동작시킬 수 있는 애뮬레이터가 필요합니다. 애뮬레이터 종류는 여러개가 있는데 저는 Nox를 다운받아서 설치해 줬습니다.

 **다운로드 링크 : [https://kr.bignox.com/](https://kr.bignox.com/){:target="_blank"}**

 ![image](https://user-images.githubusercontent.com/33647663/153408971-46d8d471-683a-4b2b-8630-6225df33b7b3.png){: .align-center}

  다운로드를 받고 실행하면 설치가 됩니다. 추가로 이후에 사용할 adb가 nox를 설치하면 기본적으로 함께 설치됩니다. cmd창에서 adb를 바로 사용할 수 있도록 환경변수를 등록해 줍니다.

  ![image](https://user-images.githubusercontent.com/33647663/153412225-f02a377d-a183-4c44-b3c8-6b8139924001.png){: .align-center}

  ![image](https://user-images.githubusercontent.com/33647663/153412738-88a5ee0d-d906-4c1a-b547-b8ea2570ae00.png){: .align-center}

  cmd 창에서 `adb version`을 입력해 보고 잘 출력되는지 확인

  ![image](https://user-images.githubusercontent.com/33647663/153412865-d7906573-3098-47a8-9aee-39e86bcb4a15.png){: .align-center}

## 2. 루팅
  시스템 설정 > 일반 > ROOT 켜기

  ![image](https://user-images.githubusercontent.com/33647663/153413861-3e98870c-f889-49b2-9646-ae2d381d200f.png){: .align-center}



## 3. 디버깅 모드 설정
  바탕화면 > Tools > 설정 

  ![image](https://user-images.githubusercontent.com/33647663/153415727-7f89eb53-ff6f-4fdd-9b72-c5bc9e21da0d.png){: .align-center}

  
  설정 > 빌드 번호 연타하여 개발자 모드 진입

  ![image](https://user-images.githubusercontent.com/33647663/153417311-5346d60d-aabb-4a81-a549-eb3cbf50c69c.png){: .align-center}

  ![image](https://user-images.githubusercontent.com/33647663/153417918-ae4e663f-9213-4d54-8654-19594bd16cb0.png){: .align-center}

  USB 디버깅 허용

  ![image](https://user-images.githubusercontent.com/33647663/153418005-a0bf8f5d-a403-44d1-8678-6581eb426f7e.png){: .align-center}



# 2. jadx
  앱을 다운로드 받으면, 분석을 위해서 디컴파일을 해줘야 합니다. 디컴파일을 하기 위해서 jadx 를 다운받아 줍니다.

  **다운로드 링크 : [https://github.com/skylot/jadx/releases/download/v1.3.2/jadx-1.3.2.zip](https://github.com/skylot/jadx/releases/download/v1.3.2/jadx-1.3.2.zip){:target="_blank"}**


# 3. Frida 설치
  Frida는 파이썬을 먼저 설치해야 합니다. 설치가 안되어 있으시면 다음 링크에서 다음받으시면 됩니다. 버전은 3.x 로 설치해주시면 됩니다.

  **파이썬 다운로드 링크 : [https://www.python.org/downloads/](https://www.python.org/downloads/){:target="_blank"}**

  로컬 서버에 Frida 관련 패키지 설치

  ```python
pip install --upgrade pip
pip install wheel
pip install frida
pip install frida-tools
  ```

  ![image](https://user-images.githubusercontent.com/33647663/153419999-f85f1835-4871-494d-a7bb-550378615edc.png){: .align-center}

  그리고 안드로이드에 설치할 Frida 서버를 다운받아야 합니다.

   **다운로드 링크 : [https://github.com/frida/frida/releases](https://github.com/frida/frida/releases){:target="_blank"}**

  링크로 접속한 다음 **frida-server-[version]-android-x86**을 다운받아 줍니다.

  ![image](https://user-images.githubusercontent.com/33647663/153419717-25d4372f-e16d-4e1f-bc6e-a80bfa47fb67.png){: .align-center}

  다운을 받은 후 압축을 해제해 줍니다. 압축해제 프로그램이 없다면 [7zip](https://www.7-zip.org/download.html)을다운 후 압축 해제해줍니다.

# 4. 안드로이드에 Frida-server 설치
  압축 해제한 Frida-server 파일을 adb를 통해서 안드로이드로 업로드 해줍니다.

  ```
adb devices // 애뮬레이터와 연결되었는지 확인

adb push ./frida-server-15.1.16-android-x86 /data/local/tmp/frida-server
  ```

  ![image](https://user-images.githubusercontent.com/33647663/153429249-da2f2293-8f4f-4752-a787-2ba5bcf9cb26.png){: .align-center}


  adb shell로 접속을 해서 다운로드가 되었는지 확인 후 실행을 시켜줍니다.

  ``` 
adb shell

# cd /data/local/tmp
# ls
# chmod 777 frida-server
# ./frida-server &  
  ```

  ![image](https://user-images.githubusercontent.com/33647663/153429502-e6694373-7a53-4b60-8875-9382e49febde.png){: .align-center}


  창을 하나 더 열어서 다음 명령어를 입력해 봅니다.

  ```
frida-ps -U
  ```

다음처럼 프로세스의 정보가 출력되면 성공적으로 안드로이드의 frida 서버와 로컬의 frida 클라이언트간의 통신이 되고 있는 것입니다.

```
C:\Users\kang>frida-ps -U
 PID  Name
----  ---------------------------------
1739  adbd
3737  android.process.acore
2475  android.process.media
1814  audioserver
2415  cameraserver
2720  com.android.carrierconfig
3722  com.android.inputmethod.pinyin
2511  com.android.keychain
2498  com.android.launcher3
2697  com.android.managedprovisioning
2771  com.android.onetimeinitializer
2228  com.android.phone
2530  com.android.printspooler
2604  com.android.providers.calendar
3555  com.android.systemui
2967  com.google.android.gms
3749  com.google.android.gms.persistent
3930  com.google.android.gms.unstable
3110  com.google.process.gapps
3632  debuggerd
3636  debuggerd:signaller
1816  drmserver
4262  frida-server
1742  gatekeeperd
1804  healthd
   1  init
1817  installd
1818  keystore
1808  lmkd
4214  logcat
4264  logcat
1676  logd
1819  media.codec
1825  media.extractor
1823  mediadrmserver
1828  mediaserver
1829  netd
1812  rild
2181  sdcard
1809  servicemanager
4025  su
1810  surfaceflinger
2105  system_server
1098  ueventd
1099  ueventd
1688  vold
2175  wpa_supplicant
1813  zygote
2927  녹스 앱 스토어
2245  설정
```

여기까지 설치를 하면 FridaLab 문제를 풀 준비가 되었습니다. 다음 포스팅에서는 FridaLab을 풀기 전에 간단히 Frida의 사용법을 익히고 그 뒤에 FridaLab풀이를 포스팅 하겠습니다.



