---
title: "[webapp] 동적인 데이터를 처리하는 서버 톰캣과, 톰캣이 동작하는 가상의 컴퓨터 JVM 그리고 자바 기반의 웹프로젝트 구성"
date: 2024-10-19 20:40 +0900
lastmod: 2024-10-27 17:17 +0900
categories: webapp
tags:
  [
    Apache,
    Tomcat,
    Catalina,
    Web Container,
    JVM,
    java byte code,
    JIT compiler,
    build,
    gradle,
    gradlew,
    Contents Root,
    Project,
    Module
  ]
---

## Apache / Tomcat

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 아파치는 아파치 소프트웨어 재단에서 관리하는 http 웹서버를 말한다
</div>

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">HTTP 웹서버</span>는 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">정적인 데이터를 처리하는 서버</span>를 말한다.

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#462679;font-weight:bold;">🐈 톰캣</span>은 아파치 소프트웨어 재단의 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">자바 기반의 웹 애플리케이션 서버</span>로써, 흔히 `WAS(Web Application Server)`라고 하는데, 해당 서버는 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">동적인 데이터를 처리하는 서버</span>이다.  
WAS는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;">웹서버</span>와 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#462679;font-weight:bold;">🥥 서블릿 컨테이너</span>의 결합으로 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">웹컨테이너</span>라 불린다.  
DB로 연결되어 데이터를 주고 받거나, 자바 등을 통해 데이터 조작이 필요한 경우에 WAS를 사용한다.  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">WAS는 자바 서블릿을 실행시키고</span>, JSP 코드가 포함되어 있는 웹페이지를 만들어 준다.

- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 Web Service 플랫폼으로써의 역할</span>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 프로그램 실행환경과 DB 접속 기능 제공</span>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 비즈니스 로직 처리</span>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 트랜잭션 관리</span>

즉, WAS 중 하나인 톰캣은 아파치(웹서버)에서 넘어온 동적인 페이지를 읽어들여 프로그램을 실행하고  
그 결과를 다시 HTML로 재구성하여 아파치에 되돌려준다.

<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">아파치는 80포트로 클라이언트 요청이 왔을 때만 응답</span>하고,  
아파치가 알아서 톰캣에 8080포트로 접근한다.

## 톰캣의 구조

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 톰캣의 구조
</div>

<span style="font-weight:700;font-size:1.03rem;">🔅 서블릿, JSP 구동환경을 제공할 뿐만 아니라 HTTP 웹서버 역할을 한다.</span>

- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🥥 Catalina (Servlet Container)</span>
  : 톰캣의 웹컨테이너이다.

  - URL을 특정 서블릿에 매핑
  - 서블릿, JSP 파일에 대한 요청 처리
  - 서블릿 생명주기 관리

- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🐈‍⬛ Coyote (HTTP Component)</span>  
  : HTTP/1.1을 웹서버로 지원하는 톰캣의 `Connector Component`이다
  Catalina는 Coyote를 사용해 웹서버로써 동작한다.

- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎠 Jasper (JSP Engine)</span>
  : 톰캣의 JSP 엔진이다.
  - JSP 파일 구문 분석, 서블릿으로 컴파일

HTTP 요청을 `Coyote`가 받아서 `Catalina`로 전달한다.  
`Catalina`에서 전달받은 HTTP 요청을 처리할 웹 애플리케이션 `Context`을 찾고 `🪛 WEB-INF/web.xml (deployment descriptor file)` 파일 내용을 참조하여 서블릿에 요청을 전달한다.  
요청된 서블릿을 통해 생성된 JSP파일들이 호출될 때, `Jasper`가 `Validation Check`/`Compile` 등을 수행한다.

<span style="font-weight:700;font-size:1.03rem;">🔅 클라이언트로부터 오는 요청을 파싱하여, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">HttpServlet객체</span>로 변환하여 제공한다.</span>

## JVM (Java Virtual Machine)

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 JVM은 자바를 실행하기 위한 가상의 컴퓨터이다
</div>

os에 종속받지 않고, 자바를 실행하기 위해서 os위에서 자바를 실행시킬 무언가가 필요한데, 그게 JVM이다.

