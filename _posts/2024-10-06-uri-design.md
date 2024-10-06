---
title: "[HTTP] Restful한 URL 설계 개념과 행위를 표현하는 HTTP Method"
date: 2024-10-06 14:13 +0900
lastmod: 2024-10-06 14:13 +0900
categories: HTTP
tags: [URI, URL, API, REST, HTTP method]
---

## URI (Uniform Resource Identifier)

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 URI는 자원 자체를 식별하는 방법이다
</div>

즉, <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">통일된 방식으로 리소스를 식별하는데 필요한 정보</span>를 말한다.
URI 식별 방법에는 위치`locator` 또는 이름`name`으로 식별할 수 있다.

<span style='color:rgb(196,58,26);'>리소스의 위치</span>를 지정하여 식별하는 방법 및 정보를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">URL</span>이라 하며,  
리소스에 이름을 부여하여 식별하는 방법 및 정보를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">URN</span>이라 한다.  
※ URN 이름만으로 리소스를 찾을 수 있는 방법은 아직 보편화되지 않았다.  
※ 때문에 URL이 URI로 불리기도 한다.

**🍪 URL의 구조**

| scheme:// [userInfo@] host [:port] [/path] [?query] [#fragment] |
| --------------------------------------------------------------- | ------------------------------------------------------ |
| scheme                                                          | 프로토콜 정보가 들어간다.                              |
| host                                                            | 도메인명 또는 ip주소를 직접 사용할 수 있다.            |
| path                                                            | 리소스가 위치한 경로로 계층적 구조로 표현한다.         |
| query (= query parameter, = query string)                       | 웹서버에 제공하는 파라미터로, 문자열 형태로 전달된다.  |
| fragment                                                        | html 내부 북마크 등에 사용된다. 서버로 전송되는 정보 x |

## API (Application Programming Interface)

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 API는 sw 프로그램이 다른 sw 프로그램으로 데이터를 전송할 수 있도록 하는 규칙 집합을 말한다
</div>

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 API는 한 소프트웨어가 다른 소프트웨어의 기능을 사용할 수 있는 수단이다.</span>  
API는 결국 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">상호작용을 위한 인터페이스</span>이고, 인터페이스는 짧게 말해 규격이다.  
하나의 프로그램에서 정해진 규칙대로<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">(Schema, API 요구사항)</span> API 요청<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">(API 호출)</span>을 하면,  
API 요청을 받은 다른 프로그램은 요청을 처리하거나, 처리한 데이터를 응답<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">(API 응답)</span>하게 된다.  
여기서 API 요청을 받은 다른 프로그램, 즉 API 응답이 시작되는 곳을 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">API 엔드포인트</span>라 한다.

## URI 설계, REST

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 API 호출은 결국 URL을 통해 서버에 요청을 보내는 것이다
</div>

API 호출하는 입장을 고려해, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">일관적인 컨벤션을 통해 API의 이해도를 향상</span>시켜야 한다.  
즉 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">API URI</span>를 설계를 할 때, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">리소스의 처리를 어떻게 표현할 것인가</span>에 대해 고려해야 한다.  
대표적인 웹서비스 설계하는 아키텍처 스타일이 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">REST(Representational State Transfer)</span>이다.

해당 아키텍처는 리소스와 행위를 분리하여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">URI는 리소스 식별만 표현</span>한다.  
예시로 '회원을 등록한다'라는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">API URI</span>을 설계할 때, URI가 표현해야 하는 것은 그저 '회원'일 뿐이다.  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">즉, 회원이라는 리소스를 URI에 매핑하면 된다.</span>

리소스는 명사로 표현한다.  
<span style="padding:3px 6px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 리소스만으로 표현하기 어려운 프로세스일 경우</span>, 예외적으로 동사를 사용한다. <span style="background-color:#ffdce0;color:rgb(196,58,26);border-radius:5px;padding:2px 3px;">컨트롤 URI</span>

