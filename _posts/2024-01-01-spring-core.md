---
title: "[Spring] 객체 지향 프로그래밍의 핵심인 다형성과 이를 위한 프레임워크, 스프링"
date: 2024-01-01 16:52 +0900
lastmod: 2024-02-24 19:46 +0900
categories: Spring
tages: [Polymorphism, Interface, DI, OCP, DIP, Spring, Spring Container]
---

## 객체 지향 프로그래밍이란

`🧠Abstraction`&emsp;`💊Encapsulation`&emsp;`🪆Inheritance`&emsp;`🦠Polymorphism`

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🐀 객체 지향의 핵심(core)은 다형성(Polymorphism)이다
</div>

애플리케이션을 **객체들의 모임**으로 파악하자  
각각의 객체는 메세지를 주고 받고 데이터를 처리할 수 있다.  
각각의 객체를 부품으로 보고,  
🎯 부품을 <span style='color:rgb(196,58,26);font-weight:bold'>쉽게 갈아 끼울 수 있게</span> 만드는 것이 **객체 지향의 핵심**이자, <span style='color:rgb(196,58,26);font-weight:bold'>다형성</span>이다.

## 다형성(Polymorphism)

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🐁 규격과 규격에 맞게 제작된 부품
</div>

다형성은 **쉽게 갈아 끼울 수 있게 만드는 것**이고,  
그렇게 만드려면 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">규격(인터페이스)</span>가 필요하다.

> 규격에 맞게 제작되면 갈아 끼워도 다른 것에 영향을 주지 않는다.

<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">다른 것에 영향을 주지 않는다</span>는 것은 내가 변화가 있을 때,  
그에 따른 **다른 것을 변경할 필요가 없는 것**을 말한다.  
이런 관계를 만드는 것을 <span style='color:rgb(196,58,26);font-weight:bold'>확장 가능한 설계</span>라고 하며,  
<span style='color:rgb(196,58,26);font-weight:bold'>\*</span>결국 규격(인터페이스)를 안정적으로 설계하는 것이 중요하다.

규격을 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">인터페이스</span>라 보고,  
규격에 맞게 제작된 부품을 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">구현 객체</span>로 본다.

혼자 있는 부품은 의미가 없듯이, **혼자 있는 객체만으로는 의미가 없다.**  
객체(클라이언트)와 객체(서버)는 <span style='color:rgb(196,58,26);font-weight:bold'>협력 관계</span>를 가진다.  
이 협력관계에서, 객체(클라이언트)에 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">영향을 주지 않고 객체(서버)의 기능을 변경할 수 있게 만드는 것</span>을  
🦠<span style='color:rgb(196,58,26);font-weight:bold'>다형성</span>이라 한다.

이렇게 하면, 구현객체를 **실행 시점에서 유연하게 변경할 수 있다.**  
또한, 객체(클라이언트)는 객체(서버)의 내부 구조를 몰라도 되니까 단순해진다.  
즉, **객체끼리의 협력관계**는 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>서로 규격(인터페이스)만을 알고 있다는 의미이고,  
이를 통해 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>의존적이지 않은 코드를 짤 수 있다는 뜻이다.

<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">🦁 ~에 의존한다 == ~를 알고있다</span>

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">🐁 좋은 객체 지향 설계의 5가지 원칙 (SOLID)</div>

- <span style='background-color:#EEEEEE;font-weight:bold'>SRP (Single Responsibility Principle): 단일 책임 원칙</span>

  > "하나의 클래스는 하나의 책임만 가져야 한다."
  >
  > > 변경이 있을 때, 파급효과🔥가 적으면 단일 책임 효과를 잘 따른 것.  
  > > 즉, 하나의 변경이 있을 때 하나의 클래스의 하나의 지점만 고치면 될 때  
  > > ⚠️ 책임의 범위를 잘 조정해야 한다.

- <span style='background-color:#EEEEEE;font-weight:bold'>OCP (Open/Closed Principle): 개방 폐쇄 원칙</span>

  > "확장에는 열려 있으나, 변경에는 닫혀있어야 한다."
  >
  > > 여기서 말하는 확장은 인터페이스를 구현한 새로운 구현 객체를 추가하는 것을 말하고,  
  > > 변경에 닫혀있다는 말은 새로운 기능 추가에 기존 코드(객체, 클라이언트)는 변경되지 않아야 하는 것을 말한다.  
  > > ⚠️ 이를 위해서는, 객체를 생성하고 연관관계를 맺어주는 별도의 조립, 설정자가 필요하다.

