---
title : "Spring Framework 시작"
excerpt : "Spring Framework 기초"

categories:
  - Spring
tags :
  - Spring
  - Java

---


##### Spring Framework 시작

jsp 다 듣고 드디어 하고싶었던 spring 프레임워크를 들어간다

돈주고 사서 들었던 spring boot 인강이 있었는데 아무것도 모르는 상태에서 바로 하니까

뭔소린지 하나도 모르겠어서 이번에도 인프런에서 spring 기초를 수강하기로 했다 ㅎㅎ

참고 : [https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew#](https://www.inflearn.com/course/스프링-프레임워크_renew#)]

복습용 글쓰기 ~~:grin::raised_hands::fist:



> Spring 프로젝트 만들기



먼저 maven 프로젝트를 만들고 pom.xml에 다음을 추가하여 라이브러리들을 자동 추가해준다

![image-20200324205220710](https://user-images.githubusercontent.com/53978090/77436792-41816380-6e27-11ea-8c56-6dab436b52a0.png)


기존에는 해당객체의 메소드나 필드를 사용하려면 new(생성자)를 이용해서 객체를 만들어줘야 했다.

> TransportationWalk.java

![image-20200324205555321](https://user-images.githubusercontent.com/53978090/77436798-42b29080-6e27-11ea-80b5-de90e5b5f174.png)


> MainClass.java

![image-20200324205621233](https://user-images.githubusercontent.com/53978090/77436804-45ad8100-6e27-11ea-86ff-3d70a54c45a8.png)




![image-20200324205634840](https://user-images.githubusercontent.com/53978090/77436809-47774480-6e27-11ea-8361-4e01e059f369.png)


하지만 spring 프레임워크에서는 resources에 컨테이너를 만들어 객체를 보관해둔다고 한다. 그래서 resuorces 폴더에 applicationContext.xml(스프링설정파일)을 추가해준다.

> applicationContext.xml

![image-20200324205819428](https://user-images.githubusercontent.com/53978090/77436815-49410800-6e27-11ea-9ef0-9d70710d39c2.png)

그다음 아까썼던 new TransportationWalk로 객체를 생성하고 메소드를 사용하는 코드를 주석처리하고

applicationContext.xml컨테이너에서 "tWalk"라는 아이디를 가진 객체를 가져오는 코드를 작성하여 준다.

그리고 마지막에는 자원을 반납해준다.


![image-20200324205956193](https://user-images.githubusercontent.com/53978090/77436830-4c3bf880-6e27-11ea-898d-3c3721689d1f.png)

spring을 사용해서 돌린 결과이다 ㅎ

![image-20200324210103195](https://user-images.githubusercontent.com/53978090/77436838-4e9e5280-6e27-11ea-95bf-c3aa9a287280.png)