`.java` <span style="font-weight:700;font-size:0.98rem;">-></span> `javac` <span style="font-weight:700;font-size:0.98rem;">-></span> `.class` <span style="font-weight:700;font-size:0.98rem;">-></span> `JVM` <span style="font-weight:700;font-size:0.98rem;">-></span> 실행

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">JVM</span>은 `java byte code (.class)`를 인식할 수 있다.  
자바 컴파일러`(javac)`에 의해 변환된 코드의 명령어 크기가 `1byte`라서 `java byte code`라고 불린다.

`java byte code`는 기계어가 아니기 때문에 os에서 바로 실행되지 않는다.  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">JVM</span>의 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">JIT 컴파일러</span>에 의해 os가 이해할 수 있는 `binary code`로 변환되는데,  
따라서, `java byte code`는 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">JVM 위에서 os와 상관없이 실행될 수 있다.</span>

## JVM의 구조

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 JVM의 구조
</div>

![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/In2oD0MDq6.png "Optional title"){: width="700" }

- 클래스 로더 `Class Loader`  
  : <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">런타임 시, 동적으로 클래스 파일을 로드하고 링크하는 역할 수행</span>  
  jar 파일 내에 저장된 클래스들을 JVM위에 탑재하는 역할 수행  
  런타임 데이터 영역에 `java byte code`를 배치시킨다.

- 실행엔진 `🚂 Execution Engine`  
  : <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">클래스를 실행시키는 역할 수행</span>  
  CPU가 실행할 수 있는 형태인 `binary code`로 변환한다.

  - 인터프리터 `Interpreter`  
    : `java byte code`를 명령어 단위로 읽어서 실행
  - JIT 컴파일러 `Just-In-Time Comliation`  
    : <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">동적 번역</span>이라고도 하는데, 프로그램을 실제 실행하는 시점에 기계어로 번역하는 컴파일러이다.  
    인터프리터 방식의 단점을 보완하기 위해 도입되었다.  
    인터프리터 방식으로 실행하다가 적절한 시점에 `java byte code` 전체를 컴파일하여 `binary code`로 번역하고  
    이후에는 인터프리팅하지 않고, 기계어로 직접 실행하는 방식을 말한다.  
    <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">컴파일된 코드는 캐시에 보관하기 때문에, 한번 컴파일된 코드는 빠르게 수행된다.</span>  
    ⚠️ 내부적으로 특정 메서드가 얼마나 자주 수행되는지를 체크하고 일정정도를 넘을때 컴파일을 수행한다.

  - 가비지 콜렉터 `🗑️ Garbage Collector`  
    : 더이상 사용되지 않는 인스턴스를 찾아 메모리에서 삭제한다.

- 런타임 데이터 영역 `Runtime Data Area`  
  : 프로그램을 실행하기 위해, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">os에서 할당받은 메모리 공간</span>을 말한다.  
  스레드가 시작될 때 생성되며, 스레드마다 하나씩 존재한다.  
  <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">현재 수행중인 JVM 명령의 주소를 가지고 있다.</span>

  - `(1) JVM Stack Area`  
    : 프로그램 실행 과정에서 임시로 할당되었다가 메서드를 빠져나가면 소멸되는 데이터를 저장하기 위한 영역  
    ex. 변수, 임시데이터, 스레드, 메서드 정보를 저장  
    <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 메서드 호출 시마다, 각각의 스택프레임이 생성</span>
  - **`(2) PC Register`**
  - `(3) Method Area` `= Class Area` `= Static Area`  
    : 클래스 정보를 처음 메모리 공간에 올릴때, <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">초기화되는 대상을 저장하는 메모리 공간</span>
    - `⛱️ Runtime Constant Pool`  
      : `Static Area`에 존재하는 별도의 관리영역  
      상수 자료형을 저장하여 참조하고, 중복을 막는 역할을 수행
  - `(4) Native Method Area`  
    : 실제 실행할 수 있는 기계어로 작성된 프로그램`(binary code)`을 실행시키는 영역
  - `(5) Heap`  
    : <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">객체를 저장하는 가상 메모리 공간</span>  
    new 연산자로 생성되는 객체와 배열을 저장

    - `Permanent Generation`  
      : 생성된 객체들의 정보의 주소값이 저장된 공간  
      클래스 로더에 의해 로드되는 클래스, 메서드 등에 대한 메타정보가 저장되는 영역  
      `Reflection`을 사용하여 동적으로 클래스가 로딩되는 경우 사용한다.  
      🍍 리플랙션 (reflection)  
       : 객체를 통해 클래스 정보를 분석해내는 프로그래밍 기법을 말한다.

    - `New / Young Area`  
      : 생명주기가 짧은 객체를 GC대상으로 하는 영역  
      이곳의 인스턴스들은 추후 가비지 콜렉터에 의해 사라진다.  
      여기서 일어나는 가비지 콜렉트를 `'Minor GC'`라고 한다.
    - `Eden`  
      : 객체들이 최초로 생성되는 공간
    - `Survivor 0,1`  
      : `Eden`에서 참조되는 객체들이 저장되는 공간  
      `Eden`에 객체가 가득차게 되면 GC가 발생  
      `Eden`에 있는 값들을 `Survivor 1`영역에 복사하고, 나머지 객체 삭제
    - `Old Area`  
      : 생명주기가 긴 객체를 GC대상으로 하는 영역  
      이곳의 인스턴스들은 추후 가비지 콜렉터에 의해 사라진다.  
      여기서 일어나는 가비지 콜렉트를 `'Major GC'`라고 한다.

