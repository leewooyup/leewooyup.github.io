---
title: "[HTTP] HTTP 상태 코드와 정보성 헤더 및 특수 헤더 (with 쿠키, 캐시)"
date: 2024-10-07 20:28 +0900
lastmod: 2024-10-15 20:28 +0900
categories: HTTP
tags:
  [
    HTTP status code,
    Origin Server,
    User-Agent,
    Payload,
    Content Negotiation,
    Statless,
    Cookie,
    Web Storage,
    Cache
  ]
---

## HTTP 상태코드

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 HTTP 상태코드
</div>

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">HTTP 상태코드</span>는 클라이언트가 보낸 요청의 상태를 응답에서 알려주는 기능을 한다.

| HTTP Status                                    | reason-phrase                                           | desc                                                                                                                                      | related-response-header                                       |
| ---------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| 1xx                                            | Informational                                           | 요청이 수신되어 처리중                                                                                                                    |                                                               |
| <span style='color:rgb(196,58,26);'>2xx</span> | <span style='color:rgb(196,58,26);'>Successful</span>   | <span style='color:rgb(196,58,26);'>요청을 정상 처리 </span>                                                                              |                                                               |
| 200                                            | OK                                                      | 요청을 정상 처리                                                                                                                          |                                                               |
| 201                                            | Created                                                 | 요청에 성공해서 새로운 리소스가 생성됨                                                                                                    | Location: /members/100<br>생성된 리소스 경로 해당 필드로 식별 |
| 202                                            | Accepted                                                | 요청이 접수되었으나, 처리가 완료되지 않음(ex. 배치처리)                                                                                   |                                                               |
| 204                                            | No Content                                              | 서버가 요청을 정상 처리 했지만, 응답 페이로드 본문에 보낼 데이터가 없음                                                                   |                                                               |
| <span style='color:rgb(196,58,26);'>3xx</span> | <span style='color:rgb(196,58,26);'>Redirection</span>  | <span style='color:rgb(196,58,26);'>요청을 완료하려면, User-Agent 추가 조치가 필요</span>                                                 |                                                               |
| 301                                            | Moved Permanently                                       | 리다이렉트 시, 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음                                                                        | Location 헤더가 있으면 해당 위치로 자동 이동                  |
| 308                                            | Permanent Redirect                                      | 리다이렉트 시, 반드시 요청 메서드와 본문 유지                                                                                             | Location 헤더가 있으면 해당 위치로 자동 이동                  |
| 302                                            | Found                                                   | 리소스가 잠시 다른 경로에 있으며, 해당 url을 여전히 사용할 것을 알림<br>리다이렉트 시, 요청메서드가 GET으로 변하고, 본문이 제거될 수 있음 | Location 헤더가 있으면 해당 위치로 자동 이동                  |
| 307                                            | Temporary Redirect                                      | 리소스가 잠시 다른 경로에 있으며, 해당 url을 여전히 사용할 것을 알림<br>리다이렉트 시, 요청메서드와 본문 유지                             | Location 헤더가 있으면 해당 위치로 자동 이동                  |
| 303                                            | See Other                                               | 리다이렉트 시, 요청메서드가 GET으로 변경                                                                                                  | Location 헤더가 있으면 해당 위치로 자동 이동                  |
| <span style='color:rgb(196,58,26);'>4xx</span> | <span style='color:rgb(196,58,26);'>Client Error</span> | <span style='color:rgb(196,58,26);'>클라이언트 오류, 잘못된 요청으로 서버가 요청을 수행할 수 없음 </span>                                 |                                                               |
| 400                                            | Bad Request                                             | 요청파라미터가 잘못되거나, API 스펙이 맞지 않음 etc                                                                                       |                                                               |
| 401                                            | Unauthorized                                            | 클라이언트가 해당 리소스에 대한 인증이 필요함                                                                                             | WWW-Authenticate 해당 필드로 인증방법을 설명                  |
| 403                                            | Client Error                                            | 서버가 요청을 이해했지만, 승인을 거부함<br>인증 자격증명은 있지만, 접근 권한이 불충분함                                                   |                                                               |
| 404                                            | Not Found                                               | 요청 리소스를 찾을 수 없음                                                                                                                |                                                               |
| <span style='color:rgb(196,58,26);'>5xx</span> | <span style='color:rgb(196,58,26);'>Server Error</span> | <span style='color:rgb(196,58,26);'>서버 오류, 서버가 정상 요청을 처리하지 못함</span>                                                    |                                                               |
| 500                                            | Internal Server Error                                   | 서버 문제로 오류 발생                                                                                                                     |                                                               |
| 503                                            | Service Unavailable                                     | 서비스 이용불가, 일시적인 과부하 또는 예정된 작업으로<br>잠시 요청을 처리할 수 없음                                                       | Retry-After 해당 필드로 복구 시점 명시 가능                   |

