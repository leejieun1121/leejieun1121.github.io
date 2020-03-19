---
title : "Connection Pool"
excerpt : "Connection Pool 에 대해서 알아보자."

categories:
  - JSP
tags :
  - JSP
  - Servlet
  - Java

---

##### Connection Pool

- Connection Pool이란 ?

: 원래 이클립스에서 데이터베이스에 접속할때 자원을 매번 요청하고 , 데이터를 다뤄주고 자원을 다 썼을때는 해제하는 방식을 썼었다.  하지만 이 방식은 굉장히 시간이 많이걸리고 복잡하다. 그래서 미리 데이터베이스와 연결된 Connection 을 만들어 pool에 저장해놓고 필요할때마다 쓰고 다시 반납하는  Connection Pool을 구현해서 사용한다 :smiley:

> Connection Pool 구현

![image-20200320025035296](https://user-images.githubusercontent.com/53978090/77098627-01e8ff00-6a56-11ea-879a-bf8d2a0f9fc5.png)

먼저 Servers-> 자신이 사용하는 톰캣 -> context.xml에 들어간다

![image-20200320025053033](https://user-images.githubusercontent.com/53978090/77098630-02819580-6a56-11ea-8db1-0f1254805e57.png)

커넥션 풀을 설정해준다. name = "연결할 커넥션 풀의 이름 !" , maxActive =연결해놓을 커넥션의 갯수 

![image-20200320025302196](https://user-images.githubusercontent.com/53978090/77098632-02819580-6a56-11ea-8328-0a441b7e50d8.png)

해당 코드로 바꿔주면 구현 끝 !