`ex. POST /members` : 회원 등록  
`/members`는 회원이라는 리소스를 나타낸다.  
리소스에 처리되는 행위는 POST로 표현하는데, 이를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">HTTP 메서드</span>라 한다.

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;color:#34343c;">
<span style="font-weight:bold;">📘 HTTP API 설계 개념</span><br>
<B><span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">document</span></B><br>
단일개념(ex. 파일하나, 객체 인스턴스, db row)<br>
ex. /members/100, /files/star.jpg<br><br>
<B><span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">collection</span></B> - <span style='color:rgb(196,58,26);'>POST기반 등록</span><br>
서버가 관리하는 리소스 디렉터리<br>
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">서버가 리소스의 URI를 생성하고 관리</span> (신규 리소스 식별자를 서버가 만들기 때문에)<br>
ex. 회원등록: <span style='color:rgb(196,58,26);'>POST /members</span><br><br>
<B><span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">store</span></B> - <span style='color:rgb(196,58,26);'>PUT기반 등록</span><br>
클라이언트가 관리하는 자원 저장소<br>
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">클라이언트가 리소스의 URI를 알고 관리</span><br>
ex. 파일수정: <span style='color:rgb(196,58,26);'>PUT /files/{filename}</span><br>
ex. 정적 리소스 관리<br><br>
<B><span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">control uri</span></B><br>
document, collection, store로 해결하기 어려운 추가 프로세스 실행<br>
<span style='color:rgb(196,58,26);'>동사를 직접사용</span>
</div>

## HTTP 메서드

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 클라이언트가 서버에 요청을 할 때 기대하는 행위을 HTTP 메서드로 표현한다
</div>

- `GET` : 리소스 조회  
  : 서버에 전달하고 싶은 데이터가 있다면, `query string`을 통해 전달한다.  
  GET으로 요청하면, <span style="padding:1px 5px;margin-right: 3px;border: 1px solid #DEDEDE;border-radius:5px;">✨ 캐싱을 사용하겠다는 서버간의 암묵적인 약속</span>이 있기 때문에, 조회할 때는 GET을 쓴다.  
  `message body`를 통해 데이터를 전달할 수도 있지만, 지원하지 않는 서버가 많아 권장하지 않는다.

  <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">📩 HTTP 요청 메세지</span>

  ```
  GET /search?q=hello&hl=ko HTTP/1.1
  Host: www.google.com

  ```

- `POST` : 새 리소스 생성(등록), 요청 데이터 처리(프로세스 처리), 애매한 경우
  : 등록하는 요청일 시, <span style='color:rgb(196,58,26);'>신규 리소스 식별자 생성 후</span> 프로세스 처리에 사용된다.  
  : `message body`를 통해 서버로 요청 데이터를 전달한다.  
  조회할 시 데이터를 넘겨야하는데, `query string`으로 넘기기 껄끄러운 경우, POST 사용

  <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">📩 HTTP 요청 메세지</span>

  ```
  POST /members HTTP/1.1
  Content-Type: application/json

  {
    "username": "woo",
    "age": 30
  }
  ```

  <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">📩 HTTP 응답 메세지</span>

  ```
  HTTP/1.1 201 Created
  Content-Type: application/json
  Content-Length: 34
  Location: /members/100

  {
    "username": "woo",
    "age": 30
  }
  ```

  서버가 새로 생성한 리소스 식별자를 가지고 URI를 지정한다.  
  해당 URI 정보를 클라이언트가 알게끔 `Location` 헤더 값으로 전달한다.

- `PUT` : 리소스를 완전히 대체(덮어쓰기), 해당 리소스가 없으면 생성  
  : 클라이언트가 리소스의 위치를 알고, URI를 지정한다. (POST와의 차이점)

  <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">📩 HTTP 요청 메세지</span>

  ```
  PUT /members/100 HTTP/1.1
  Content-Type: application/json

  {
    "username" : "moo"
  }
  ```

- `PATCH` : 리소스 부분변경
- `DELETE` : 리소스 삭제
- `HEAD` : GET과 동일하지만, 상태줄과 헤더까지만 반환
- `OPTIONS` : 대상 리소스에 대한 통신 가능 옵션을 설명 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 주로 CORS에서 사용</span>

**🍪 HTTP 메서드의 속성**  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">안전(Safety Method)</span> : 호출해도 리소스를 변경하지 않는다.  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">멱등(Idempotent Method)</span> : 몇번을 호출하든 결과가 같다.  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">캐시가능(Cacheable Method)</span> : 웹브라우저가 응답 결과 리소스를 캐시해서 사용해도 된다.

