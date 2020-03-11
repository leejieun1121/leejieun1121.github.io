---
title : "Servlet으로 form 데이터 넘겨받기"
excerpt : "form으로 입력한 데이터의 동작방식(get/post)"

categories:
  - JSP
tags :
  - JSP
  - Servlet
  - Java

---

> form 으로 데이터 넘겨 받기 get / post 방식

##### 2학년 이후로 웹프로그래밍을 한적이 없어서 그런지 오랜만에 봐서 html 코드가 가물가물하다ㅠ

##### 그래도 앱프로그래밍만 계속 하다가 웹프로그래밍하니까 신기하고 재밌다 :laughing:



##### 먼저 html 로 작성된 아래의 코드를 보자.

![image-20200311233124630](https://user-images.githubusercontent.com/53978090/76430171-5acfec00-63f3-11ea-9e66-8b2f6ad01c2d.png)


##### - form 태그: 실질적으로 보이는 태그라기보다는 사용자가 입력하는 데이터들을 action-어디로 보낼지(저기선 mSignUP 어노테이션이 붙은곳으로 보내진다), 그리고 method-어떤 방식으로 보낼지의 역할을 하는 태그

##### - form 태그 안의  input 태그 : 실질적으로 보여지는 태그 type으로 텍스트를 입력하는애를 만들지, 라디오박스를 만들지, 체크박스를 만들지 등등을 결정하고 name으로 앱프로그래밍에서 마치 id를 주는것처럼 속성의 이름을 정해준다. value는 값 !

##### html을 서버로 돌리면 이러한 형태의 웹페이지가 등장

![image-20200311233137600](https://user-images.githubusercontent.com/53978090/76430184-5efc0980-63f3-11ea-962d-15ebe683ed5c.png)


##### 위의 html에서 form 태그에 method부분으로 post를 써주었다. doPost를 호출하겠다는 뜻이다.

##### 여기서 doPost함수를 보면 받은걸 다시 doGet으로 보내주는 모습을 볼 수 있다.

##### 강사님께서 get을 쓰든, post를 쓰든 한꺼번에 몰아서 처리할 수 있도록 이렇게 코드를 짰다고 하셨고 실제로 많이 사용되는 방식이라고 한다.:+1:

##### request 는 사용자가 입력한 값들의 객체!

##### - getParmater : 단일값들을 뽑아낼때 사용

##### - getParameterValues : 다중값들을 받아올 때 사용 (배열을 사용함, 체크박스기때문에 여러값이 들어올 수 있다) 

##### 그리고 getParameterNames로 아까 html로 입력한 name들이 뭐가 있는지 확인할 수 있다. 



![image-20200311233220332](https://user-images.githubusercontent.com/53978090/76430190-5f94a000-63f3-11ea-8ab6-f3e2b7d5956c.png)
![image-20200311233244824](https://user-images.githubusercontent.com/53978090/76430193-615e6380-63f3-11ea-8859-13fd1079660d.png)
##### 위의 웹페이지에 정보를 입력하면, 입력한 값을 잘 받아와서 출력한다!! 그리고 post방식이라 formEx.html 뒤에 지저분하게 붙는 정보가 없이 깔끔하다.:innocent:

![image-20200311234651622](https://user-images.githubusercontent.com/53978090/76430239-70451600-63f3-11ea-88e8-4ceffbb8f23b.png)

![image-20200311234552635](https://user-images.githubusercontent.com/53978090/76430197-61f6fa00-63f3-11ea-9ac6-176a6a34e742.png)
