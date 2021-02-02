---
title : "SpringBoot로 앱에서 사용할 수 있는 API 서버 만들기 "
excerpt : "SpringBoot로 간단한 유저 생성,조회,삭제,수정 API만들기 "

categories:
  - Spring
tags :
  - SpringBoot 
  - Java
---


 3학년 2학기부터 2번의 대외활동을 하는 동안 Spring을 배워보고 싶어 인강도 결제하고 스터디도 계속 참여했었다.
하지만 백엔드쪽을 한번도 접해보지 않았던 나에게 느껴지는 장벽은 너무 컸고 ㅠ 처음부터 하나하나 세팅하는것도 너무 오래걸렸었다. 
언젠가 해야지 하고 미루고 있었는데  유튜브 돌아다니다가 세팅할것도 적고 좀 더 간단한 spring boot로 API만드는 영상을 발견해서 바로 해봤다!!
맨날 백엔드분이 만들어주시는 api 를 직접 만들어서 쓸 수 있다니 ㅎㅎㅎ 너무 색다르고 재밌다. 
**슬기로운 코딩생활** 감사합니다  :pray:

---
### 1. Spring boot 툴 설치
spring.io로 들어가서 Projects -> Spring Tools 4를 버전에 맞게 설치한다. 
[설치 홈페이지 바로가기](https://spring.io/tools)

### 2. Spring project 만들기
설치된 Spring Tools4를 실행시키고
File -> New ->Spring Starter Project를 만들어준다.
<img width="594" alt="스크린샷 2021-02-02 오후 1 32 51" src="https://user-images.githubusercontent.com/53978090/106552715-2a3cf780-655b-11eb-8b06-73e1be9d456d.png">


Available에서 Web -> Spring web 라이브러리만 추가해준다. 
<img width="594" alt="스크린샷 2021-02-02 오후 1 33 28" src="https://user-images.githubusercontent.com/53978090/106552753-3d4fc780-655b-11eb-83cf-6d0e25def686.png">

Finish누르면 생성 완료
<img width="594" alt="스크린샷 2021-02-02 오후 1 34 22" src="https://user-images.githubusercontent.com/53978090/106552815-5e181d00-655b-11eb-9938-2e89fd4570dc.png">

생성된 프로젝트를 Spring Boot App으로 실행시켜주면 된다.
*나는 맨 처음에 저게 안떴었는데 밑의 Run Configurations...를 클릭해서 Spring Boot App을 추가해줬다. 
<img width="328" alt="스크린샷 2021-02-02 오후 1 37 11" src="https://user-images.githubusercontent.com/53978090/106552993-c5ce6800-655b-11eb-825d-3e0c5bf0cf7b.png">

실행에 성공하면 다음과 같은 화면이 뜨고 
8080포트번호로  톰캣이 실행된다고 나온다. (*주의할 점은 API테스트 할때 계속 종료하고 다시 켜주고 해야한다.) 
<img width="1055" alt="스크린샷 2021-02-02 오후 1 40 04" src="https://user-images.githubusercontent.com/53978090/106553192-37a6b180-655c-11eb-9996-f68145f5e0ff.png">

아직 생성한 api가 없어서 이렇게 뜬다.
![스크린샷 2021-02-02 오후 1 41 35](https://user-images.githubusercontent.com/53978090/106553255-602eab80-655c-11eb-8865-2595804309ba.png)

ㅠㅠㅠ 직접 톰캣 설치하고 경로 설정해주고 라이브러리 하나하나 찾아서 복붙해줄 필요 없어서 너무 편한거같다.

프로젝트 설정을 완료하고, Model과 Controller 를 만들어준다. 
안드에서는 DTO 혹은 Data 이런식으로 만드는데, 여기서는 Model이라고 하더라 
model 패키지 생성 -> UserProfile 만들기 

**UserProfile**
~~~
package com.example.demo.model;

  

public  class  UserProfile  {

//멤버 변수들을 private +게터와 세터

private  String  id;

private  String  name;

private  String  phone;

private  String  address;

public  UserProfile(String  id,  String  name,  String  phone,  String  address)  {

super();

this.id  =  id;

this.name  =  name;

this.phone  =  phone;

this.address  =  address;

}

  

public  String  getId()  {

return  id;

}

  

public  void  setId(String  id)  {

this.id  =  id;

}

  

public  String  getName()  {

return  name;

}

  

public  void  setName(String  name)  {

this.name  =  name;

}

  

public  String  getPhone()  {

return  phone;

}

  

public  void  setPhone(String  phone)  {

this.phone  =  phone;

}

  

public  String  getAddress()  {

return  address;

}

  

public  void  setAddress(String  address)  {

this.address  =  address;

}

  

}
~~~

그다음 API를 작성하기 위한 함수들을 만들기 위해 Controller 를 만들어준다.
controller 패키지 생성 -> UserProfileController 만들기

UserProfileController 가 컨트롤러라는걸 인식시켜주기 위해서 
@RestController 어노테이션을 붙여준다. 

원래는 database와 연동해서 값을 생성,삭제,조회,수정 등 하지만 맛보기로 HashMap을 만들어 API테스트를 진행한다. 아까 만들어둔 model에 맞게 Hashmap을 초기화 시켜준다.
~~~
private  Map<String,  UserProfile>  userMap;

//데이터 초기화

_@PostConstruct_

public  void  init()  {

userMap  =  new  HashMap<String,  UserProfile>();

userMap.put("1",  new  UserProfile("1",  "홍길동",  "111-1111",  "서울시 강남구 대치1동"));

userMap.put("2",  new  UserProfile("2",  "홍길돈",  "111-1112",  "서울시 강남구 대치2동"));

userMap.put("3",  new  UserProfile("3",  "홍길독",  "111-1113",  "서울시 강남구 대치3동"));

}
~~~

REST API의 HTTP Method는 다음과 같다.
GET : 조회
POST : 생성
PUT/PATCH : 수정 
DELETE : 삭제 

더 많은 내용은 [REST API 참고 블로그](https://meetup.toast.com/posts/92)여기서 찾아보면 좋을거같다. 

### 특정 User 조회 API
조회 API는 GET방식을 사용한다.
GET방식을 사용하기 위해 @GetMapping 이라는 어노테이션을 붙여주고 ()안에 주소를 넣어준다. {}안에 들어가는 id는 PathVariable로 인식하고, 그걸 메소드의 인자로 넘긴다고 한다.
Map에서 해당 id와 일치하는 value 를 return 해준다. 
~~~
//{}안의 부분을 PathVariable로 인식하고 id로 넘겨 호출하게 됨

@GetMapping("/user/{id}")

public  UserProfile  getUserProfile(_@PathVariable_("id")  String  id)  {

//객체만 넘겨줘도 알아서 json으로 넘겨줌

return  userMap.get(id);

}
~~~

![스크린샷 2021-02-02 오후 1 54 25](https://user-images.githubusercontent.com/53978090/106554167-2ced1c00-655e-11eb-8700-11103eb7c1fe.png)

### 전체 User 조회 API

~~~
_@GetMapping_("/user/all")

public  List<UserProfile>  getUserProfileList(){

//usermap이 가지고 있는 value들을 arraylist로 바꿔 리턴

return  new  ArrayList<UserProfile>(userMap.values());

}
~~~
![스크린샷 2021-02-02 오후 1 57 12](https://user-images.githubusercontent.com/53978090/106554380-90774980-655e-11eb-862c-ef19cea739e7.png)

### User 생성 API

값을 전달하는 어노테이션으로는 
- PathVariable
- RequestParam 
두가지가 있다.
PathVariable은 간단하게 한가지만 넘길 수 있고 , 주로 값을 넘길때는 RequestParam을 쓴다고 한다. 

~~~
_@PostMapping_("/user/{id}")

public  void  postUserProfile(_@PathVariable_("id")  String  id,  _@RequestParam_("name")  String  name,  _@RequestParam_("phone")  String  phone,  _@RequestParam_(value  =  "address",  required=false)  String  address)  {
UserProfile  userProfile  =  new  UserProfile(id,  name,  phone,  address);

userMap.put(id,  userProfile);

}
~~~
id는 아까 PathVariable로 생성해줬기 때문에 RequestParam을 쓰지 않은것 같다. 

![스크린샷 2021-02-02 오후 5 53 55](https://user-images.githubusercontent.com/53978090/106575625-a3e6dc80-657f-11eb-9e5a-7def4a9494cc.png)

![스크린샷 2021-02-02 오후 5 55 52](https://user-images.githubusercontent.com/53978090/106575897-e90b0e80-657f-11eb-92fb-cc05f239d96a.png)

똑같이 따라서 코드 입력하고 Postman으로 생성 API 실행해봤는데  Required String parameter 'address' is not present 라는 오류가 계속 떴다. 
찾아봤더니 required=false 를 써줌으로써 null이여도 예외가 발생하지 않도록 해주는것 같았다.
왜 null로 인식하는지는 잘 모르겠다...:joy:

### User 수정 API
어노테이션으로 PutMapping 을 사용하고 id를 통해 해당 id의 이름, 전화번호, 주소 등을 변경할 수 있다.
~~~
_@PutMapping_("/user/{id}")

public  void  putUserProfile(_@PathVariable_("id")  String  id,  _@RequestParam_("name")  String  name,  _@RequestParam_("phone")  String  phone,  _@RequestParam_(value  =  "address",  required=false)  String  address)  {

UserProfile  userProfile  =  userMap.get(id);

userProfile.setName(name);

userProfile.setPhone(phone);

userProfile.setAddress(address);

}
~~~

순심씨를 심술씨로, 화성에서 수원시로 이사시켜보았다. 
![스크린샷 2021-02-02 오후 5 59 13](https://user-images.githubusercontent.com/53978090/106576363-620a6600-6580-11eb-9b3e-9d4d2271a66c.png)

![스크린샷 2021-02-02 오후 5 59 45](https://user-images.githubusercontent.com/53978090/106576422-70f11880-6580-11eb-98db-aa71e2a75e6a.png)

### User 삭제 API
DeleteMapping 을 사용해서 해당 id를 삭제한다. 
~~~
_@DeleteMapping_("/user/{id}")

public  void  deleteUserProfile(_@PathVariable_("id")  String  id)  {

userMap.remove(id);

}
~~~

심술씨를 없애보았다.
![스크린샷 2021-02-02 오후 6 02 05](https://user-images.githubusercontent.com/53978090/106576664-c4fbfd00-6580-11eb-88f1-766bfe53d477.png)

잘 없어진것을 볼 수 있다 ㅎㅎ

![스크린샷 2021-02-02 오후 6 02 26](https://user-images.githubusercontent.com/53978090/106576713-d1805580-6580-11eb-8546-50ebd49453d0.png)

---
**참고강의**<https://www.youtube.com/watch?v=QzHkJsALmyw>