## 일반 정보성 헤더

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 일반 정보성 헤더
</div>

| header     | Req or Res | desc                                                         |
| ---------- | ---------- | ------------------------------------------------------------ |
| User-Agent | Request    | 클라이언트 애플리케이션(웹브라우저) 정보                     |
| From       | Request    | User-Agent의 이메일 정보<br>검색엔진에서 주로 사용           |
| Referer    | Request    | 현재 요청된 페이지의 이전 웹페이지 주소<br>유입경로 분석가능 |
| Server     | Response   | 요청을 처리하는 Origin 서버의 정보                           |
| Date       | Response   | 메세지가 발생한 시간                                         |

🍍 Origin 서버  
 : HTTP 요청을 보내면 중간에 여러 프록시 서버를 거치게 되는데  
 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">HTTP 응답을 해주는 마지막 서버</span>

## 명시가 필요한 헤더

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 명시가 필요한 헤더
</div>

| header           | Req or Res | desc                                                                                                                        |
| ---------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------- |
| Host             | Request    | 요청한 호스트 정보(도메인)<br>하나의 ip주소에 여러 도메인이 적용되어 있을 때, 정확하게 명시해줘야 한다                      |
| Location         | Response   | 페이지 리다이렉션 리소스<br>201일 때는 생성된 리소스 URI, 3xx일 때는 리다이렉션 대상 리소스를 가리킨다                      |
| Allow            | Response   | 허용가능한 HTTP 메서드<br>405(Method Not Allowed)응답과 함께 명시                                                           |
| Retry-After      | Response   | User-Agent가 다음 요청을 하기까지 기다려야 하는 시간<br>503(Service Unavailable)응답과 함께 서비스가 언제까지 불능인지 명시 |
| Authorization    | Request    | 클라이언트 (인증 메커니즘에 따른) 인증정보 전달                                                                             |
| WWW-Authenticate | Response   | 리소스 접근 시 필요한 인증방법 정의<br>401(Unauthorized) 응답과 함께 명시                                                   |

🍍 가상호스팅을 통해 여러 도메인을 한번에 처리할 수 있는 서버는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">Host 필드</span>를 통해 각각의 도메인을 구분한다.

## RFC7230 ~ 7235, 표현(Representation) 헤더

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 표현 헤더는 message body (payload) 를 해석할 수 있는 정보를 제공한다
</div>

<span style="font-weight:700;font-size:1.03rem;">표현 헤더는 Request, Response 둘다 사용 가능 하다.</span>

| representation-header | desc                | value                                                    |
| --------------------- | ------------------- | -------------------------------------------------------- |
| Content-Type          | 페이로드 형식       | text/html;charset=UTF-8<br>application/json<br>image/png |
| Content-Encoding      | 페이로드 압축방식   | gzip<br>deflate<br>identity (압축안함)                   |
| Content-Language      | 페이로드 자연언어   | ko<br>ko-KR<br>en<br>en-US                               |
| Content-Length        | 페이로드 길이(byte) |                                                          |

**🍪 전송 방식에 따른 헤더**