- <span style='background-color:#EEEEEE;font-weight:bold'>LSP (Liskov Substitution Principle): 리스코프 치환 원칙</span>

  > "구현 객체는 기능적인 보장을 해주어야 한다."
  >
  > > ⚠️ 컴파일 단계의 성공을 말하는 것이 아니다.

- <span style='background-color:#EEEEEE;font-weight:bold'>ISP (Interface Segregation Principle): 인터페이스 분리 원칙</span>

  > "작은 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다."
  >
  > > 더 쉽게 갈아 끼울 수 있다.

- <span style='background-color:#EEEEEE;font-weight:bold'>DIP (Dependency Inversion Principle): 의존관계 역전 원칙</span>
  > "추상화에 의존해야지, 구체화에 의존하면 안된다."
  >
  > > 인터페이스(규격)에만 의존해라. 즉, 객체(클라이언트)가 인터페이스만을 바라보라.  
  > > 객체(클라이언트)는 구현 객체는 몰라야한다.

⚠️ <span style='color:rgb(196,58,26);font-weight:bold'>🦠다형성만으로는 OCP, DIP를 지킬 수 없다.</span>

`오리.java`

```java
class 오리 {

  private Long id;
  private 오리타입 type;
  private String color;

  public 오리(Long id, 오리타입 type) {

    this.id = id;
    this.type = type;
    if(type == 오리타입.흰오리) {
      this.color = "White";
    } else if (type == 오리타입.청둥오리) {
      this.color = "Dark Green";
    } else {
      this.color = "Yellow";
    }
  }

  public Long getId() {
    return this.id;
  }

  public 오리타입 getType() {
    return this.type;
  }

  @Override
  public String toString() {
    return "{" +
      "id=" + id +
      ", type=" + type.getValue() +
      ", color='" + color + '\'' +
      '}';
  }
}
```

`오리타입.java`

```java
public enum 오리타입 {

  흰오리("White Duck"),
  청둥오리("Mallard"),
  고무오리("Rubber Duck");

  private String value;

  오리타입(String value) {
    this.value = value;
  }

  public String getValue() {
    return value;
  }
}
```

`아이템.java`

```java
public interface 아이템 {

  void 작동();
}
```

`비행아이템.java`

```java
public interface 비행아이템 extends 아이템 {}

```

`날개비행아이템.java`

```java
public class 날개비행아이템 implements 비행아이템 {

  @Override
  public void 작동() {
    System.out.println("The duck flies to the wings.");
  }
}
```

`못나는비행아이템.java`

```java
public class 못나는비행아이템 implements 비행아이템 {

  @Override
  public void 작동() {
    System.out.println("The duck can't fly.");
  }
}
```

`헤엄아이템.java`

```java
public interface 헤엄아이템 extends 아이템 {}
```

`물갈퀴헤엄아이템.java`

```java
public class 물갈퀴헤엄아이템 implements 헤엄아이템 {

  @Override
  public void 작동() {
    System.out.println("The duck swims on its web.");
  }
}
```

`둥둥헤엄아이템.java`

```java
public class 둥둥헤엄아이템 implements 헤엄아이템 {

  @Override
  public void 작동() {
    System.out.println("The duck floats around.");
  }
}
```

`오리Repository.java`

```java
public interface 오리Repository {

  void save(오리 duck);

  오리 findById(Long id);
}
```

`Memory오리Repository.java`

```java
public class Memory오리Repository implements 오리Repository {

  private static Map<Long, 오리> store = new HashMap<>();

  @Override
  public void save(오리 a오리) {
    store.put(a오리.getId(), a오리);
  }

  @Override
  public 오리 findById(Long id) {
    return store.get(id);
  }
}
```

`오리Service.java`

```java
public interface 오리Service {

  void 오리_저장(오리 a오리);

  오리 오리_가져오기(Long id);

  void 오리_작동();
}
```

`오리ServiceImpl.java`

```java
public class 오리ServiceImpl implements 오리Service {

  // 인터페이스에 의존하지만, 구현체에도 의존한다, 또한 다른 구현체로 갈아 끼우려면 이 코드를 변경해야 한다
  // private final 오리Repository a오리Repository = new Memory오리Repository();
  private final 오리Repository a오리Repository; // 인터페이스(추상화)에만 의존한다
  private final 비행아이템 a비행아이템;
  private final 헤엄아이템 a헤엄아이템;

  // 생성자를 이용한 (구현체)주입
  public 오리ServiceImpl(오리Repository a오리Repository, 비행아이템 a비행아이템, 헤엄아이템 a헤엄아이템) {

    this.a오리Repository = a오리Repository;
    this.a비행아이템 = a비행아이템;
    this.a헤엄아이템 = a헤엄아이템;
  }

  @Override
  public void 오리_저장(오리 a오리) {
    a오리Repository.save(a오리);
  }

  @Override
  public 오리 오리_가져오기(Long id) {
    return a오리Repository.findById(id);
  }

  @Override
  public void 오리_작동() {
    a비행아이템.작동();
    a헤엄아이템.작동();
  }
}
```

`오리공장.java`

```java
public class 오리공장 {

  public 오리Service a오리Service(오리타입 type) {
    return new 오리ServiceImpl(a오리Repository(), a비행아이템(type), a헤엄아이템(type));
  }

  public 비행아이템 a비행아이템(오리타입 type) {

    if (type == 오리타입.고무오리) {
      return new 못나는비행아이템();
    }

    return new 날개비행아이템();
  }

  public 헤엄아이템 a헤엄아이템(오리타입 type) {

    if (type == 오리타입.고무오리) {
      return new 둥둥헤엄아이템();
    }

    return new 물갈퀴헤엄아이템();
  }

  private 오리Repository a오리Repository() {
    return new Memory오리Repository();
  }
}
```

`오리App.java`

```java
public class 오리App {

  public static void main(String[] args){

    오리공장 a오리공장 = new 오리공장();
    오리Service a오리Service;
    오리 a오리;

    // 흰오리
    a오리 = new 오리(1L, 오리타입.흰오리);

    a오리Service = a오리공장.a오리Service(a오리.getType()); // 클라이언트 코드를 변경하지 않음
    a오리Service.오리_저장(a오리);

    오리 a흰오리 = a오리Service.오리_가져오기(1L);
    System.out.println("White Duck = " + a흰오리);
    a오리Service.오리_작동(); // 주입된 객체에 따라 작동이 다르다

    // 청둥오리
    a오리 = new 오리(2L, 오리타입.청둥오리);

    a오리Service = a오리공장.a오리Service(a오리.getType()); // 클라이언트 코드를 변경하지 않음
    a오리Service.오리_저장(a오리);
    System.out.println();

    오리 a청둥오리 = a오리Service.오리_가져오기(2L);
    System.out.println("Mallard = " + a청둥오리);
    a오리Service.오리_작동(); // 주입된 객체에 따라 작동이 다르다
    System.out.println();

    // 고무오리 생성
    a오리 = new 오리(3L, 오리타입.고무오리);

    a오리Service = a오리공장.a오리Service(a오리.getType()); // 클라이언트 코드를 변경하지 않음
    a오리Service.오리_저장(a오리);

    오리 a고무오리 = a오리Service.오리_가져오기(2L);
    System.out.println("Rubber  Duck = " + a고무오리);
    a오리Service.오리_작동(); // 주입된 객체에 따라 작동이 다르다
  }

//  출력
//  White Duck = {id=1, type=White Duck, color='White'}
//  The duck flies to the wings.
//  The duck swims on its web.
//
//  Mallard = {id=2, type=Mallard, color='Dark Green'}
//  The duck flies to the wings.
//  The duck swims on its web.
//
//  Rubber  Duck = {id=3, type=Rubber Duck, color='Yellow'}
//  The duck can't fly.
//  The duck floats around.
}
```

<span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:#E1FEE5;">'오리공장(AppConfig)'는 실제 동작에 필요한 구현 객체를 생성하고 연결하는 책임을 가지는 설정 클래스다.</span>  
이렇게 <span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:#ffdce0;">관심사를 명확히 분리</span>하고,  
생성자를 통해 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>외부에서 객체를 주입해주는 책임만을 가진 설정 클래스가 있기 때문에,  
오리ServiceImpl은 <span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">추상화에만 의존</span>할 수 있고, 이는 <span style='color:rgb(196,58,26);font-weight:bold'>DIP</span>를 지킬 수 있게 만든다.

또한, 오리의 비행 또는 헤엄 아이템을 확장하기 위해서, <span style="margin-bottom:15px;border-radius:5px;background-color:rgba(193,151,210,0.7);">오리공장(AppConfig)만 바꿔주면 되고</span>,  
오리ServiceImpl<span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">(객체, 클라이언트)은 변경하지 않아도 된다.</span> 이는 <span style='color:rgb(196,58,26);font-weight:bold'>OCP</span>를 지킬 수 있게 만든다.