🧶 스레드  
 : <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">프로세스 내에서 실제로 작업을 수행하는 주체</span>를 말한다.

## 톰캣 인스턴스 (서버)

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 톰캣은 JVM 위에서 동작한다
</div>

<span style="font-weight:700;font-size:1.03rem;">🍪 톰캣을 실행시키기 위해서 JRE 1.1 이상에 부합되는 자바 런타임 환경(Java Runtime Environment)이 필요하다.</span>

![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/1HtcckW25Y.png "Optional title"){: width="700" }

JVM 위에서 하나의 톰캣 인스턴스(서버)가 하나의 프로세스로 동작한다.  
하나의 톰캣 인스턴스에는 여러개의 Service가 존재할 수 있다.

각각의 Service는 하나의 `Catalina Servlet Engine`과 여러개의 `Connector`로 구성된다.  
`Connector`로 들어온 HTTP 요청을 해당 Host에게 전달하는 역할을 한다.

하나의 Engine에는

- 여러개의 `Host`가 존재할 수 있다.  
  : Host는 가상호스트 이름을 나타내며, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">호스트이름이 곧 url에 매핑된다.</span>

- 여러개의 `Context`가 존재할 수 있다.  
  : `Context`는 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">하나의 웹애플리케션</span>을 나타내며 주로 .war 파일의 형태로 배포된다.

톰캣은 자체적으로 보유하고 있는 내부 웹서버와 함께 독립적으로 사용될 수 있지만  
보통 아파치와 같은 다른 웹서버와 함께 사용된다.

굳이 다른 웹서버와 함께 사용하는 이유는  
톰캣의 웹서버 기능은 아파치 웹서버보다 느린 처리속도를 보이기 때문이다.  
즉, 모든 정적/동적 데이터를 톰캣이 처리한다면, 결과적으로 사용자의 요청에 대한 응답이 느려지게 될 것이다.

반면에, 정적데이터는 웹서버이 아파치가 처리하고 동적데이터는 WAS인 톰캣이 처리함으로써  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">서버의 전체적인 부하를 분산하여 응답을 빠르게 하기위해 두 서버를 연동해서 사용한다.</span>

또한, 아파치에서 제공하는 유용한 모듈을 톰캣에서는 사용할 수 없는 부분 등의 이유가 있다.

## 빌드 자동화 도구

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 빌드 자동화 도구, Gradle
</div>

<span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">빌드란 실행파일을 만드는 과정</span>을 말한다.  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(255, 165, 0, 0.7);">작업한 파일들(소스코드, 라이브러리, 이미지)</span>을 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">출시하기 적합한 형태로 포장</span>하는 일을 말한다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">압축</span> <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">변환</span>  
결국 빌드란 <span style="padding:1px 5px;margin-right: 3px;border: 1px solid #DEDEDE;border-radius:5px;">✨ 소스코드 파일을 컴파일한 후 여러 개의 모듈을 묶어</span> 실행파일로 만드는 과정이다.  
빌드되어 나온 결과물을 **`🗿 artifact`**라 한다.

자바 웹 애플리케이션의 경우,  
`.java`, `.xml`, `resource`, `.jpg` `.jpa`을 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">JVM (java virtual machine)</span>이나 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#462679;font-weight:bold;">🐈 톰캣(tomcat)</span> 같은 WAS가 인식할 수 있도록  
패키징하는 과정 및 결과물을 말한다.  
즉, `.java` 파일을 컴파일하여 `.class`로 변환하고  
`resource`를 `.class`가 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">참조할 수 있는 적절한 위치로 옮기고</span>  
`META-INF`와 `MANIFEST.MF`들을 하나로 압축하는 과정을 거친다.