| transfer-method | related-header                                                                            |
| --------------- | ----------------------------------------------------------------------------------------- |
| 단순전송        | Content-Length                                                                            |
| 압축전송        | Content-Encoding<br>Content-Length                                                        |
| 분할전송        | Transfer-Encoding: chunked<br><span style='color:rgb(196,58,26);'>Content-Length x</span> |
| 범위전송        | (Request)Range: byte=1001-2000<br>(Response)Content-Range: bytes 1001-2000/2000           |

## 협상(Content Negotiation) 헤더

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 클라이언트가 선호하는 페이로드 우선순위
</div>

| 기본 항목       |
| --------------- | -------------------------------------- |
| Accept          | 클라이언트가 선호하는 미디어 타입 전달 |
| Accept-Charset  | 클라이언트가 선호하는 문자 인코딩      |
| Accept-Encoding | 클라이언트가 선호하는 압축 인코딩      |
| Accept-Language | 클라이언트가 선호하는 자연언어         |

<span style="font-weight:700;font-size:1.03rem;">🍪 협상과 우선순위 (Quality Values, q)</span>  
 : `ex. Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7` ⚠️ `q`가 생략되어 있으면 1  
 `ex. Accept: */*,text/*,text/plain,text/plain;format=flowed` ⚠️ 구체적인 것이 우선한다.

## 쿠키와 헤더

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🌋 HTTP는 무상태 프로토콜이다
</div>

클라이언트와 서버가 요청과 응답을 주고 받으면 연결이 끊어지고,  
<span style="padding:3px 6px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 서버는 이전 요청을 기억하지 못한다.</span>  
즉, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">클라이언트와 서버는 서로의 상태를 유지하지 않는다.</span>

웹브라우저에는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🍪 쿠키 저장소</span>가 있다.  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🍪 쿠키 저장소</span>에 최소한의 정보(상태)를 저장할 수 있다.  
<span style='color:rgb(196,58,26);'>쿠키 저장소에 저장된 정보는 모든 요청에 자동 포함된다.</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 네트워크 트래픽을 추가 유발하므로, 최소한의 정보만 담는것을 권장</span>

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 쿠키의 주 사용처는 사용자 로그인 세션 관리에 사용된다.</span>  
 : 로그인 시, 서버에서 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">세션키</span>라는 것을 만들어 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">Set-Cookie</span>에 세션키를 담아 클라이언트에 전달한다.

| Cookie-related-header | Req or Res | desc                                                                             |
| --------------------- | ---------- | -------------------------------------------------------------------------------- |
| Set-Cookie            | Response   | 서버에서 클라이언트로 쿠키전달<br>웹브라우저는 해당 필드 값을 쿠키 저장소에 저장 |
| Cookie                | Request    | 클라이언트가 HTTP 요청 시 서버로 자동 전달 (쿠키값 존재 시)                      |

`ex. set-cookie: sessionId=abc123; expires=Sat, 25-Dec-2024 00:00:00 GMT; path=/; domain=google.com; Secure`

- `expires`  
  : 쿠키 만료일 명시  
  만료일이 되면 쿠키 삭제
- `max-age`  
  : 쿠키 만료시간 초단위 명시  
  0이나 음수를 지정하면 쿠키 삭제
- `domain`
  : 클라이언트가 쿠키를 포함해 요청할 도메인 명시, 서브도메인 포함  
  도메인을 생략하면 현재 문서 기준 도메인만으로 한정, <span style='color:rgb(196,58,26);'>서브도메인x</span>
- `path`
  : 경로로 한번 더 필터링, 해당 경로를 포함한 하위 경로 페이지만 쿠키 전송  
  일반적으로 path=/ 루트로 지정
- `Secure`
  : https인 경우에만 쿠키 전송 가능  
  (원래 쿠키는 http, https 구분하지 않고 전송)
- `HttpOnly`
  : XSS 공격 방지, 자바스크립트에서 접근 불가 <span style='color:#9E0ADD;'>document.cookie</span>  
  : HTTP 전송에만 사용
- `SameSite`
  : XSRF 공격 방지, 요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 쿠키 전송

