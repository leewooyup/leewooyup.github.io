---
title: "[Spring] 웹 애플리케이션에서 사용하는 싱글톤과 이를 기반하여 객체를 관리하는 스프링 컨테이너 및 무상태성 (stateless)"
date: 2024-04-19 16:53 +0900
lastmod: 2024-04-20 16:53 +0900
categories: Spring
tags: [Singleton, static, final, stateless, Configuration, Spring Container]
---

## 웹 애플리케이션과 싱글톤

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 웹 애플리케이션에서 싱글톤이 많이 사용되는 이유
</div>

웹 애플리케이션은 보통 여러 고객이 동시에 요청을 할 수 있는 가능성이 있다.  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">요청이 올 때마다, 새로운 객체를 생성</span>한다면 메모리 낭비가 심하게 된다.

예를 들어, 고객 트래픽이 초당 200이 나오면 초당 200개의 객체가 생성되고 소멸된다.  
싱글톤 패턴으로 객체를 생성하면 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#E1FEE5;">해당 객체가 딱 한개만 생성되고, 생성된 객체 인스턴스를 공유한다.</span>

스프링 애플리케이션은 대부분 웹 애플리케이션이다.  
즉, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">스프링은 객체를 싱글톤으로 생성하도록 설계되어 있다.</span> <span style="margin-bottom:15px;font-size:15px;background-color:#99CCFF;color:#1196C1;font-weight:bold;border-radius:5px;padding:2px 3px;">스프링 빈</span>

## 싱글톤 패턴

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 해당 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 생성 패턴이다
</div>

(1) <span style='color:rgb(196,58,26);'>private 생성자</span>를 만들어, 외부에서 객체를 마음대로 생성하지 못하게 막는다.  
(2) <span style='color:rgb(196,58,26);'>멤버필드를 static으로 하나만 선언</span>하고, 내부에서 객체를 생성한다.  
(3) 이 멤버필드를 가져다 쓸 수 있는 <span style='color:rgb(196,58,26);'>public 메서드</span>를 만든다.

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;"><span style="font-weight:bold;">📕 기타제한자 static 멤버란</span><br>
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">객체 생성없이 접근 가능한 멤버</span>이며, 애플리케이션 실행 시, <span style='color:rgb(196,58,26);'>메모리에 무조건 올라간다.</span><br>
즉, <span style='color:rgb(196,58,26);'>클래스 level 당</span>, 딱 하나만 만들어서 이후 생성될 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">인스턴스끼리 공유</span>할 수 있다.<br>
⚠️instance 멤버는 객체 level에 소속된 멤버로써, 객체를 생성해야만 사용할 수 있는 멤버이고, 객체 간에 공유되지 않는다.
</div>

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;"><span style="font-weight:bold;">📘 기타제한자 final 멤버란</span><br>
한번 final 키워드로 선언되면, <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">초기화가 필수이며 이후의 값변경이 불가능하다.</span> (즉, <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">일종의 상수역할</span>)<br>
메서드에 final 키워드를 사용하면 <span style='color:rgb(196,58,26);'>상속될 수 없고, 오버라이딩(재정의)될 수 없다.</span><br>
☄️ <B>진정한 상수란</B><br>
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">static final int CONSTANT = 100;</span><br>
<span style='color:rgb(196,58,26);font-weight:bold;'>static</span>과 <span style='color:rgb(196,58,26);font-weight:bold;'>final</span>이 같이 붙은 변수가 마치 진정한 상수처럼 동작한다고 할 수 있다.<br>
상수란 누가 써도 같아야 진정한 상수이다.<br>
static을 붙이지 않으면 사용자마다 다른값으로 초기화해서 그 값이 다를 수 있다.<br>
즉, static을 붙여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">하나만 만들어 공유</span>하고, final을 붙여 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">무조건 초기화하여 변경할 수 없게 만든다.</span>
</div>

```java
public class SingletonService {
    private static final SingletonService instance = new SingletonService();
    // 클래스 level에 올라가기 때문에 딱 하나만 존재하게 된다
    // static 영역에 객체 인스턴스를 미리 하나 생성해서 올려둔다

    private SingletonService() {
        // private 생성자로 외부에서 객체 생성을 막는다
    }
    public static SingletonService getInstance() {
        return instance; // 항상 같은 인스턴스를 반환
    }

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```

`스프링 컨테이너🥥`가 기본적으로 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">객체를 싱글톤으로 만들어 관리</span>한다.  
때문에, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">이미 만들어진 객체를 공유</span>해서 효율적으로 사용할 수 있다.

스프링은 CGLIB를 사용해서 AppConfig 클래스를 상속받은 <span style='color:rgb(196,58,26);font-weight:bold'>\* </span>임의의 클래스를 만들고, 해당 클래스를 스프링 빈으로 등록한 것이다.  
 이 <span style='color:rgb(196,58,26);font-weight:bold'>\* </span>임의의 클래스의 내부로직으로 싱글톤을 보장해준다.

⚠️ 스프링 기본 등록방식은 싱글톤이지만, **빈 스코프**를 설정하여 요청 시마다 새로운 객체를 생성해서 반환할 수도 있다.

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;"><span style="font-weight:bold;">📘 CGLIB (Code Generation Library)란</span><br>
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#dee2e6;">자바 바이트코드 조작 라이브러리</span>로써, 주로 <span style='color:rgb(196,58,26);'>프록시 객체를 만드는데 사용</span>된다.<br>
즉, 런타임에 클래스를 생성하고 변경할 수 있다.<br>
이는, 상속을 통해 <span style='color:rgb(196,58,26);'>클래스의 기능을 확장</span>할 수 있도록 한다.<br>
⚠️ 단, 자바 바이트 코드를 생성하고 조작하기 때문에, <span style="margin-bottom:15px;font-size:15px;background-color:#ffdce0;color:rgb(196,58,26);font-weight:bold;border-radius:5px;padding:2px 3px;">런타임시 성능 오버헤드가 발생할 수 있다.</span>
</div>

🍪 싱글톤의 문제점

- 클라이언트가 구체 클래스에 의존한다. <span style="margin-bottom:15px;font-size:15px;background-color:#ffdce0;color:rgb(196,58,26);font-weight:bold;border-radius:5px;padding:2px 3px;">DIP 위반</span>  
  : 때문에 <span style='color:rgb(196,58,26);'>OCP 원칙을 위반</span>할 가능성도 높다.

- 스프링 컨테이너가 관리하기 때문에 테스트하기 어렵다.  
  : 내부 속성을 변경하거나 초기화하기 어렵다.

- `private 생성자`로 자식 클래스를 만들기 어렵다.  
  : 유연성이 떨어진다.

`스프링 컨테이너🥥`는 이러한 싱글톤 패턴의 문제점을 내부적으로 해결하면서,  
객체 인스턴스를 싱글톤으로 관리한다.  
싱글톤으로 객체를 생성하고 관리하는 작업을 **싱글톤 레지스트리**에서 하고,
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">이 싱글톤 레지스트리를</span> `스프링 컨테이너🥥`<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">라 한다.</span>

## 싱글톤과 무상태성(stateless)

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🏈 싱글톤 생성패턴은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 상태를 유지하도록 설계하면 안된다
</div>

즉, <span style="margin-bottom:15px;font-size:16px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">무상태(stateless)</span>로 설계해야 한다.  
<span style="margin-bottom:15px;font-size:16px;background-color:#ffdce0;color:rgb(196,58,26);font-weight:bold;border-radius:5px;padding:2px 3px;">⛔ 객체가 특정 클라이언트에 의존적인 필드가 있으면 안된다.</span> 있더라도 가급적 읽기만 가능해야 한다.  
필드 대신 자바에서 공유되지 않는 <span style="margin-bottom:15px;font-size:16px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">지역변수</span> <span style="margin-bottom:15px;font-size:16px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">파라미터</span> <span style="margin-bottom:15px;font-size:16px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">ThreadLocal</span> 등을 사용해야 한다.

🍪 if, 스프링 빈의 필드에 공유값을 설정하면 큰 장애가 발생할 수 있다.

`☕ StatefulService.java`

```java
public class StatefulService {
    private int price;

    public void order(String name, int price) {
        System.out.println("name= " + name + " price=" + price);
        this.price = price;
    }

    public int getPrice() {
        return price;
    }
}
```

`🦕 StatefulServiceTest.java`

```java
class StatefulServiceTest {

    @Test
    @DisplayName("상태를 유지하도록 클래스를 설계하면 데이터 오류가 생길 수 있다")
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        // 웹에서 요청이 오면, Thread가 할당된다
        // ThreadA: 사용자A 10000 주문
        statefulService1.order("userA", 10000);

        // ThreadB: 사용자B 20000 주문
        statefulService2.order("userB", 20000);

        int price = statefulService1.getPrice();
        // statefulService1 과 statefulService2는 같은 객체다
        assertThat(price).isEqualTo(20000);
    }

    static class TestConfig {
        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```

사용자A가 주문한 금액은 10000원임에도,  
`getPrice()`를 호출하면 20000원으로 변경되어 있음을 확인할 수 있고,  
이는 `스프링 컨테이너🥥`에 의해 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">싱글톤으로 관리되는 객체가 상태를 유지하도록 잘못 설계</span>되었기 때문이다.  
따라서, 스프링의 클래스는 항상 <span style="margin-bottom:15px;font-size:16px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">무상태(stateless)</span>로 설계한다.

```java
public class StatelessService {
    public int order(String name, int price) {
        System.out.println("name= " + name + " price= " + price);
        this.price = price;
        return price;
    }
}
```

## @Configuration과 싱글톤 보장

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🏈 @Configuration을 클래스에 붙이지 않으면 스프링 컨테이너에서 관리하지 않아 싱글톤이 깨진다
</div>

`@Configuration`을 붙이면 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">CGLIB 기술을 이용해서 싱글톤을 보장</span>한다.  
즉, `@Bean`만 사용해도 <span style='color:rgb(196,58,26);'>스프링 빈으로 등록</span>되지만, 싱글톤을 보장하지 않는다.  
⚠️ 이 때, 의존관계 주입을 위해 메서드를 직접 호출할 때, <span style="margin-bottom:16px;font-size:15px;background-color:#ffdce0;color:rgb(196,58,26);font-weight:bold;border-radius:5px;padding:2px 3px;">⛔ 싱글톤을 보장하지 않는다.</span>
