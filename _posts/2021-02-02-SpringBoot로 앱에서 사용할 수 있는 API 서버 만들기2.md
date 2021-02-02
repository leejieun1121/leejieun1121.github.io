---
title : "SpringBoot로 앱에서 사용할 수 있는 API 서버 만들기2 "
excerpt : "SpringBoot+mysql+mybatis를 이용한 UserAPI 만들기 "

categories:
  - Spring
tags :
  - SpringBoot 
  - Java
---

저번에는 메모리에만 저장되는 HashMap을 사용해서 API 테스트를 했었다.
이번엔 MySql과 MyBatis를 사용한 API만들기 !

---
### 1. mysql에 테이블 생성하기

나는 예전에 spring 스터디를 하면서 mysql  workbench를 설치했었다. 그래서 설치과정은 생략 ㅠ 

테이블을 만들때, 모델 클래스 이름(UserProfile)과 테이블 명을 일치시키는게 좋고, 멤버 변수의 이름과 컬럼명을 일치시키는게 좋다고 한다.

~~~
CREATE TABLE UserProfile(
	id VARCHAR(64) PRIMARY KEY,
	name VARCHAR(64),
	phone VARCHAR(64),
	address VARCHAR(256)
)
~~~

### 2. 의존성 추가하기 
 mysql과 mybatis를 사용하기위해  pom.xml 에 의존성을 추가해줘야한다.
<https://mvnrepository.com/> 여기서 
mysql과 mybatis를 검색한뒤 가장 최신 버전을 복사해서 붙여준다.

<img width="723" alt="스크린샷 2021-02-02 오후 10 20 53" src="https://user-images.githubusercontent.com/53978090/106605983-eb339400-65a4-11eb-9c7b-cee2944c6bad.png">

<img width="751" alt="스크린샷 2021-02-02 오후 10 21 31" src="https://user-images.githubusercontent.com/53978090/106606097-10280700-65a5-11eb-8c06-97a4c58488ee.png">

<img width="646" alt="스크린샷 2021-02-02 오후 10 23 21" src="https://user-images.githubusercontent.com/53978090/106606298-51201b80-65a5-11eb-8a4f-ed85926226da.png">

### 3. mysql 정보 추가해주기
그리고 만들어둔 테이블과 연결하기 위해서 resources -> application.properties에 다음 정보를 추가해준다.
~~~
spring.datasource.url = jdbc:mysql://localhost:3306/sys?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Seoul
spring.datasource.username=root
spring.datasource.password=1121
~~~

![스크린샷 2021-02-02 오후 10 26 28](https://user-images.githubusercontent.com/53978090/106606695-c55abf00-65a5-11eb-8b0c-44af678ec9e8.png)

sys -> 데이터베이스 이름 , 나머지는 그냥 써준다고한다. 
username -> 계정 이름
password -> 비밀번호 

### 4.Mapper 만들어주기 
생성해준 테이블과 매핑되는 Mapper를 만들어준다. 오랜만에 쿼리문 작성 ㅎ

Mapper 라는걸 인식하기 위해 
UserProfileMapper 인터페이스를 생성하고 @Mapper 어노테이션을 붙여준다.

**특정 User 조회**
~~~
_@Select_("SELECT * FROM UserProfile WHERE id=#{id}")

UserProfile  getUserProfile(_@Param_("id")  String  id);
~~~

**전체 User조회**
~~~
_@Select_("SELECT * FROM UserProfile")

List<UserProfile>  getUserProfileList();
~~~

**User 생성 **
~~~
_@Insert_("INSERT INTO UserProfile VALUES(#{id}, #{name}, #{phone}, #{address})")

int  insertUserProfile(_@Param_("id")  String  id,  _@Param_("name")  String  name,  _@Param_("phone")  String  phone,  _@Param_("address")  String  address);
~~~

**User 수정**
~~~
_@Update_("UPDATE UserProfile SET name=#{name}, phone=#{phone}, address=#{address} WHERE id=#{id}")

int  updateUserProfile(_@Param_("id")  String  id,  _@Param_("name")  String  name,  _@Param_("phone")  String  phone,  _@Param_("address")  String  address);
~~~

**User 삭제**
~~~
_@Delete_("DELETE FROM UserProfile WHERE id=#{id}")

int  deleteUserProfile(_@Param_("id")String  id);
~~~

Mapper에서 값을 전달하려면 #을 써야하고 삽입, 수정 ,삭제의 return은 int로, 변경된 레코드의 개수가 반환된다. 

그다음 컨트롤러로 가서 HashMap을 사용했던 코드를 만들어둔 Mapper 를 사용한 코드로 바꿔준다.

**UserProfileController**
~~~
package com.example.demo.controller;

import java.util.List;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.example.demo.mapper.UserProfileMapper;
import com.example.demo.model.UserProfile;


//컨트롤러 인식 
@RestController
public class UserProfileController {
		
	private UserProfileMapper mapper;
	public UserProfileController(UserProfileMapper mapper) {
		this.mapper = mapper;
	}
	
	@GetMapping("/user/{id}")
	public UserProfile getUserProfile(@PathVariable("id") String id) {
		return mapper.getUserProfile(id);
		
	}
	
	@GetMapping("/user/all")
	public List<UserProfile> getUserProfileList(){
		return mapper.getUserProfileList();
	}
	

	@PostMapping("/user/{id}")
	public void postUserProfile(@PathVariable("id") String id, @RequestParam("name") String name, @RequestParam("phone") String phone, @RequestParam(value = "address", required=false) String address) {
		mapper.insertUserProfile(id, name, phone, address);
	}
	
	@PutMapping("/user/{id}")
	public void putUserProfile(@PathVariable("id") String id, @RequestParam("name") String name, @RequestParam("phone") String phone, @RequestParam(value = "address", required=false) String address) {
		mapper.updateUserProfile(id, name, phone, address);
	}
	
	@DeleteMapping("/user/{id}")
	public void deleteUserProfile(@PathVariable("id") String id) {
		mapper.deleteUserProfile(id);
	}
	
	
	

}
~~~
---
유저 생성 테스트
![스크린샷 2021-02-02 오후 10 35 20](https://user-images.githubusercontent.com/53978090/106607680-f091de00-65a6-11eb-8b38-2e2f087a6edc.png)

![스크린샷 2021-02-02 오후 10 42 09](https://user-images.githubusercontent.com/53978090/106608558-e45a5080-65a7-11eb-9a37-545c86883eec.png)


유저 변경 테스트
![스크린샷 2021-02-02 오후 10 44 54](https://user-images.githubusercontent.com/53978090/106608883-45822400-65a8-11eb-8164-7a398188b05d.png)
![스크린샷 2021-02-02 오후 10 45 20](https://user-images.githubusercontent.com/53978090/106608933-559a0380-65a8-11eb-96d1-570eb9e067a4.png)

유저삭제 테스트
![스크린샷 2021-02-02 오후 10 46 07](https://user-images.githubusercontent.com/53978090/106609002-71050e80-65a8-11eb-8258-e3cfffa85438.png)

![스크린샷 2021-02-02 오후 10 46 41](https://user-images.githubusercontent.com/53978090/106609068-85490b80-65a8-11eb-8756-b1931bacf8b6.png)

전부 잘 된다 ㅎㅎ 

[**전체코드**](https://github.com/leejieun1121/SpringBoot-API-TEST)

---

**참고**

<https://www.youtube.com/watch?v=QzHkJsALmyw>
