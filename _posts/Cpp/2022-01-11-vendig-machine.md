---
title: "[C++] qt를 이용한 자판기 만들기"
categories:
  - cpp
tags:
  - c++
  - qt
  - gui
toc: true
toc_sticky: true
toc_label: "qt설치하기"
toc_icon: "bookmark"
author_profile: true
---

💡 qt creator를 이용하여 자판기를 구현하는 코드입니다.
{: .notice--warning}

# 프로젝트 생성

![image](https://user-images.githubusercontent.com/33647663/148917729-3039c62c-b4b8-433c-b3a9-b81bdbd4bbc7.png)

New > Qt Widgets Application 


![image](https://user-images.githubusercontent.com/33647663/148917941-d853f64a-2db7-48e1-aac6-d3e46294fb01.png)


![image](https://user-images.githubusercontent.com/33647663/148918081-65173279-4f10-495e-ae7e-856ea04cff52.png)


![image](https://user-images.githubusercontent.com/33647663/148918127-de93c86a-1019-402c-89b0-28c161ea728f.png)

![image](https://user-images.githubusercontent.com/33647663/148918232-9c7dfd08-38fd-47d1-857b-8e87efbc642e.png)

![image](https://user-images.githubusercontent.com/33647663/148918287-3465965d-ad83-41bd-b3bb-1c8fb4d88aad.png)

프로젝트를 생성하면
1. vending-machine.pro
2. Headers
3. Sources
4. Forms

4개의 폴더가 생성된다.

![image](https://user-images.githubusercontent.com/33647663/148918975-e44aabd1-48d1-4a5a-aefd-42c5790580ce.png)

메인 폴더는 위젯을 열고 닫는 기능만 있다. 위젯 내부에서의 기능은 widget.cpp에서 처리한다.

![image](https://user-images.githubusercontent.com/33647663/148919061-afe1ab37-d839-4453-b7f0-dbdd9c3f3762.png)

지금은 위젯 내부에 아무런 기능을 넣지 않아서 비어있지만, 위젯 내부에서 어떻게 동작할지를 구현해 줘야 한다.


![image](https://user-images.githubusercontent.com/33647663/148919240-7a9e35f5-994a-499a-9373-07e2ed1b49ea.png)

widget.ui는 xml 파일이다. gui로 표현될 요소들은 xml로 형태로 저장되며, qt에서는 이를 코드로 짜는게 아닌, gui 형태로 붙여 넣으면 자동으로 해당 요소를 xml로 변환해 준다.

![image](https://user-images.githubusercontent.com/33647663/148919483-182d9612-d0c7-4654-9882-69cc77d107a7.png)

Design 탭에서 왼쪽의 요소들을 마우스로 드래그 앤 드롭 형태로 추가해 나가면, wiget.ui에 생성된다. 그리고 widget.ui에 생성된 요소들을 widget.cpp에서 이벤트가 발생했을 때 어떻게 처리를 할지를 구현하면 된다.

# 요구사항
- Widget에 동전 버튼(500원, 100원, 50원, 10원)을 생성하여 버튼을 클릭하면 돈이 증가하도록 한다.


- 음료수 버튼(Coffee-200원, Tea-150원, Milk-100원)을 생성하고 버튼을 클릭하면 돈이 감소하도록 한다.


- 돈에 따라 음료수 버튼의 활성화, 비활성화를 설정하도록 한다.


- Reset 버튼을 누르면 잔돈(500원 몇 개, 100원 몇 개, ...)라는 정보를 QMessageBox 클래스를 활용하여 보여 준다.


# 코딩

## 화면 구성

 ![image](https://user-images.githubusercontent.com/33647663/148921010-86431474-358c-42b1-9e89-2906356c07eb.png)

 우선 위와 같이 위젯에 표시할 내용들을 올려 놓는다. 각각의 요소들의 의미에 맞는 이름들을 작명한다. Camel 표기법으로 pbCoin500,pbCoin100과 같이 만든다.

 이제 해당 버튼들을 눌렀을 때 발생하는 이벤트를 어떻게 처리할지 만들어야 한다.

 ![image](https://user-images.githubusercontent.com/33647663/148921414-a28a3e08-878b-4b57-8b28-ebb184ad3894.png)

 마우스 우클릭 -> Go to slot 
 위의 그림처럼 어떤 이벤트에 대한 처리를 할지 선택할 수 있다. 참고로 qt에서는 이벤트를 signal 이라고 부르며,그걸 처리하는 핸들러를 slot이라고 부른다.

 Click 이벤트에 대해서 처리를 해야 하기 때문에 clicked() 시그널을 선택해 준다. 그러면 아래와 같이 widget.cpp 파일에서 로직을 구현할 수 있다.


 ![image](https://user-images.githubusercontent.com/33647663/148931719-d6da5b0c-5dbd-4e37-ba18-b7e0781c1c69.png)

widget.cpp 파일에서 구현한 내부 로직은 다음과 같다.
우선 각각의 동전을 클릭하는 signal이 오면, slot에서 changeMoney()함수로 money변수의 값을 변경해 주고, 변경된 값을 출력하는 로직이다.
이때, setControl 함수는 매번 체크해 줌으로써 money가 일정수준 이하면, 자판기에서 물건을 뽑을 수 없도록 하였다.

 ```cpp
void Widget::changeMoney(int coin)
{
    money += coin;
    setControl();
}

void Widget::setControl()
{
    ui->pbCoffee->setEnabled(money>=200);
    ui->pbTea->setEnabled(money>=150);
    ui->pbMilk->setEnabled(money>=100);
}



void Widget::on_pbCoin500_clicked()
{
    changeMoney(500);
    ui->lcdNumber->display(money);

}

 ```

 마지막으로 자판기에 남은 거스름돈을 출력하는 로직은 다음과 같이 구현하였다.

 ```cpp
void Widget::on_pbReset_clicked()
{
    QMessageBox msgBox;
    int change500 = money / 500;
    money = money % 500;
    int change100 = money / 100;
    money = money % 100;
    int change50 = money / 50;
    money = money % 50;
    int change10 = money / 10;
    money = money % 10;
    char str[128];
    sprintf(str,"Coin Change is \n500: %d, 100: %d, 50: %d, 10: %d",change500,change100,change50,change10);
    msgBox.setText(str);
    msgBox.exec();
    ui->lcdNumber->display(money);
}
 ```


## 실행 결과
 ![image](https://user-images.githubusercontent.com/33647663/148971317-11b24aaf-88c2-4590-89cf-a61a093c387b.png)

 해당 소스코드는 [깃허브](https://github.com/kangmyoungseok/vending-machine)에 업로드.
 

## 코드 리뷰
 - 동전의 개수가 늘어날 것을 대비해서 배열로 만들기
 - 반환을 하고나서 setControl이 실행 안되서 - 돈이 가능 
 - msgBox.exec함수가 실행되고 display가 나오니까 누르고 나서 ok를 눌러야 돈이 바뀜
 
 ## 수정사항
   - 동전 개수가 늘어날 것을 대비해서 CoinList 배열로 만들어 줘서, 만약 1000원짜리 동전이 추가 된다면 배열에 원소 하나만 추가해 주면 된다.
   ```cpp
   const int COIN_LIST[] = {500,100,50,10};
   ```
   - 동전 반환의 기능을 더 깔끔하게 바꿈
   ```cpp
   int Widget::getChange(int coin)
    {
        int change = money / coin;
        money %=coin;
        return change;
    }

    void Widget::on_pbReset_clicked()
    {
        QMessageBox msgBox;
        QString msg = "Coin Change\n";
        int coinListNum = sizeof(COIN_LIST) / sizeof(int);
        for(int i=0; i<coinListNum; i++){
            int coinNum = getChange(COIN_LIST[i]);
            if(coinNum > 0){
                QString strCoinType = QString::number(COIN_LIST[i]);
                QString strCoinNum = QString::number(coinNum);
                msg += strCoinType + ":" + strCoinNum + "\n";
            }
        }
        changeMoney(0);
        msgBox.information(this,"coin reset",msg);
    }
   ```

 


🔔 **포스팅 공지** <br><br>
해당 포스팅은 BoB 10기 이경문 멘토님의 과제를 정리한 포스팅 입니다.<br>
{: .notice--success}