| 메서드 | 안전 | 멱등 | 캐시가능 |
| ------ | ---- | ---- | -------- |
| GET    | ㅇ   | ㅇ   | ㅇ       |
| POST   | x    | x    | ㅇ       |
| PUT    | x    | ㅇ   | x        |
| PATCH  | x    | x    | ㅇ       |
| DELETE | x    | ㅇ   | x        |
| HEAD   | ㅇ   | ㅇ   | ㅇ       |

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🐖 POST는 멱등이 아니다.</span>  
멱등의 판단 근거는, 서버가 요청을 처리한 후, 일시적인 오류로 정상응답을 주지 못했을 때  
클라이언트가 같은 요청을 다시해도 되는가를 따져보면 된다.(자동 복구 매커니즘)  
ex. 주문 생성 요청에서, 이를 두번 호출하면 중복 결제가 발생할 수 있다.

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#462679;font-weight:bold;">🕷️ GET, HEAD 정도만 캐시로 사용한다.</span> <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ URL을 캐시키로 잡고 구현이 쉽다.</span>  
`POST`, `PATCH`는 본문 내용까지 캐시키로 고려해야 하는데 구현이 쉽지 않다.

## 서버로 데이터 전송방식

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 서버로 데이터를 전송하는 방식에는 크게 query string 또는 message body를 이용한 방식이 있다
</div>

`query string`을 통해 데이터를 전송할 때는  
주로 GET 메서드를 통해, 검색어를 필터할 때이다.

`message body`를 통한 데이터 전송에는 크게 3가지 상황이 있다.

- **(1) 동적 데이터 조회**
- (2) `HTML form`을 통한 데이터 전송 - POST
  : `HTML form` 전송은 `GET`, `POST`만 지원한다.

  <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">📩 웹브라우저가 생성한 HTTP 요청 메세지</span>

  ```
  POST /save HTTP/1.1
  Host: localhost:8080
  Content-Type: application/x-www-form-urlencoded

  username=woo&age=30
  ```

  <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">📩 바이너리 데이터(파일) 전송 시, 웹브라우저가 생성한 HTTP 요청 메세지</span>

  ```
  POST /save HTTP/1.1
  Host: localhost:8080
  Content-Type: multipart/form-data; boundary=---XXX
  Content-Length: 10457

  ---XXX
  Content-Disposition: form-data; name="username"

  woo
  ---XXX
  Content-Disposition: form-data; name="username"

  30
  ---XXX
  Content-Disposition: form-data; name="file1"; filename="intro.png"
  Content-Type: image/png

  jfownboelwjf2o34nfwekonowfw...
  ---XXX
  ```

- (3) `HTTP API`를 통한 데이터 전송  
  : HTTP 요청 메세지를 만들어주는 클라이언트 라이브러리가 있다.  
  ex. 앱클라이언트  
  ex. 웹클라이언트, javascript 통신 (ajax)  
  ex. `React`, `VueJS`와 같은 웹클라이언트와 통신

## 예제: 웹페이지 회원관리 HTTP API 설계, HTML form만 사용

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 예제: 웹페이지 회원관리 HTTP API 설계, HTML form만 사용
</div>

<span style="font-weight:700;font-size:1.03rem;">GET과 POST 밖에 사용하지 못한다.</span>  
<span style="font-weight:700;font-size:1.03rem;">※ HTTP 메서드를 해결하기 어려운 경우 컨트롤 URI(동사로된 URI) 사용</span>

| 메서드 | URL                  | 프로세스       |
| ------ | -------------------- | -------------- |
| GET    | /members             | 회원목록 조회  |
| GET    | /members/new         | 회원 등록 폼   |
| POST   | /members/new         | 회원 등록      |
| GET    | /members/{id}        | 특정 회원 조회 |
| GET    | /members/{id}/edit   | 회원 수정 폼   |
| POST   | /members/{id}/edit   | 회원 수정      |
| POST   | /members/{id}/delete | 회원 삭제      |

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">회원 등록/수정 폼 경로</span>와 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">회원 등록/수정 처리 경로</span>를 맞춰놓으면
처리 중 다시 폼으로 돌아가야하는 경우가 생길 때, 깔끔하게 돌아갈 수 있다.
