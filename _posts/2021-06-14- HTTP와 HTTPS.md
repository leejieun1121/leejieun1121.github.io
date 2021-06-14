---
title : "HTTP와 HTTPS란? "
excerpt : "HTTP와 HTTPS를 알아보자"

categories:
  - CS
tags :
  - cs
---



[얄팍한 코딩사전님의 HTTPS강의]([https://www.youtube.com/watch?v=H6lpFRpyl14](https://www.youtube.com/watch?v=H6lpFRpyl14))를 참고하여 정리했습니당 

### HTTP(Hyper Text Transfer Protocol)

Hypertext인 HTML을 전송하기 위한 통신규약 = 서버-클라이언트 데이터를 주고 받기 위한 프로토콜

= 인터넷에서 하이퍼텍스트를 교환하기 위한 통신 규약

**암호화가 되지 않은 평문 데이터를 전송하는 프로토콜이였기 때문에, 개인정보를 조회할 수 있었고** 이러한 문제를 해결하기 위해 HTTPS가 등장

### HTTPS(S: Secure Socket)

HTTP에 데이터 암호화가 추가된 프로토콜 **공개키 암호화 방식**을 지원한다.

**SSL 인증서(디지털 인증서, 보안 소켓 계층 인증서)**를 사용함 

### 대칭키 vs 비대칭키(공개키)

- **대칭키**

    **양측이 같은 키를 가지고 있어 암호화 복호화할때 동일한 키를 사용하는것**

    **장점**

    속도가 빠르다 → 간단한(보안 필요x) 데이터 주고받을 때 사용 

    **단점** 

    애초에 양쪽에서 동일한 키를 가지려면 키를 한쪽으로 보내줘야하는데 그 과정에서 제3자가 키를 알게 될수도 있다. (보안상의 문제)

  ![image](https://user-images.githubusercontent.com/53978090/121906811-6fbdcc80-cd66-11eb-8581-2b4ade84f4d0.png)

- **비대칭키(공개키)**

    A키로 암호화 → B키로 복호화 / B키로 암호화 → A키로 복호화

    **장점**

    **1) 우리가 보내는 정보(id, pw등 중요한 정보)를 제3자가 가로챌 수 없음**

    **2) 접속을 시도하는 사이트가 우리가 들어가려는 사이트가 맞는지, 안전한 사이트인지 신뢰할 수 있음**

    **단점**

    대칭키방식에 비해 암호화 복호화 과정이 추가되기 때문에 속도가 느리고, 컴퓨터에 부하가 걸린다. 

    https로 되어있는 A기업 사이트에 접속한다고 가정했을때,

    초록 : A기업의 공개키, 누구나 가지고 있음

    빨강 : A기업의 개인키, 하나만 가지고 있음

    ![image](https://user-images.githubusercontent.com/53978090/121906845-79473480-cd66-11eb-948f-01e75cc680fc.png)
    
    ![image](https://user-images.githubusercontent.com/53978090/121906865-7ea47f00-cd66-11eb-9e3d-f2be9616fb68.png)
    
    A기업의 공개키(누구나 다 아는키)로 암호화해서 A기업으로 보냄, A기업은 개인키를 가지고 있기 때문에 그걸로 복호화함 제 3자가 A기업의 공개키를 가지고 있다하더라도, 개인키로만 복호화가 가능하기 때문에 정보를 가로챌 걱정을 하지 않아도 됨 

    
    ![image](https://user-images.githubusercontent.com/53978090/121906890-8401c980-cd66-11eb-915b-07ad0829bc57.png)


    ![image](https://user-images.githubusercontent.com/53978090/121906908-882de700-cd66-11eb-8728-f824ba8c4fd6.png)
    
    그리고 다시 A기업에서 암호화해서 나에게 보내면 나는 A기업의 공개키를 가지고 있기 때문에(해당 기업이 개인키로 다시 암호화 한거라서 내가 가지고 있는 공개키로 풀 수 있음) 다른 사이트(B기업)에서 보낸 정보는 A 기업의 공개키가 아니기 때문에, 내가가진 공개키로 풀 수 없음! →  신뢰할 수 있는 사이트라는걸 알게됨 ! 

    ⇒ 우리가(+다른 사람들도) 가지고 있는 공개키가 A기업의 정품 공개키라는걸 어떻게 보장할 수 있지?! → CA(Certificate Authority)라는 공인된 민간기업에 의해 보장받을 수 있음! 어떻게 ?! handshake라는 과정을 통해

    ### handShake 과정


    ![image](https://user-images.githubusercontent.com/53978090/121906937-8e23c800-cd66-11eb-9a56-9be1e3e72bf0.png)
    
    **1.클라이언트는 랜덤데이터를 생성해서 서버에 보냄(파란색)**

    **2. 서버는 랜덤데이터 + 해당 기업의 인증서를 보냄(초록색)**

    **3. 클라이언트는 브라우저에있는 CA 목록과 해당기업의 인증서를 비교하여 해당 기업이 신뢰할 수 있는 기업이 맞는지 확인 !(비대칭키 방식으로)**

    CA인증을 받은 인증서들은 해당 CA개인키로 암호화 되어있음

    서버에서 받은 인증서가 CA가 인증한 인증서라면, 브라우저에는 CA목록들이 저장되어 있다고 했으니 CA의 공개키로 복호화 할 수 있음

    1) **여기서 브라우저의CA목록과 일치하는것이 없다면** 

    다음과 같이 ! 주의요함 표시가 뜸(CA의 인증을 받지 않았다는것)

    ![image](https://user-images.githubusercontent.com/53978090/121906957-92e87c00-cd66-11eb-98ce-7fff4e28585f.png)
    
    ex) http 사이트 검색해서 가장 맨위에 뜬 환경부의 사이트

    2) **일치한다면** 

    CA의 인증을 받은 기업! →  안심하고 정보를 주고받아도 된다.

    여기서부터 대칭키와 비대칭키가 혼용되어 사용됨 !(비대칭키의 단점인 속도, 부담 떄문에) 

    ![image](https://user-images.githubusercontent.com/53978090/121906978-967c0300-cd66-11eb-9b02-38066af0d22a.png)
    
    서버에서 받은 인증서에는 서버의 공개키가 포함되어 있음 각자 주고받은 두가지의 랜덤데이터를 어떤 데이터로 만들어 임시키로 만들고, 서버의 공개키로 암호화하여 서버로 보낸뒤, 양쪽에서 일련의 절차를 통해 대칭키를 만들어 공개키 방식으로 주고받음!  

---

**참고**

[https://mangkyu.tistory.com/98](https://mangkyu.tistory.com/98)

[https://www.youtube.com/watch?v=H6lpFRpyl14](https://www.youtube.com/watch?v=H6lpFRpyl14)