그렇다면 <span style="padding:3px 6px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;font-weight:bold;">빌드 자동화 도구</span>란 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">빌드를 포함하여 테스트 및 배포를 자동화하는 도구</span>를 말한다.  
빌드 자동화 도구는 다음과 같은 작업을 순서대로 수행한다.

```
(1) 전처리(Preprocessing) - 종속성 다운로드
(2) 컴파일(Compile) - 소스코드를 바이너리코드로 변환
(3) 패키징(Packaging)
(4) 테스트(Testing)
(5) 배포 (Distribution)
```

`🐘 gradle`은 `groovy`라는 언어를 기반으로 한 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">오픈소스 형태의 빌드 자동화 도구</span>이다.  
`groovy`는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">JVM</span>에서 실행되는 스크립트언어이다.  
`🐘 gradle`은 필요한 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">라이브러리를 땡겨오거나 버전설정 및 빌드된 라이브러리의 라이프사이클 및 의존관계를 관리해주는 역할</span>을 한다.  
특히 `🪴 spring boot` 프로젝트에서 많이 사용된다.

`🐘 gradle`은 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">캐시를 사용하므로 빌드와 테스트 실행 속도가 빠르다.</span>  
또한, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">🏆 가독성</span>이 우수하다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">필요한 부분만 명확하게 기술</span>  
<span style="padding:1px 5px;margin-right: 3px;border: 1px solid #DEDEDE;border-radius:5px;">✨ gradle 설치 없이</span> `🦣 gradle wrapper`를 이용하여 빌드를 지원한다.  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;font-weight:bold;">설정 주입 방식(Configuration Injection)</span> <span style='color:rgb(196,58,26);font-weight:bold;'>\*</span>을 사용하므로 재사용에 용이하다.

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;color:#34343c;"><span style="font-weight:bold;">📘 설정 주입 방식(Configuration Injection)</span><br>
<span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">플러그인(plugins)</span>, <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">의존성(dependencies)</span>, <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">task</span> 등의 설정을 빌드 스크립트에 외부로부터 주입하는 방식을 말한다.<br>
빌드 로직을 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">재사용</span>하거나 다른 프로젝트에서 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">쉽게 적용할 수 있도록 한다.</span><br>

</div>

```groovy
plugins {
    // 해당 플러그인이 제공하는 기본 빌드 설정과 태스크가 주입된다
    id 'java'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web:2.5.4'
}
// 각 의존성은 gradle이 빌드를 수행할 때, 외부 라이브러리 혹은 모듈을 가져와 사용하도록 구성된다.

```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;color:#34343c;">
<span style="font-weight:bold;">🦣 gradlew</span><br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">gradle wrapper</span>를 실행하는 스크립트다. <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">명령줄 도구</span>
<span style='color:rgb(196,58,26);'>프로젝트에 gradle을 설치하지 않고도</span>, gradle을 다운로드 하고<br>
프로젝트에 지정된 버전의 gradle을 사용하여 빌드할 수 있다.<br>
프로젝트를 공유하거나 다른 컴퓨터에서 빌드할 때 일관된 결과를 얻을 수 있다.<br>
⚠️ 🐘 gradle은 로컬시스템에 gradle이 설치되어있지 않은 경우 프로젝트를 빌드할 수 없다.
</div>

```
# 빌드 및 실행
gradlew.bat build
cd build/libs
java -jar PROJECT_NAME-0.0.1-SNAPSHOT.jar

gradlew clean build
# build 폴더를 지우고 다시 만든다
```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;color:#34343c;"><span style="font-weight:bold;">🪶 Maven</span><br>
Apache 라이센스로 배포되는 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">오픈 소스 형태의 Java전용 빌드 자동화 도구</span>이다.<br>
필요한 라이브러리를 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">pom.xml(project object model)</span>에 정의한다.<br>
해당 파일에는 프로젝트 정보, 빌드 관련 설정, 빌드 환경, pom 연관정보를 담고 있다.<br>
gradle보다 빌드 속도가 느리며, 멀티 프로젝트에서 특정 설정을 다른 모듈에서 사용하려면 상속을 받아야해서 번거롭다.

