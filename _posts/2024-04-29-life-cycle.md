---
title: "[Spring] 빈을 생명주기 콜백을 지원하는 스프링 프레임워크와 빈의 유효범위 '빈 스코프' 및 '웹 스코프'"
date: 2024-04-29 19:47 +0900
lastmod: 2024-05-03 19:47 +0900
categories: Spring
tags: [Singleton Scope, Prototype Scope, Provider, Request Scope, Logger, Proxy]
---

## 빈 생명주기 콜백

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 스프링 빈이 생성되거나 죽기 직전에 스프링 프레임워크가 빈 안에 있는 메서드를 호출해 줄 수 있는 기능을 말한다
</div>

즉, `스프링 빈🥔`이 생성되고, <span style='color:rgb(196,58,26);'>초기화</span> 될 때 또는 빈이 사라지기 직전에 <span style='color:rgb(196,58,26);'>안전하게 종료</span>할 수 있는 메서드를 스프링 프레임워크가 호출해 줄 수 있다.

`Database Connection Pool🏖️`와 콜백  
: <span style='color:rgb(196,58,26);'>웹 애플리케이션 시작 시점</span>에 필요한 연결을 미리 해두고,  
: <span style='color:rgb(196,58,26);'>종료 시점</span>에 연결을 모두 종료하려는 작업을 진행할 때 쓰인다.

```java
@Setter
public class NetworkClient {
    private String url;

    public NetworkClient() {
        System.out.println("call the constructor: url=" + url);
        connnect();
        call("initialization connect message");
    }

    public void connect() { // 빈 생성 직후 호출
        System.out.println("connect=" + url);
    }

    public void call(String message) {
        System.out.println("call=" + url + " message=" + message);
    }

    public void disconnect() { // 빈 소멸 직전 호출
        System.out.println("disconnect=" + url);
    }
}
```

`🦕 BeanLifeCycleTest.java`

```java
public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);

        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();

        // 결과
        // call the constructor: url=null
        // connect=null
        // call=null message=initialization connect message
    }

    @Configuration
    static class LifeCycleConfig {
        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://example-test.dev"); // setter주입
            return networkClient;
        }
    }
}
```

다음과 같이 객체를 생성한 다음에 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">의존관계 주입이 되기도 전에 초기화 작업이 수행</span>되었기 때문에, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">null을 출력하는 결과를 냈다.</span>

즉, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">초기화 작업은 객체가 생성되고, 의존관계 주입이 모두 완료되고 난 다음에 호출되어야 한다.</span>

보통은 객체 생성단계를 거치고, 의존관계 주입단계를 거친다.  
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);">(⚠️ 생성자 주입은 객체 생성시, 스프링 빈이 같이 들어와야하기 때문에 예외)</span>  
초기화 작업은 이 의존관계 주입단계가 완료되고 일어나야하는데,  
<span style='color:rgb(196,58,26);'>개발자가 의존관계 주입이 완료된 시점을 어떻게 알 수 있을까?</span>

**스프링 프레임워크**는 의존관계 주입이 완료되면 `스프링 빈🥔`의 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">콜백 메서드를 통해 초기화 시점을 알려준다.</span>  
또한, 스프링은 `스프링 컨테이너🥥` 또는 `스프링 빈🥔`이 종료되기 직전에 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">소멸 콜백 메서드를 통해, 안전하게 종료작업을 진행할 수도 있다.</span>

**☄️ 스프링 빈의 LifeCycle**  
: `스프링 컨테이너🥥` 생성 **->** `스프링 빈🥔` 생성 **->** 의존관계 주입 **->** <span style='color:rgb(196,58,26);'>초기화 콜백</span> **->** 사용 **->** <span style='color:rgb(196,58,26);'>소멸 콜백</span> **->** 스프링 종료

<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#E1FEE5;"><B>초기화 콜백</B>은 빈이 생성되고, 의존관계가 모두 주입 완료된 직후 호출되고,
<B>소멸 콜백</B>은 빈이 소멸되기 직전에 호출된다.</span>

🍪 객체의 생성과 초기화는 분리하는게 좋다.  
초기화 작업에는 외부 연결을 맺는 <span style='color:rgb(196,58,26);'>무거운 작업이 포함될 수 있기 때문에</span>,  
따로 빼놓는 것이 `🏆유지보수성`에 좋다.  
또한, 초기화 작업을 분리하면 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">동작을 지연할 수 있다.</span>  
즉, 객체를 생성하는 것까지만 하고, 실제 초기화 작업(외부 커넥션을 맺거나...)은 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">최초의 액션이 일어날때까지 미룰 수 있는</span> 장점이 있다.  
⚠️ 객체 생성은 딱 메모리를 할당하는 것까지를 말한다.

## 빈 생명주기 콜백을 지원하는 스프링 프레임워크의 3가지 방법

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 초기화/소멸 인터페이스, 초기화/소멸 메서드, 초기화/소멸 어노테이션
</div>

- **초기화/소멸 인터페이스🗿 (InitializingBean, DisposableBean)**

```java
@Setter
public class NetworkClient implements InitializingBean, DisposableBean {
    private String url;

    public NetworkClient() {
        System.out.println("call the constructor: url=" + url);
    }

    public void connect() { // 빈 생성 직후 호출
        System.out.println("connect=" + url);
    }

    public void call(String message) {
        System.out.println("call=" + url + " message=" + message);
    }

    public void disconnect() { // 빈 소멸 직전 호출
        System.out.println("disconnect=" + url);
    }

    @Override
    public void afterPropertiesSet() throws Exception { // 의존관계 주입 직후 호출
        connect();
        call("initialization connect message");
    }

    @Override
    public void destroy() throws Exception {
        disconnect();
    }
}
```

☄️ `초기화, 소멸 인터페이스`의 단점  
: <span style='color:rgb(196,58,26);'>스프링 전용 인터페이스</span>이다. 즉, 해당 코드가 <span style='color:rgb(196,58,26);'>스트링 전용 인터페이스에 의존한다.</span>  
: (코드를 고칠수 없는) <span style='color:rgb(196,58,26);'>외부 라이브러리에 적용할 수 없다.</span>

---

- 설정정보 빈 등록 초기화/소멸 메서드  
  : `@Bean(initMethod="init", destroyMethod="close")`  
  : 라이브러리는 대부분 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">close, shutdown과 같은 이름의 소멸 메서드를 사용한다.</span>  
  : destroyMethod는 기본값이 <span style="margin-bottom:16px;font-size:15px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">(inferred)</span>로 등록되어 있다. <span style="margin-bottom:16px;font-size:15px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">이 기본값은 close, shutdown이라는 이름의 메서드를 자동으로 호출해준다.</span>  
  : 즉, <span style='color:rgb(196,58,26);'>종료 메서드를 추론해서 호출</span>해준다. 따라서 직접 스프링 빈으로 등록하면, 종료 메서드는 따로 적어주지 않아도 잘 동작한다.

```java
@Setter
public class NetworkClient {
    private String url;

    public NetworkClient() {
        System.out.println("call the constructor: url=" + url);
    }

    public void connect() {
        System.out.println("connect=" + url);
    }

    public void call(String message) {
        System.out.println("call=" + url + " message=" + message);
    }

    public void disconnect() {
        System.out.println("disclose=" + url);
    }

    public void init() throws Exception {
        connect();
        call("initialization connect message");
    }

    public void close() throws Exception {
        disconnect();
    }
}
```

`🦕 BeanLifeCycleTest.java`

```java
public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);

        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configurataion
    static class LifeCycleConfig {

        @Bean(initMethod="init", destroyMethod="close")
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://example-test.dev");
            return networkClient;
        }
    }
}
```

♻️ `설정정보 초기화/소멸 메서드`의 장점  
: <span style='color:rgb(196,58,26);'>설정정보를 사용</span>하기 때문에, <span style="margin-bottom:16px;font-size:15px;background-color:#E1FEE5;color:rgb(45,204,112);border-radius:5px;padding:2px 3px;">외부 라이브러리에도 초기화/종료 메서드를 적용할 수 있다.</span>  
⚠️ 또한, 정해진 메서드 이름이 아닌 자유롭게 지정할 수 있다.

---

- `@PostConstruct` `@PreDestroy` 어노테이션 지원 <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:#1196C1;font-weight:bold;border-radius:5px;padding:2px 3px;">자바 표준</span> <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:#1196C1;font-weight:bold;border-radius:5px;padding:2px 3px;">권고</span>  
  : `java.annotation` 패키지로 스프링에 종속적인 기술이 아닌 자바 표준이다.  
  : ⚠️ (코드를 고칠 수 없는) 외부 라이브러리에 적용할 수 없다.

```java
@Setter
public class NetworkClient {
    private String url;

    public NetworkClient() {
        System.out.println("call the constructor: url=" + url);
    }

    public void connect() {
        System.out.println("connect=" + url);
    }

    public void call(String message) {
        System.out.println("call=" + url + " message=" + message);
    }

    public void disconnect() {
        System.out.println("disconnect=" + url);
    }

    @PostConstruct
    public void init() throws Exception {
        connect();
        call("initialization connect message");
    }

    @PreDestroy
    public void close() throws Exception {
        disconnect();
    }
}
```

## 빈이 존재할 수 있는 범위, 빈 스코프

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 스프링 빈은 기본적으로 싱글톤 스코프로 생성된다
</div>

- **`Singleton Scope`**  
  : `스프링 컨테이너🥥`의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다.

- **`Prototype Scope`**  
  : 사용자가 요청을 할 때 `스프링 컨테이너🥥`는 `스프링 빈🥔`의 생성과 의존관계 주입 및 초기화까지만 불러주고, 이후 클라이언트에 반환하고 더이상 관리하지 않는다.  
  ⚠️ 때문에, 소멸 메서드 호출이 안된다.  
  🎯 **요청할 때마다 의존관계 주입이 완료된 새로운 객체가 필요할 때**  
  : 서로 필요한 시점이 다르니까 <span style="margin-bottom:16px;font-size:15px;background-color:#ffdce0;color:rgb(196,58,26);font-weight:bold;border-radius:5px;padding:2px 3px;">📛순환참조 문제</span>가 발생하지 않는다.

싱글톤 스코프 빈을 조회하면 `스프링 컨테이너🥥`는 항상 같은 인스턴스의 `스프링 빈🥔`을 반환한다.  
하지만 프로토타입 스코프 빈을 조회하면, `스프링 컨테이너🥥`는 항상 새로운 인스턴스의 `스프링 빈🥔`을 생성해서 반환하고, 그 이후부터는 프로토타입 빈을 조회한 클라이언트가 직접 관리해야 한다.

즉, 프로토타입 스코프는 `스프링 컨테이너🥥`에서 `스프링 빈🥔`을 조회할 때 생성이 되고,  
초기화 메서드가 이때 실행된다.

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;"><span style="font-weight:bold;">📕 웹 관련 스코프</span><br>
Spring-WEB이 들어가야 쓸 수 있는 스코프<br>
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">request</span>: 웹 요청이 들어오고 나갈때까지 생존범위를 가지는 스코프<br>
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">session</span>: 웹 세션이 생성되고 종료될때까지만 생존범위를 가지는 스코프<br>
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">application</span>: 웹 서블릿 컨텍스트와 같은 생존범위를 가지는 스코프
</div>

`🦕 PrototypeBeanTest.java`

```java
public class PrototypeBeanTest {

    @Test
    void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);
        ac.close();
    }

    @Scope("prototype")
    static class PrototypeBean {
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

## 싱글톤빈과 프로토타입 빈을 함께 사용시 문제점

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🏈 프로토타입 스코프 빈이 정상적으로 동작하지 않을 수 있다
</div>

만약 싱글톤 빈이 스프링 컨테이너 생성 직후, 의존관계 주입을 통해 프로토타입 스코프 빈을 주입받는다고 가정했을 때,  
`싱글톤 스코프 빈`이 내부에 가지고 있는 `프로토타입 스코프 빈`은 <span style='color:rgb(196,58,26);'>이미 주입이 끝난 빈이다.</span>  
즉, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">주입 시점에 요청되어</span> 이미 `스프링 컨테이너🥥`에 의해 **생성되고 초기화까지 마친 빈**이다.  
따라서 이후, 해당 프로토타입 스코프 빈의 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">관리는 호출한 클라이언트 객체가 맡게되고</span>,  
때문에 싱글톤 스코프 빈의 <span style='color:rgb(196,58,26);'>생존범위와 동일하게 동작</span>하게 된다.

☄️ **`프로토타입 스코프 빈`을 주입 시점에서만 새로 생성하는게 아닌, <span style='color:rgb(196,58,26);'>요청할 때마다 새로 생성해서 사용</span>하려면??**  
<span style="margin-bottom:16px;font-size:15px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">Provider</span>를 이용하면 된다.  
Provider는 의존관계를 외부에서 주입받는 것이 아니라,  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(157, 216, 75, 0.7);">직접 필요한 의존관계를 조회한다.</span> <span style="margin-bottom:16px;font-size:15px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">Dependency Lookup</span>  
즉, 지정한 빈을 Provider가 `스프링 컨테이너🥥`에 <span style='color:rgb(196,58,26);'>대신 요청하여 찾아주는 DL 작업을 한다.</span>  
스프링이 제공하는 기능을 사용하지만, 기능이 단순해서 **단위테스트**를 만들거나 **mock 코드**를 만들기 쉬워진다.

`ObjectProvider`: getObject() 하나만 제공  
`ObjectFactory`: 편의 기능 확장

```java
public class SingletonWithPrototypeTest {

