---
title : "Servlet Mapping"
excerpt : "Servlet Mapping 방법을 알아보자"

categories:
  - JSP
tags :
  - JSP
  - Servlet
  - Java

---
> Servelt Mapping 이란?

##### Servlet Mapping이란 패키지명+풀 이름 대신 url주소를 좀더 간결하게 표현하기 위해 간단한 이름을 지어 맵핑하는것을 말한다.  저번에 Servlet 파일을 만들때 url Mappings: 에서 edit하여 맵핑할 이름을 써주었다.하지만 다 만들고 나서 맵핑할 이름을 바꾸고 싶을때도 있다 이때 두가지 방법을 알아보자

##### Servers 우클릭 -> Dynamic Web Project 생성

![image-20200311202938804](https://user-images.githubusercontent.com/53978090/76415398-7a0f4f00-63dc-11ea-9ac2-9a2712a3e82f.png)

![image-20200311202956156](https://user-images.githubusercontent.com/53978090/76415402-7c71a900-63dc-11ea-9cda-d385ea3f67fd.png)

![image-20200311203004995](https://user-images.githubusercontent.com/53978090/76415404-7d0a3f80-63dc-11ea-8d0b-dd5680b60e83.png)

![image-20200311203119691](https://user-images.githubusercontent.com/53978090/76415414-81cef380-63dc-11ea-80a8-f8878d059ab8.png)

>  1) web.xml 파일을 이용하기

##### 먼저 ServletEx에서 어노테이션을 주석처리한 후 

![image-20200311205530991](https://user-images.githubusercontent.com/53978090/76415447-901d0f80-63dc-11ea-8380-fd076ae311cb.png)

##### 밑에 web.xml 에 들어가서 다음 다음 코드를 작성해준다.

#####  < servlet>으로 맵핑할 < servlet>서블릿을 정해주고 < servlet-mapping>으로 위에 써준 서블릿을  < url-pattern>에 써져있는 이름으로 맵핑하겠다라는 의미 ! 



##### 이렇게 써주고 실행을 하면 404 가나오는데 

![image-20200311204407056](https://user-images.githubusercontent.com/53978090/76415480-9dd29500-63dc-11ea-9931-7309e0fc875c.png)

##### 그 뒤에 아까 url mapping으로 맵핑하겠다 써준 이름인 SE를 붙이면 잘 나온다 ㅎ

![image-20200311204351355](https://user-images.githubusercontent.com/53978090/76415499-a6c36680-63dc-11ea-9759-d397bff23358.png)

> Java Annotation 을 이용하기 

##### 얘는 더 간단하게 그냥 어노테이션의 이름을 바꿔주면 된다 !!

![image-20200311204311051](https://user-images.githubusercontent.com/53978090/76415502-aa56ed80-63dc-11ea-83a6-8550fedc22b3.png)

![image-20200311204324364](https://user-images.githubusercontent.com/53978090/76415511-adea7480-63dc-11ea-9290-93fa0ecf243c.png)

![image-20200311204427092](https://user-images.githubusercontent.com/53978090/76415518-b0e56500-63dc-11ea-8387-0640fbafd4ee.png)

![image-20200311204444761](https://user-images.githubusercontent.com/53978090/76415527-b478ec00-63dc-11ea-94fb-a44f48f38941.png)

##### web.xml에 맵핑하겠다고 썼던 /SE로도 가능하고, 어노테이션으로 썼던 /SE1로도 맵핑 가능하다.

##### 이렇게 원래 ServletEx로 맵핑해놨던 url mappings: 를 내맘대로 바꿔서 맵핑할 수 있다 :satisfied:
