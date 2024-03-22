---
title: "[자바8] 인스턴스화 하지 않고도 메서드에 접근할 수 있는 메서드 참조와 함수형 인터페이스의 유틸리티 메서드"
date: 2024-03-22 12:45 +0900
lastmod: 2024-03-22 12:45 +0900
categories: 자바8
tags:
  [
    Method Reference,
    Constructor Reference,
    Lambda,
    Anonymous Class,
    Funtional Interface
  ]
---

## 메서드 참조

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🐁 <span style='color:#fff5b1;'>메서드참조</span>, 특정 람다표현식을 축약한 것
</div>

⚠️ 실제로 메서드를 호출하는 것이 아님으로 **괄호는 필요없다.**

```java
(args) -> ClassName.staticMethod(args)
ClassName::staticMethod

(arg0, rest) -> arg0.instanceMethod(rest)
ClassName::instanceMethod

(args) -> expr.instanceMethod(args)
expr::instanceMethod
```

## 생성자 참조

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🐀 클래스명과 new키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.
</div>

🎯 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">인스턴스화하지 않고도 생성자에 접근</span>할 수 있다.

`📜 Supplier.java`  
: 아무런 인자를 받지 않고, T 타입 객체 리턴 `() -> T`, 공급자

```java
 @FunctionalInterface
 public interface Supplier<T> {
    T get();
}
```

```java
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get(); // Supplier의 get method를 호출해서 Apple 객체를 만들 수 있다.
```

`📜 Function.java`  
: T 타입 인자를 받아서 R 타입 객체를 리턴 `T -> R`, 수학에서의 함수

```java
@FuntionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

```java
// ex1.
Function<Integer, Apple> c2 = Apple::new;
Apple a2 = c2.apply(110); // Function의 apply method에 무게를 인수로 호출해서 Apple 객체를 만들 수 있다.

//ex2.
List<Integer> weights = Arrays.asList(7, 3, 4, 10);
List<Apple> apples = map(weights, Apple::new);

public List<Apple> map(List<Integer> list, Function<Integer, Apple> f) {
    List<Apple> result = new ArrayList<>();
    for(Integer i : list) {
        result.add(f.apply(i));
    }
    return result;
}
```

## 코드 전달 -> 익명클래스 -> 람다표현식 -> 메서드 참조

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🍪 <span style='color:#fff5b1;'>자바8</span>, List.sort method에 정렬 전략을 간단하게 전달하는 법
</div>

`List.sort` method는 `Comparator` 객체를 인수로 받아 두 객체를 비교한다.  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">객체 안에 동작을 포함시키는 방식</span>으로 정렬 전략을 전달하고,  
sort에 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">인수로 전달된 정렬 전략에 따라</span> sort의 동작이 달라지므로,  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">sort의 동작이 파라미터화 되었다</span> 할 수 있다.

sort함수의 시그니처: `void sort(Comparator<? super E> c)`

`📜 Step1`: 코드전달

```java
public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}

inventory.sort(new AppleComparator());
```

`📜 Step2`: 익명클래스

```java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```

`📜 Step3`: 람다 표현식  
: <span style='color:rgb(196,58,26);font-weight:bold'>\*</span><span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">추상메서드의 시그니처</span>는 <span style='color:rgb(196,58,26);font-weight:bold'>\*</span><span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">람다 표현식의 시그니처를 정의</span>한다.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// ⚠️ Comparator는 Comparable 키를 추출해서 Comparator 객체로 만드는
// (Function함수를 인수로 받는) comparing 정적메서드를 포함한다.
import static java.util.Comparator.comparing;
inventory.sort(comparing((Apple a) -> a.getWeight()));
```

`📜 Step4`: 메서드 참조

```java
import static java.util.Comparator.comparing;
inventory.sort(comparing(Apple::getWeight));
// Apple을 weight별로 비교해서 inventory를 sort해라.

inventory.sort(comparing(Apple::getWeight).reversed());
// Apple을 weight별로 비교해서 inventory를 reverse sort해라.
```

## 함수형 인터페이스의 유틸리티 메서드

<div style="margin-bottom:15px;font-size:20px;background-color:#652DC1;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;">
    🐙 람다 표현식을 조합할 수 있는 유틸리티 메서드 (default method)
</div>

<span style='color:rgb(196,58,26);font-weight:bold;'>Comparator</span> 조합

> `Comparator.thenComparing`  
> 함수를 인수로 받아, 첫번째 비교자를 이용해서 두 객체가 같다고 판단되면 두번째 비교자에 객체를 전달

```java
inventory.sort(comparing(Apple::getWeight).reversed().thenComparing(Apple::getCountry));
// 두 사과의  무게가 같으면 원산지별로 정렬
```

<span style='color:rgb(45,204,112);font-weight:bold;'>Predicate</span> 조합

> `Predicate.negate`: Predicate 반전  
> `Predicate.and` `Predicate.or`: 연결해서 새로운 Predicate를 만든다 (두 람다 조합)

```java
Predicate<Apple> notRedApple = redApple.negate();
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(apple -> apple.getWeight > 150).or(apple -> GREEN.equals(apple.getColor())));
```

<span style='color:#6CA0DC;font-weight:bold'>Function</span> 조합

> `Function.andThen`: 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환  
> `Function.compose`: 인수로 주어진 함수를 먼저 실행한 다음, 그 결과를 외부 함수의 인수로 제공

```java
Function<String, String> addHeader = Letter::addHeader;
Function<String, String> transformationPipeline = addHeader.andThen(Letter::checkSpelling).andThen(Letter::addFooter);
```