<span style="font-weight:700;font-size:1.03rem;">🍪 세션쿠키 : 만료날짜/초를 생략하면 브라우저 종료시까지만 쿠키 유지</span>  
<span style="font-weight:700;font-size:1.03rem;">🍪 영속쿠키 : 만료날짜/초를 입력하면 해당 날짜/초까지 유지</span>

### 쿠키와 웹스토리지

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🥯 <span style="font-family: Impact,Charcoal,sans-serif;">Cookie &nbsp;/&nbsp; Web Storage</span>
</div>

웹브라우저 내부에 데이터를 저장해두고,  
서버에 요청을 보낼 때마다 해당 데이터와 함께 보내고 싶으면 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">🍪 쿠키</span>를 쓰면 되고  
서버에 전송하지 않고, 웹브라우저 내부에 데이터를 저장하고 싶으면  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">🪹 웹스토리지</span> <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">localStorage</span> <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">sessionStorage</span>를 쓰면된다.  
즉, 클라이언트 자바스크립트 로직에서만 쓸거면, 웹스토리지에 저장한다.

## 캐시와 조건부 요청 헤더, 검증 헤더

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🌋 조건부 요청 헤더에 데이터를 보내고, 검증헤더로 검증과정을 거쳐 캐시를 구현할 수 있다
</div>

캐시 유효시간 안에 같은 요청이 발생했을 때,  
캐시 유효시간을 검증하고, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🥟 브라우저 캐시</span>에서 결과를 가져오면  
유효시간 동안은 네트워크를 통해서 데이터를 다운받지 않아도 된다. <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">빠른 사용자 경험</span>

더 나아가서, 캐시가 만료되었더라도  
해당 요청에 대해 <span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">클라이언트가 가진 캐시 데이터와 서버가 가진 데이터가 같다면</span>  
그 역시 데이터를 다시 다운받지 않아도 된다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">캐시를 재사용</span>  
검증을 위한 네트워크를 통한 데이터 전송은 발생하지만 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">전송 용량을 줄일 수 있다.</span>

서버가 <span style='color:rgb(196,58,26);'>미리 비교를 위한</span> 검증데이터를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">검증헤더</span>에 실어 보내고,  
클라이언트는 재요청시 해당 검증데이터를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">조건부 요청 헤더</span>에 실어 보낸다.

| cache-related-header | Req or Res | desc                                   |
| -------------------- | ---------- | -------------------------------------- |
| Last-Modified        | Response   | 페이로드 최종 수정일                   |
| if-modified-since    | Request    | 비교를 위한 캐시된 페이로드 최종수정일 |

서버는 `if-modified-since` 필드값과 재요청에 대한 리소스의 최종 수정일을 비교하여  
`HTTP Status code`를 반환한다.

```
HTTP/1.1 304 Not Modified
Content-Type: image/jpeg
Cache-Control: max-age=60
Last-Modified: ...
Content-Length: 34012

```

캐시된 최종수정일과 리소스 최종수정일이 같으면, <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">캐시된 데이터가 바뀌지 않은 값임을 알리는</span>  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">응답 코드(304)</span>를 반환하고, 이때 HTTP BODY는 전송하지 않는다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">캐시된 데이터를 재사용할꺼니까.. 단, 응답 헤더 정보로 캐시의 메타정보는 갱신한다.</span>

해당 응답이 오면 웹브라우저는 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">캐시로 리다이렉트</span>하여,  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🥟 브라우저 캐시</span>에서 캐시된 데이터를 조회한다.

<span style="font-weight:700;font-size:1.03rem;">☄️ 검증헤더 Last-Modified와 If-Modified-Since 조건부 요청헤더의 한계</span>  
데이터를 수정하여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">최종 수정일이 다르지만 데이터 결과가 캐시된 데이터와 같은 경우</span>  
또는, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">서버에서 별도로 캐시로직을 관리하고 싶은 경우</span>는 다룰 수 없다.

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">검증헤더</span> `ETag (Entity Tag)`와 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">조건부 요청헤더</span> `if-None-Match`를 사용하면 위의 상황을 다룰 수 있다.

