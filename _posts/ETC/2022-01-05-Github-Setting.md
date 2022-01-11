---
title: "[Github] 우분투 깃허브 연동"
categories:
  - etc
tags:
  - github
  - ubuntu
toc: true
toc_sticky: true
toc_label: "깃허브 우분투 연동"
toc_icon: "bookmark"
author_profile: true
---

💡 본 포스팅 에서는 우분투에서 깃허브 계정을 연동하는 방법과, 기본적인 사용 방법에 대해서 작성하겠습니다.
{: .notice--warning}

# 깃허브 계정 연동
## Access token 생성
 우분투에서 작업한 코드들을 깃허브에 올리고, 연동하는 방법에 대해서 설명해 드리겠습니다.


 1. 깃허브에서 Settings를 들어간다.

  ![image](https://user-images.githubusercontent.com/33647663/148936695-3b70e564-89b8-43b3-a9ed-a6fcfe1fa6be.png)
 
 2. Developer settings로 들어간다.

  ![image](https://user-images.githubusercontent.com/33647663/148937303-f35fb2a8-ae1f-48a4-a029-2d95a8d1f164.png)

 3. Personal access tokens에서 Generate new token을 누른다.

 ![image](https://user-images.githubusercontent.com/33647663/148937566-bc141e00-6daf-4c8b-9329-9c65fe8d4f86.png)

 4. Note, Expriation, select scopes 부분을 그림과 같이 체크한다음 맨 아래에 Generate token을 누른다.
 
 ![image](https://user-images.githubusercontent.com/33647663/148937746-ba40a5aa-03a8-4580-a338-a463c603d1df.png)
 
 5. ghp_xxx로 시작하는 부분을 복사해서 저장해 놓는다. 이 부분이 앞으로 비밀번호 대신에 사용한 access token이다.
 ![image](https://user-images.githubusercontent.com/33647663/148936982-07175390-3f3c-44d9-98b2-67bf35c8b92d.png)

## Repository 생성
 이제는 우분투 환경에 있는 코드들을 업로드 할 레포지토리를 만들어야 한다.

 1. 깃허브 페이지에서 Repositories -> New를 눌러준다.

 ![image](https://user-images.githubusercontent.com/33647663/148938413-e862c8d7-2d07-4a0b-a55a-023c556b541f.png)

 2. 아래와 같이 레포지토리 이름, 설명을 입력해 주고 Create Repository를 눌러 준다. ADD a REANE file도 해주자.

 ![image](https://user-images.githubusercontent.com/33647663/148938602-20e855b1-c61e-4f26-9eb9-e997ada749e1.png)
 

 3. 그림과 같이 레포지토리가 생성되면, code > https 부분을 복사해 주자.
 
 ![image](https://user-images.githubusercontent.com/33647663/148938948-88f6d299-8478-4a0b-a62f-8b915e1d1f6d.png)


## github repository 다운
 1. 이제 깃허브 서버에 있는 레포지토리를 우분투 로컬에 다운을 받는다. 처음이라면 깃허브 용도의 디렉터리를 만들어서 그곳에 다운받는 것을 추천한다.

 ```md
 mkdir github
 cd github 
 git clone https://github.com/kangmyoungseok/test.git 
 ```
 문제가 없다면 아래와 같이 레포지토리 이름으로 폴더가 생겼을 것이다. 해당 레포지토리에는 깃서버에 있는 레포지토리 폴더가 복사된 것이다. 지금은 README 파일밖에 작성하지 않았기 때문에 README.md 파일밖에 없다. 이제 이곳에 올리고 싶은 코드들을 작성해 주면 된다.

 ![image](https://user-images.githubusercontent.com/33647663/148939606-3500ff75-7bdb-4d03-8928-1ee804a5194b.png)

## Github와 로컬서버 연결하기
 [Acess Tokens](#access-token-생성)부분에서 만든 Access token을 가지고 Github와 로컬서버를 연동시킬 수 있다. 2021년 8월 13일 부터는 깃허브 비밀번호로는 연동 시킬 수 없고 무조건 access token을 생성해서 연결해야 한다고 한다.

 다음 명령어를 입력한다.
 ```md
 git config --global user.name [이름]
 git config --global user.email [이메일]
 ```

 자신의 깃허브 주소를 입력해 봤을때 https://github.com/[이름] 이부분을 이름으로 입력하면 된다.
 
 예시로 나의 경우 github의 주소가 https://github.com/kangmyoungseok/ 이므로 다음과 같이 적었다.
 ```md
 git config --global user.name kangmyoungseok
 git config --global user.email kms00129@naver.com 
 ```

 그리고 아래 명령어를 입력하면 한번 연동을 성공하면 캐싱을 해서 다음부터는 access token을 계속 입력하지 않아도 된다.

 ```md
 git config --global credential.helper store
 ```
 
 위의 명령이 잘 입력되었는지 다음과 같이 확인 할 수 있다.

 ```md
 git config --global --list
 ```

 ![image](https://user-images.githubusercontent.com/33647663/148944570-ae42246e-17da-4463-ad90-098373a2c208.png)

 참고로 ```vi ~/.gitconfig``` 로 파일에서 수정할 수도 있다.
 
 이제 자신이 로컬에서 작업한 내용을 깃허브 서버로 업로드 해보겠다.
 
 파일을 업로드 할 때는
 1. git add *
 2. git commit -m "파일업로드"
 3. git push
 
 3단계만 기억해 주면 된다.

 다음 예시를 보자.
 아까 다운받은 레포지토리에 helloworld.c파일을 만들었다.

 ![image](https://user-images.githubusercontent.com/33647663/148940965-d25598dd-ac54-4163-88bb-59ec9605cc54.png)

 ```md
 git status
 ```
 명령어를 입력해 보면 helloworld.c 파일이 추가 되었다고 나온다.

 ![image](https://user-images.githubusercontent.com/33647663/148941222-249dd93d-e842-4418-a941-1ae8cba70a14.png)

 ```md
 git add helloworld.c
 git commit -m "파일 업로드"
 git push
 ```
 를 입력해 준다.

 ![image](https://user-images.githubusercontent.com/33647663/148941773-80865e2f-d7e6-4c5b-a2da-4b6fdef92c1d.png)

 ```git push```를 입력하면 username을 입력하고, 비밀번호는 위에서 말한 Access token을 입력해 주면 된다.

 업로드가 성공적으로 수행되고 깃허브를 확인해 보면 다음과 같이 업로드가 된 것을 볼 수 있다.

 ![image](https://user-images.githubusercontent.com/33647663/148941905-2b20c3e5-93a3-441e-aa9c-3de7c150fe36.png)

## 추가적인 작업
 이번에는 README.md파일을 수정해 본다.
 
 ```git status``` 명령어를 입력해 보면 현재 변경 사항이 출력된다.

 ![image](https://user-images.githubusercontent.com/33647663/148942269-41b5a6e9-1a99-4d47-9777-4acd31826619.png)

 ```git add README.md```, ```git commit -m "test"```, ```git push```를 차례대로 입력하여 다시 깃허브 서버로 업로드 한다.
 이때, 이전에 비밀번호를 한번 입력했기 때문에 이번에는 다시 묻지 않는다.

 ![image](https://user-images.githubusercontent.com/33647663/148943287-3cba02f4-21fa-4156-b43a-6cb165e1c107.png)
 
 변경 된 모습
 ![image](https://user-images.githubusercontent.com/33647663/148943360-fba4aa3b-e084-41d3-9d6c-faacdc655da9.png)