    @Test
    void singletonClientUsePrototype() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);

        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);

        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(1);

        @Scope("singleton")
        static class ClientBean {

            @Autowired
            private ObjectProvider<PrototypeBean> prototypeBeanProvider;

            public int logic() {
                PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
                // 호출하면 그때서야 스프링 컨테이너에서 프로토타입 스코프 빈을 찾아 반환.
                // 필요할 때마다 스프링 컨테이너에 대신 요청하는 작업을 할 수 있다.

                prototypeBean.addCount();
                int count = prototypeBean.getCount();
                return count;
            }
        }

        @Scope("prototype")
        static class PrototypeBean {
            private int count = 0;

            public void addCount() {
                count++;
            }

            public int getCount() {
                return count;
            }

            @PostConstruct
            public void init() {
                System.out.println("PrototypeBean.init=" + this);
            }

            @PreDestroy
            public void destroy() {
                System.out.println("PrototypeBean.destroy");
            }
        }
    }
}
```

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;"><span style="font-weight:bold;">📘 스프링에 의존하지 않는 자바표준 JSR-330 Provider</span><br>
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">javax.inject.Provider</span><br>
위의 코드에서 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">Provider&lt;PrototypeBean> prototypeBeanProvider;</span>로 변경해도 의도한 결과가 나온다.<br>
자바 표준이고 기능이 단순하므로, 단위테스트를 만들거나 mock 코드를 만들기가 훨씬 쉽다.<br>
⚠️ <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">implementation 'javax.inject:1'</span> 라이브러리를 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">gradle🐘</span>에 추가.
</div>

## 웹 환경에서만 동작하는, 웹 스코프

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 웹 스코프는 스프링이 스코프의 종료 시점까지 관리한다
</div>

🌵 SPRING-WEB 라이브러리 추가 in SpringBoot  
`🐘 implementation 'org.springframework.boot:spring-boot-starter-web'`

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;"><span style="font-weight:bold;">📕 스프링 웹 기술</span><br>
스프링부트는 내장 톰캣 서버🐈를 활용해서 웹 서버와 스프링을 함께 실행시키는데,<br>
해당 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">웹 관련 라이브러리</span>가 스프링에 포함되면 웹 기술이 들어가면서 애플리케이션이 서버에 띄워진다.<br>
스프링부트는 웹 라이브러리가 없으면 스프링 컨테이너를 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">AnnotationConfigApplicationContext</span>를 기반으로 애플리케이션을 구동하는데<br>
다음과 같은 웹 라이브러리가 있으면 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">AnnotationConfigServletWebServerApplicationContext</span>를 기반으로 애플리케이션을 구동한다.<br><br>

<span style='color:rgb(196,58,26);'>Tomcat started on port(s): 8080 (http) with context path ' '</span><br>
<span style='color:rgb(196,58,26);'>Started CoreApplication in 0.914 seconds (JVM running for 1.528)</span>

</div>

- `request Scope`  
  : <span style='color:rgb(196,58,26);'>request HTTP 요청 하나가 들어오고 나갈 때까지</span> 생존범위를 가지는 스코프  
  : 각 HTTP 요청마다 <span style='color:rgb(196,58,26);'>각각의 스코프를 가진다.</span>  
  : 즉 각 HTTP 요청마다 <span style='color:rgb(196,58,26);'>별도의 빈 인스턴스가 생성되고 관리된다.</span>  
  : ⚠️ <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">HTTP 요청이 같으면 같은 객체 인스턴스를 바라보게 된다.</span>

- `session Scope`  
  : HTTP 세션과 동일한 생명주기를 가지는 스코프

- `application Scope`  
  : 서블릿 컨텍스트와 동일한 생명주기를 가지는 스코프

- `websocket Scope`  
  : 웹 소켓과 동일한 생명주기를 가지는 스코프

## 각 HTTP 요청마다 요청정보 로그찍기

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 예제: 각 HTTP 요청마다 요청정보 로그찍기
</div>

동시에 많은 HTTP요청이 들어올 때, 정확히 어떤 요청이 남긴 로그인지 확인하고 싶을 때  
`request Scope🦚`를 사용하면 딱 좋다.

🍪 HTTP 요청당 **스코프가 생성**되고, HTTP 요청이 끝나는 시점에 **소멸**된다.

`☕ MyLogger.java`

```java
@Component
@Scope(value="request")
public class MyLogger {
    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "] " + "[" + requestURL + "] " + message);
    }

    @PostConstruct
    public void init() {
        String uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean created=" + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "] request scope bean closed=" + this);
    }
}
```

`☕ LogDemoController.java`

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {
    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        // 자바에서 제공하는 표준 서블릿 규약, 고객 요청정보를 받을 수 있다.
        String requestUrl = request.getRequestURL().toString();
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}
```