| cache-related-header | Req or Res | desc                                                                                      |
| -------------------- | ---------- | ----------------------------------------------------------------------------------------- |
| ETag                 | Response   | 캐시용 데이터에 임의의 고유한 이름을 달아둠<br>일반적으로 해시값이나 버전으로 지정한다    |
| if-None-Match        | Request    | 조건을 만족하면 200 0K<br>조건을 만족하지 않으면 304 Not Modified 응답, 캐시로 리다이렉트 |

※ 해시알고리즘은 내용이 동일하면 같은 해시값을 뱉는다.

`ETag`에 실어 보낼 고유한 이름 즉, <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">캐시 제어 로직은 서버에서 완전히 관리한다.</span>  
클라이언트는 캐시 메커니즘을 모른 채 단순히 전달받은 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">ETag 필드값</span>만 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">조건부 요청헤더</span>에 실어 보내면 된다.

## 프록시 캐시 서버와 캐시 제어 헤더

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 프록시 캐시 서버는 인터넷 망 중간에서 공용으로 사용하는 캐시서버(노드)를 말한다
</div>

보통 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">프록시 캐시 서버</span>`(public cache)`가 존재하면,  
<span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">웹브라우저가 프록시 캐시 서버에 접근하게 만든다.</span>  
즉, 첫번째 요청 유저가 한번 프록시 캐시에 리소스를 다운로드 받아놓으면  
두번째 요청 유저부터는 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">해당 프록시 캐시에서 캐시된 데이터를 사용할 수 있게 설정되어 있다.</span> <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">빠른 응답</span>

웹브라우저의 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🥟 브라우저 캐시</span>는 로컬에서 사용하는 캐시로 `private cache`라면  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">프록시 캐시 서버</span>는 공용으로 사용하는 `public cache`이다.

| cache-related-header | Req or Res | desc                                                                                                                                                                                          |
| -------------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cache-Control        | Response   | public이면 응답이 프록시 캐시 서버(public cache)에 저장되어도 됨을 의미<br>private면 브라우저 캐시에 저장해야 됨을 의미(default)<br>s-maxage=60은 프록시 캐시서버에만 적용되는 max-age를 의미 |
| Age                  | Response   | origin 서버에서 응답 후 프록시 캐시 서버에 머문 시간(초)을 의미                                                                                                                               |

### 캐시 유효시간

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🥯 <span style="font-family: Impact,Charcoal,sans-serif;">Cache-Control: max-age &nbsp;/&nbsp; expires</span>
</div>

둘다 캐시 유효시간을 의미하지만 `max-age`는 초단위로 지정하고,  
`expires`는 날짜로 캐시 만료일을 지정한다.  
초단위가 훨씬 유용하므로 `Cache-Control: max-age`를 권장한다.  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🚫 Cache-Control과 함께 사용되면 expires는 무시된다.</span>

## 예제: 캐시 무효화

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 예제: 캐시 무효화
</div>

| cache-related-header | Req or Res | desc                                                                                                                                                                                                                            |
| -------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cache-Control        | Response   | max-age는 캐시 유효 시간(초)를 의미<br>no-cache는 캐시해도 되지만, 항상 조건부 요청헤더를 통해 origin 서버에서 검증하고 캐시된 데이터를 사용해야 함을 의미<br>no-store는 저장하면 안되고, 메모리에서 사용 후 삭제해야 함을 의미 |
| Cache-Control        | Response   | must-revalidate는 캐시 만료 후 최초 조회 시 origin 서버에서 검증해야함을 의미<br>origin 서버 접근 실패 시, 반드시 오류 발생 (504 Gateway Timeout)<br>캐시 유효시간이라면 캐시 사용                                              |

<span style="font-weight:700;font-size:1.03rem;">확실한 캐시 무효화 응답</span>  
`Cache-Control: no-cache, no-store, must-revalidate`  
`Pragma: no-cache`

<span style="font-size:1.03rem;">※ Pragma는 HTTP/1.0 버전의 하위호환 캐시 제어헤더이다.</span>
