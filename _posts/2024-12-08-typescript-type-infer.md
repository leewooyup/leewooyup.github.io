---
title: "[ts] 재사용성을 위한 제네릭(Generic)과, 넘겨받은 타입을 통한 타입추론의 유용성 및 타입제한, 타입단언, 타입가드, 타입호환"
date: 2024-12-08 17:56 +0900
lastmod: 2024-12-08 17:56 +0900
categories: ts
tags:
  [
    Generic,
    type inference,
    union,
    language server,
    async,
    keyof,
    type assertion,
    DOM API,
    type guard,
    type compatibility,
    structural typing
  ]
---

## 제네릭(Generic)과 타입추론

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 제네릭은 재사용성이 높은 컴포넌트를 만들 때, 자주 활용되는 특징이다.
</div>

<span style="padding:0 3px;font-size:16px;font-weight:700;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🗣️: "호출하는 시점에 타입을 넘겨, 그 타입으로 쓰겠다."</span>

즉, <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">타입을 마치 파라미터 개념으로 받게 되는 것이 제네릭</span>이다.  
제네릭이 없다면 인자의 타입마다 메서드를 따로 선언해야 하는데, 이는 유지보수 관점에서 좋지 않다.

```ts
function logText(text: string): string {
  console.log(text);
}
return text;

function logNumber(num: number): number {
  console.log(num);
  return num;
}

// 제네릭 사용: 동일한 동작을 하는 함수를 타입마다 구현할 필요없다.
// <T>: 제네릭을 사용할 것을 암시
function logInfo<T>(info: T): T {
  console.log(info);
  return T;
}

logInfo<string>("hi");
logInfo<number>(10);
```

즉, 제네릭은 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">함수를 호출하는 시점에, 파라미터에 대한 타입을 지정하면서 넘긴다.</span>

```ts
function logInfo(info: string | number) {
  console.log(info);
  return info;
}
```

유니온 타입을 이용해도, 타입마다 메서드 선언을 따로 할 필요가 없다.

<span style="font-weight:700;font-size:1.03rem;">🍪 그럼에도 제네릭을 이용하는 것이 더 유용하다.</span>  
 : 유니온 타입의 문제점은 공통속성/api에 대해서만 접근이 가능하고,  
 <span style='color:rgb(196,58,26);'>반환값의 타입 또한 정확히 알수없기 때문에</span> 내장 속성/api를 사용할 수 없다.  
 &nbsp;  
 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">제네릭을 이용하면 호출받은 시점에 넘겨받은 타입을 이용해 타입추론이 가능하고</span>,  
 이로 인해 최종 반환값까지 타입을 붙일 수 있다.  
 &nbsp;  
 즉, 제네릭을 이용한 동기적인 코드에 대해서 ts가 타입 추론을 할 수 있고,  
 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">추론한 반환 타입에 따라 내장 속성/api를 사용할 수 있다.</span>

```ts
function logInfoByUnion(info: string | number) {
  console.log(info);
  return info;
}

var str1 = logInfoByUnion("hello world");
str1.split(" "); // (x)

function logInfoByGeneric<T>(info: T): T {
  console.log(info);
  return info;
}

var str2 = logInfoByGeneric<string>("hello world");
str2.split(" "); // (ㅇ)
```

---

`🗿 dropdown-ex.ts`  
 : 제네릭을 이용한 하나의 인터페이스로, 여러 타입을 커버할 수 있다.

```ts
interface Dropdown<T> {
  value: T;
  selected: boolean;
}

const obj: Dropdown<string> = { value: "abc", selected: false };
const obj: Dropdown<number> = { value: 1, selected: false };
```

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;">
<span style="font-weight:bold;">☑️ language server</span><br>
vscode의 ts를 지원하는 내장 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">language server</span>가 타입을 추론한다.<br>
이러한 추론 기능을 호함하는 기능을 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">intelliSense</span>라 하는데, 내부적으로 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">typescript language server</span>가 돌아야 한다.<br>
<span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">변수, 속성을 선언 및 초기화하거나</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">인자의 기본값 및 함수의 반환값을 설정할 때 타입추론이 일어난다.</span><br><br>

<span style='color:rgb(196,58,26);'>Best Common Type</span><br>
타입스크립트가 특정 코드가 어떤 타입인지 매겨나가는 알고리즘이라 보면된다.<br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">var arr = [1, 2, true]</span><br>
해당 코드를 ts는 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">(number | boolean)[]</span> 타입으로 추론한다.<br>
배열에 있는 값 중 가장 교집합이 될 수 있는 타입을 유티온 타입으로 지정해 나가는 방식이다.

</div>

## 비동기코드와 제네릭

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 비동기코드의 결과는 추론할 수 없기 때문에, 명시해준다
</div>

```ts
function fetchItems() {
  let items: string[] = ["a", "b", "c"];
  return new Promise((resolve) => {
    setTimeout(() => resolve(items), 3000);
  });
}
```

`vscode`에서 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">fetchItems의 preview</span>를 보면 반환값이 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">Promise&lt;Unknown></span>으로 추론한 것을 볼 수 있다.  
타입스크립트가 Promise 안의 타입은 잘 모르겠다고 추론한 것이다.  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">즉, ts가 Promise 안에 들어오는 비동기 코드에 대해서는 타입추론을 하지 못한다.</span>  
사실, Promise 내부적으로도 제네릭을 이용해서 정의되어 있다.  
때문에, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">비동기코드가 들어간 함수의 반환값을 명시적으로 정의해야 ts가 알 수 있다.</span>  
`function fetchItems(): Promise<string[]>`