</div>

## IntelliJ 프로젝트 구성

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 모듈은 하나의 작은 프로젝트 개념이다
</div>

`intelliJ`에서 하나의 프로젝트 안에 여러 모듈을 구성할 수 있다.  
<span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">하위 모듈을 감싸는 프로젝트</span>를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">루트 프로젝트</span>라 하는데,  
모듈 관리만 담당하므로 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">루트 프로젝트의 src 폴더는 지워도 된다.</span>

모듈을 사용하면 하나의 애플리케이션에서 여러 기술과 프레임워크를 결합할 수 있다.  
각 모듈은 자체 프레임워크를 가질 수 있고, `.jar` 혹은 `.war` 형태의 단일 유형이다.  
또한 모듈은 프로젝트에 맞게 구성된 것과는 다른 **`SDK`**, **`Language Level`**, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">
자체 라이브러리</span>를 가질 수 있다.  
각 모듈에 대한 정보는 `📑 .iml` 파일에 저장된다.

`🎱 Project Structure` `ctrl + alt + shift + s`

- `Platform Setting` `🌎 Global`

  - `SDKs`  
    : 여기에 설정된 SDK는 여러 프로젝트(모듈)에서 사용할 수 있다.  
    <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 각 모듈에 대한 개별적인 SDK도 지정가능하다.</span>
  - **`Global Libraries`**

- `Project Setting`  
  : `.idea` 디렉토리에 .xml 형식으로 저장된다.  
  `📑 .iml` 파일과 `🫠 .idea` 디렉토리를 생성하여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">프로젝트 설정을 유지한다.</span>

  - `Project`
    - `SDK`  
      : 여기에 설정된 SDK는 여러 하위 모듈에서 사용할 수 있다.  
      <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 각 하위모듈에 대한 개별적인 SDK도 지정가능하다.</span>
    - `Language Level`  
      : compiler에도 영향을 미친다.
    - `Compiler Output`  
      : `IntelliJ`에 컴파일 결과를 저장하는 디렉토리 경로를 지정할 수 있다.  
      두 개의 하위 디렉토리를 만든다. `production` `test`  
      <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 각 하위모듈에 대한 개별적인 출력 경로 설정도 가능하다.</span>

일반적인 프로젝트에는 일련의 설정과 하나 또는 여러개의 모듈이 있다.  
프로젝트는 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">모듈을 함께 유지하고, 모듈 간 종속성을 제공하며, 공유 구성을 저장하는 껍데기</span>이다.

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;font-weight:bold;">🚀 모듈 (Module)</span>은 모듈 구성을 유지하는 `📑 .iml` 파일과  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">소스코드, 리소스, 테스트를 저장하는</span> **`🩸 Contents Root`**로 구성된다.

`🩸 Contents Root`는 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">코드를 저장하는 폴더</span>이다.  
`🩸 Contents Root` 내의 폴더는 보관하는 콘텐츠에 따라 여러 카테고리에 할당할 수 있다.

`📂 Source`  
 : 해당 폴더에는 컴파일해야하는 프로덕션 코드가 포함되어 있다.

`📂 Generated Source`  
 : 소스폴더의 파일이 수동으로 작성되지 않고 자동으로 생성되며, 재생성할 수 있다고 간주한다.

`📂 Test Source`  
 : 테스트와 관련된 코드를 프로덕션 코드와 별도로 보관한다.  
 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 테스트 소스에 대한 컴파일 결과는 일반적으로 다른 폴더에 배치된다.</span>

`📂 Generated Test Sources`  
 : 소스폴더의 파일이 수동으로 작성되지 않고 자동으로 생성되며, 재생성할 수 있다고 간주한다.

`📂 Resources`  
 : 애플리케이션에 사용되는 리소스 파일, 이미지, xml 및 속성파일이 포함되어 있다.  
 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 해당 폴더의 파일은 기본적으로 그대로 출력폴더에 복사된다.</span>

`📂 Test Resources`  
 : 해당 폴더는 테스트 소스와 관련된 리소스 파일을 위한 것이다.

`📂 Excluded`  
 : 해당 폴더의 파일은 코드 완성, 탐색 및 검사에 의해 무시된다.  
 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 일반적으로 컴파일 출력 폴더는 제외된 것으로 표시된다.</span>  
 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 단, 배포에 영향을 미치지 않는다.</span>