이러한 방식으로, <span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:#E1FEE5;">다형성을 극대화하고 OCP, DIP를 지켜지도록</span> 만들어 주는 프레임워크가  
<span style='color:rgb(196,58,26);font-weight:bold'>스프링(Spring)</span>이다.  
스프링은 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>DI와 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>IoC 컨테이너를 제공함으로써 **의존관계를 외부에서 주입**해  
<span style="margin-bottom:15px;border-radius:5px;background-color:#ffdce0;">다형성 + OCP + DIP</span>를 지키는 코드를 짜도록 유도한다.

> 스프링이란 객체(클라이언트)의 코드 변경없이 구현 객체를 갈아 끼우는 방식으로  
> 기능을 확장시켜 개발하게 도와주는 프레임워크이다.

![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/QBH5VYCQs0.png "Optional title"){: width="500" height="300"}

`org.springframework:spring-core`

## 외부에서 객체를 주입해주는 책임을 가진 설정 클래스, Spring Container

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🍪 Spring Container
</div>

설정정보를 바탕으로 <span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">객체를 생성</span>하고, <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);"> (⚠️ Spring Container가 관리하는 객체를 Bean이라 한다.)</span>  
생성된 객체를 내부에 담아 <span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">라이프사이클 관리</span> 및 <span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">의존성 주입을 담당</span>하는 **컴포넌트**를 말한다.

🎯 객체를 낭비하지 않는다.

Spring Container는 크게 두가지 유형으로 나뉜다.

- BeanFactory
  : <span style="margin-bottom:15px;padding: 0 3px;border-radius:5px;background-color:#ffdce0;">Bean객체를 생성하고 제공</span>하는 역할을 한다.  
  : 생성된 Bean 객체는 Application 내에서 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>공유되어 재사용된다 **(Singleton Scope)**.  
  : 또한, 실제로 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>필요한 시점에서 초기화되고 **(Lazy Initialization)**.  
  : 객체 간 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>의존성을 자동으로 처리한다 **(Dependency Injection)**.  
  : BeanFactory를 통해 객체를 생성하고 연결하는 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>책임(관심사)을 분리하여 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span>모듈화할 수 있다 **(Aspect-Oriented Programming)**.

<div style="margin-bottom:15px;margin-left:40px;font-size:16px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:2px;"><span style="font-weight:bold;">📕 Bean 객체를 Singleton으로 만드는 이유</span><br>
: 매번 클라이언트에서 요청이 올 때마다 서버에서 해당 로직을 맡은 객체를 새로 만든다고 가정했을 때,<br>
누적되면 자원소모가 크다. (설사 Garbage Collection이 있다하더라도)<br>
⚠️ Singleton 개념은 객체 생성 측면에서 자원소모를 효율적으로 하기 위한 디자인패턴이다.
</div>
  
- ApplicationContext  
: ApplicationContext는 <span style='color:rgb(196,58,26);font-weight:bold'>BeanFactory를 구현</span>하고 있어 <span style="margin-bottom:15px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">BeanFactory의 확장된 버전</span>이다.  
: **확장된 기능**  
: `🍕 Environment`: 소스 설정 및 프로퍼티 값을 가져올 수 있다
: `🍕 MessageSource`: 메세지 설정파일을 모아, 로컬라이징을 통한 맞춤 메세지 제공

![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/EskS7rVcg8.png "Optional title"){: width="500" height="300"}

- `ClassPathXmlApplicationContext`: ClassPath에 지정한 경로에서 xml파일을 읽어 context 정의내용을 load
- `FileSystemXmlApplicationContext`: FileSystem에 지정한 경로에서 xml 파일을 읽어 context 정의내용을 load
- `XmlWebApplicationContext`: Web Application에 포함된 xml파일에서 context 정의내용을 load

## Spring의 설계 철학

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🐁 Spring Framework에 종속되지 않고 POJO 객체를 사용하여 애플리케이션을 개발할 수 있게 하는 것 
</div>

> 🍍 POJO (Plain-Old Java Object)  
> 특별한 제약이나 규칙을 따르지 않는 평범한 자바객체를 가리킨다.  
> Spring Framework는 Spring에 특화된 클래스를 요구하지 않으며,  
> 의존성 주입을 통해 POJO 객체를 연결할 수 있는 강력한 매커니즘을 제공한다.