<span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">비동기 코드에서 제네릭은 결국, 타입을 명시하고, 그 타입을 돌려받는 것이다.</span>

## 제네릭(Generic) 타입제한

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 사실, 타입을 제한한다는 것은 어떤 타입이 올 지 힌트를 더 주는 것이다.
</div>

- 기존타입으로 타입제한 하기

  ```ts
  function logTextLength<T>(text: T): T {
    console.log(text.length); // error!, 어떤 타입이 들어올지 알 수없다.
    return text;
  }

  // 타입제한
  function logTextLength<T>(text: T[]): T[] {
    // 들어올 타입이 속성에 무조건 length가 있을 것임을 암시
    console.log(text.length);
    text.forEach(function (t) {
      console.log(t);
    });
    return text;
  }

  logTextLength(["hi"]);
  ```

- 새로 정의된 타입으로 타입제한 하기

  ```ts
  interface LengthType {
    length: number;
  }

  function logTextLength<T extends LengthType>(text: T): T {
    console.log(text.length);
    return text;
  }

  logTextLength("a"); // (ㅇ)
  logTextLength({ length: 10 }); // (ㅇ)
  ```

- 제네릭 타입제한 예약어 keyof

  ```ts
  interface ShoppingItem {
    name: string;
    price: number;
    stock: number;
  }

  function getShoppingItemOption<T extends keyof ShoppingItem>(
    itemOption: T
  ): T {
    // 해당 인터페이스에 있는 키(속성) 중 하나가 넘어올 것이다, 타입 무관
    return itemOption;
  }

  getShoppingItemOption("name"); // (ㅇ)
  getShoppingItemOption("price"); // (ㅇ)
  getShoppingItemOption("coupon"); // (x)
  ```

## 타입단언, type assertion

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 개발자가 정의한 타입으로 간주해라
</div>

타입스크립트의 `language server`는 변수의 할당된 값의 타입이 바꼈을 때,  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 타입추론 방식으로 코드를 추적할 수 없다.</span>

```ts
let a;
a = 20;
var b = a;
// ts는 var a: any, var b: any로 추론한다.

// 타입단언
var b = a as string;
// ts는 var a: any, var b: string으로 추론한다.
```

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 보통 DOM API 조작에서 많이 사용된다.</span>

🍍 DOM API  
 : 웹페이지에서 tag정보에 접근하고, 조작할 수 있는 API를 말한다.  
 document 접근자에서 제공하는 속성 및 API이다.

`🗿 DOM-API_manipulation-ex.ts`

```ts
const div = document.querySelector("div"); // const div: HTMLDivElement | null
div.innerText; // error; Object is possibly 'null', div가 있다는 보장이 없다

if (div) {
  // 일반적인 패턴
  div.innerText;
}

// 타입단언
const divAssertion = document.querySelector("div") as HTMLDivElement; // 해당 타입이 있을 것이다.
divAssertion.innerText; // (ㅇ)
```

## 타입가드, type guard

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 타입가드란 특정타입으로 타입의 범위를 좁혀나가는 과정을 말한다
</div>

```ts
interface Developer {
  name: string;
  skill: string;
}

interface Person {
  name: string;
  age: number;
}

function introduce(): Developer | Person {
  return { name: "Peter", age: 30, skill: "ts" };
}

const peter = introduce();
console.log(tony.skill); //error!, 유니온타입을 쓰게 되면 타입들의 공통된 속성만 접근할 수 있다.

// 타입 단언
if ((tony as Developer).skill) {
  const skill = (tony as Developer).skill;
  console.log(skill);
} else if ((tony as Person).age) {
  const age = (tony as Person).age;
  console.log(age);
}
```

유니온 타입을 쓸 때의 단점을 타입단언으로 해결했지만,  
코드가 반복되고 가독성도 떨어진다.  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">이 때, 타입가드를 쓰면 해결된다.</span>

```ts
// 타입가드 정의
function isDeveloper(target: Developer | Person): target is Developer {
  // Developer로 취급
  return (target as Developer).skill !== undefined; // 해당조건을 만족할 때
}

if (isDeveloper(peter)) {
  console.log(tony.skill);
} else {
  console.log(tony.age);
}
```

## 타입호환, type compatibility

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 타입호환이란 특정 타입이 다른 타입에 잘 호환되는지를 말한다
</div>

타입스크립트 관점에서 해당 타입에 정의된 <span style='color:rgb(196,58,26);'>속성과 타입</span>을 가지고 호환 여부를 판정한다. - <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">구조적 타이핑(structural typing)</span>

```ts
// ex-1.
interface Developer {
  name: string;
}

class Person {
  name: string;
}

let d: developer;
d = new Person(); // Ok, because of structural typing

// ex-2.
interface Developer {
  name: string;
  skill: string;
}

interface Person {
  name: string;
}

let d: Developer = {
  name: "peter",
  skill: "ts"
};
let p: Person = {
  name: "peter"
};

developer = person; // (x)
person = developer; // (ㅇ)

// ex-3.
const add = function (a: number) {
  // 생략...
};

const sum = function (a: number, b: number) {
  // 생략...
};

add = sum; // (x)
sum = add; // (ㅇ)
```

변수 호환을 따질 때는,  
오른쪽에 있는 타입이 구조적으로 더 컸을 때, 왼쪽과 호환된다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">(왼쪽 타입의 속성을 오른쪽 타입이 가지고 있다면)</span>

함수의 호환을 따질 때는,  
오른쪽에 있는 함수의 파라미터가 구조적으로 더 작았을 때, 왼쪽과 호환된다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">(오른쪽에 있는 함수의 파라미터를 왼쪽 함수의 파라미터가 가지고 있다면)</span>
