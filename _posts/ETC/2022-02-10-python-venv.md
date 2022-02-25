---
title:  "[Python] 가상환경 (venv,anaconda) 설치"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/anaconda.png

categories:
  - python

tags:
  - python
  - anaconda
  - venv
  - activate
  - 아나콘다
  - activate
  - deactivate
  - conda install
  - requirements
  - 가상환경
  - 버전관리
  - virtual
  - environment

---

💡 python venv 환경 설정에 대한 포스팅입니다.
{: .notice--warning}


# 개요  

![png](/assets/images/anaconda.png){: .align-center}{: width="80%" height="80%"}  

이번 포스팅에서는 가상환경에 대해서 정리해 보려고 합니다. 지금까지는 가상환경을 사용하지 않고 있었는데, 가끔씩 라이브러리의 의존성 문제때문에 오류가 생겼고 가상환경을 만들어야 할 필요성을 느꼈습니다. 이번 포스팅에서는 anaconda와 venv를 사용해서 가상환경을 설정하는 방법에 대해서 작성해 보겠습니다.

  
<br/>

# 1. anaconda
Anaconda를 활용하여 가상환경을 설정하는 방법입니다. 


## 1. 가상환경 생성하기


```python
$ conda create -n 가상환경이름 python=버전
-------------------------------------------
$ conda create -n py38 python=3.8
```

위와같이 입력해주면 `py38`이라는 이름으로 python3.8 버전의 가상환경이 설치가 됩니다.

![image](https://user-images.githubusercontent.com/33647663/153382220-7d99fe91-575c-4af2-989a-c1210192ab46.png)

가상환경이 설치된것은 다음과 같이 확인 가능합니다.
```python
$ conda info --envs
```

![image](https://user-images.githubusercontent.com/33647663/153382609-e9047ffe-44b2-4a73-b6ce-d2de24121fa9.png)


## 2. 가상환경 활성화하기  
생성한 가상환경은 다음과 같이 사용 가능합니다.

```python
$ conda activate 가상환경이름
-------------------------------------------
$ conda activate py38
```

![image](https://user-images.githubusercontent.com/33647663/153382833-d44fcbb7-1e12-480b-bab6-a631ff7af484.png)


가상환경의 비활성화는 다음과 같이 사용 가능합니다.

```
$ conda deactivate
```

![image](https://user-images.githubusercontent.com/33647663/153383372-8f8dc170-552d-4e58-876d-df847c39db5c.png)

## 3. 라이브러리 설치
가상환경을 활성화 한다음에 pip로 설치하면 됩니다.

![image](https://user-images.githubusercontent.com/33647663/153384490-6018517f-ed2c-4280-9c92-89e9a1b09342.png)

이렇게 pip로 설치한 것은 가상환경 내에서만 설치된 것이며, 기본 python에는 적용되지 않습니다.

![image](https://user-images.githubusercontent.com/33647663/153384834-4ad575e5-8f60-4cf6-ab96-c6a8a7af7434.png)



## 4. 가상환경 삭제하기  
생성한 가상환경을 삭제할 수도 있습니다.
```python
$ conda remove -n 가상환경이름 --all
----------------------------------------
$ conda remove -n py38 -all
```


## 5. 가상환경에 설치된 라이브러리 버전 관리
깃허브에서 오픈소스 프로젝트를 볼때 종종 requirements.txt 파일을 볼 수 있습니다. 현재 가상환경에서 설치된 라이브러리의 버전정보를 담은 파일로 다음과 같이 생성할 수 있습니다.

```python
$ pip freeze > requirements.txt
```

그리고 새로운 가상환경에서 requirements.txt에 존재하는 라이브러리를 설치하기 위해서는 다음과 같이 설치해 주면 됩니다.

```python
$ pip install -r requirements.txt
```

<br/>

## 6. VScode 에서 가상환경 사용하기
위에서 생성한 가상환경을 VSCode에서 사용할 때는 다음과 같이 사용해 주시면 됩니다.


VSCode를 열고 파이썬 파일을 만들면 화면 좌측 하단에 이전에 사용하고 있던 인터프리터가 설정되어 있습니다. 
![image](https://user-images.githubusercontent.com/33647663/153396865-330525aa-4e58-46fb-8e1b-f5b51fc60c39.png)

해당 인터프리터를 클릭하면 다음과 같이 선택을 할 수 있습니다.

![image](https://user-images.githubusercontent.com/33647663/153397087-d8156173-4faf-4569-a8e2-802f1b3c37d5.png)

만약 자동으로 VSCode가 찾지 못했다면 ```Enter Interpreter path```를 통해서 직접 가상환경이 설치된 경로에서 python.exe파일의 경로를 등록해 줘야 합니다.

## 7 . 요약
```python
$ conda create -n 가상환경이름 python=버전
$ conda info --envs
$ conda activate 가상환경이름
$ conda deactivate
$ conda remove -n 가상환경이름 --all
$ pip freeze > requirements.txt
$ pip install -r requirements.txt
```
# 2.venv 사용하기
아나콘다는 따로 아나콘다를 설치해 줘야 하지만, venv는 Python 3.3부터 venv module로 standard library로 포함되어 있어 별도의 설치 과정이 필요 없습니다.

만약 Deep Learning으로 작업하는 프로젝트에서 버전이 중요하다면 다음과 같이 독립된 가상환경을 만들어 줄 수 있습니다.


## 1. 가상환경 생성하기
다음과 같이 프로젝트가 저장된 공간으로 이동한 뒤에, 가상환경을 만들어 주면 됩니다.

```cmd
$ cd <프로젝트 디렉터리>
$ python -m venv .venv
$ dir 
```

## 2. 가상환경 활성화 하기
가상환경을 실행하는 파일은 .venv\Scripts\activate.bat 파일입니다. 다음과 같이 실행해 줍니다.

```cmd
$ .venv\Scripts\activate.bat
```

![image](https://user-images.githubusercontent.com/33647663/153401966-dbcd1bff-d506-42b0-936b-515c292a174d.png)


가상환경의 비활성화는 conda와 동일하게 deactivate 입니다.

![image](https://user-images.githubusercontent.com/33647663/153402408-029c99eb-d45f-4311-845a-0e7b0b2c9824.png)

나머지 라이브러리 설치는 conda와 동일한 개념으로 가상환경 내에서 라이브러리를 설치하면 외부의 파이썬에는 설치되지 않습니다. 그리고 가상환경의 삭제는 .venv 폴더 자체를 휴지통으로 버려서 삭제가 가능합니다.