`MyLogger`는 고객 요청이 올 때, 객체가 주입되는 `request Scope🦚`를 가진다.  
따라서, 해당 객체의 주입은 처음 빈 생성 후, 의존 관계 주입 단계가 아닌  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">실제 고객 요청이 왔을 때로 지연시켜야 한다.</span>

<span style="margin-bottom:16px;font-size:15px;background-color:#E1FEE5;color:rgb(45,204,112);border-radius:5px;padding:2px 3px;">ObjectProvider</span>를 사용하여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">요청이 올 때마다</span>, Provider가 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">필요한 빈을 스프링 컨테이너에서 조회할 수 있도록 만든다.</span>  
⚠️ 사실 이러한 Logger는 컨트롤러보다는 <span style='color:rgb(196,58,26);'>공통처리가 가능한</span> `스프링 인터셉터`나 `서블릿 필터`같은 곳에서 활용하는 것이 좋다.

🍍 인터셉터  
: <span style='color:rgb(196,58,26);'>컨트롤러 호출 직전</span>에 공통화한 로직을 처리할 수 있다.

`☕ LogDemoService.java`

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
    private final ObjectProvider<MyLogger> myLoggerProvider;

    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id=" + id);
    }
}
```

마찬가지로 **requestURL**과 같은 웹과 관련된 정보는 <span style='color:rgb(196,58,26);'>서비스 계층까지 넘어가지 않는 게 좋다.</span>  
즉, 서비스 계층은 <span style='color:rgb(196,58,26);'>웹기술에 종속되지 않은 채</span>, 비즈니스 로직을 처리하고  
왠만하면 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">웹 관련 정보는 컨트롤러 단에서 처리하도록 한다.</span> `🏆유지보수성`

`☕ MyLogger`의 멤버변수에 **requestURL**과 같은 정보를 저장하여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#E1FEE5;">코드와 계층을 깔끔하게 유지할 수 있다.</span>

## 스코프와 프록시

<div style="margin-bottom:15px;font-size:20px;background-color:#9E0ADD;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 CGI 라이브러리로 특정 클래스를 상속받은 가짜 프록시 객체를 만들어 주입할 수도 있다
</div>

`☕MyLogger.java`

```java
@Component
@Scope(value="request", proxyMode="ScopeProxyMode.TARGET_CLASS")
public class MyLogger {
    private String uuid;
    private String requestURL;

    // ...생략
}
```

다음과 같이 어노테이션 설정을 변경해주면 원본객체를 프록시 객체로 대체할 수 있다.

`☕ LogDemoController.java`

```java
public class LogDemoController {
    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestUrl = request.getRequestURL().toString();
        myLogger.setRequestURL(requestUrl);

        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}
```

빈이 생성되고, 의존관계가 주입될 때 `myLogger`에 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">가짜 프록시 클래스를 만들어두고 주입해준다.</span>  
<span style='color:rgb(196,58,26);font-size:16px;'>MyLogger$$EnhancerBySpringCGLIB$$...</span>

가짜 프록시 객체는 <span style='color:rgb(196,58,26);'>실제 요청이 올때 그제서야</span> <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">내부에서 진짜 빈을 요청하는 위임로직을 호출한다.</span>  
호출하는 시점에 **진짜 객체를 찾아 동작한다.**  
이 때, 가짜 프록시 객체는 `request Scope🦚`와는 <span style='color:rgb(196,58,26);'>관계가 없다.</span>  
내부에 단순히 위임로직만 있고, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">싱글톤처럼 동작한다.</span>

프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯 `request Scope🦚`를 사용할 수 있다.  
이 프록시 객체를 사용하는 클라이언트는 사실 주입받은 객체가 원본인지 아닌지도 모르게 사용한다.

<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#E1FEE5;">핵심은 진짜 객체 조회를 꼭 필요한 시점까지 지연처리한다는 점이다.</span>

`☕ LogDemoService.java`

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
    private final MyLogger myLogger;

    public void logic(String id) {
        myLogger.log("service id=" + id);
    }
}
```
