---
title: "[ts] 에러를 사전에 방지하기 위한, 타입스크립트와 스펙을 정의하는 ts의 인터페이스"
date: 2024-12-05 14:37 +0900
lastmod: 2024-12-08 14:37 +0900
categories: ts
tags:
  [
    compile,
    typing,
    enum,
    tsconfig.json,
    optional parameter,
    type,
    interface,
    extends,
    type guard,
    class,
    prototype
  ]
---

## 타입스크립트를 사용하는 이유

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 타입스크립트는 자바스크립트에 타입을 부여한 언어이다
</div>

데이터에 타입이 정의되어 있을 때, 다음과 같은 효과를 볼 수 있다.

- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 에러를 사전에 방지</span>
- <span style="padding:0 3px;font-size:16px;font-weight:normal;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 코드 가이드 및 자동완성</span>
  : 해당 타입이 제공하는 api를 preview로 노출해준다.

js와는 다르게 <span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">ts는 브라우저에서 실행하기 위해</span> 파일을 브라우저가 인식할 수 있는 형태의  
<span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">js 파일로 변환</span>해 주어야 하는데, <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">이러한 변환과정을 컴파일</span>이라 한다.

`tsc (tyscript compiler)` 명령어를 실행하기 위해서 관련 라이브러리를 설치해야 한다.  
`npm i typescript -g`

<span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">타입이 정의되지 않은 코드에 타입을 입혀주는 것을 타이핑</span> `(typing)`이라고 하며,  
타입 어노테이션은 `:` 이다.

ts의 타입에는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">문자열</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">숫자</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">배열</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">튜플</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">객체</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">불리언</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">any</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">void</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">never</span> 등이 있다.  
any 타입은 모든 타입을 통칭하는데, 해당 타입으로 정의하는 것은 별 의미가 없다.  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">변수에 대한 구체적인 타입을 정의하는게, 타입스크립트를 사용하는 이유이자 에러를 줄이는 방법이다.</span>

```ts
// 배열
let arr: number[] = [1, 2, 3]; // let arr: Array<number> = [1, 2, 3]

// 튜플: 배열의 특정 인덱스 타입까지 정의
let address: [string, number] = ["gangnam", 100];

// 객체
let obj: object = {
  name: "peter",
  age: 30
};
```

객체의 타입을 정의할 때 위와같이 `object`로 정의해도 되지만,  
이렇게 정의하면 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">객체의 형상</span>이 드러나지 않는다.

```ts
// 객체의 형상이 드러나게 정의
let obj: { name: string; age: number } = {
  name: "peter",
  age: 30
};
```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;color:#34343c;">
<span style="font-weight:bold;">📕 이넘(enum) 타입</span><br>
<span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">특정값들의 집합을 의미하는 자료형</span>이다.<br>
별도의 값으로 초기화하지 않으면 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">숫자형 이넘(enum)</span>으로 취급한다.<br><br>
인자를 이넘(enum)으로 정의하면 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">이넘(enum)에서 제공하는 값만을 넘길 수 있다.</span>
</div>

```ts
enum Answer {
  Yes = "Y",
  No = "N"
}

function askQuestion(anser: Answer) {
  if (answer == Answer.Yes) {
    console.log("정답입니다.");
  }

  if (answer == Answer.No) {
    console.log("오답입니다.");
  }
}

askQuestion(Answer.Yes); // (ㅇ)
askQuestion("Yes"); // error!
askQuestion("Y"); // error!
```

### js 유연함 / ts 엄격함

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;font-weight:bold;">
    🥯 js 유연함 &nbsp;/&nbsp; ts 엄격함
</div>

자바스크립트에서는 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">정의된 함수 스펙의 파라미터 개수보다 많은 인자를 넣어 호출해도 문제가 안된다.</span>  
이를 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">자바스크립트의 유연함</span>이라 하는데, <span style='color:rgb(196,58,26);'>타입스크립트는 이를 허용하지 않는다.</span>

대신에, 타입스크립트는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">옵셔널 파라미터</span> `?` <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">(= 선택적 파라미터)</span>를 제공하는데,  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">이는 타입스크립트에 어느정도의 유연함을 제공한다.</span>

```ts
function logText(a: string, b?: string, c?: string) {
  // ...
}
```

타입정의 시 `?`를 추가적으로 붙인 인자는 호출할 때, 있을 수도 있고, 없을 수도 있다는 것을 뜻한다.

## 타입스크립트 설정파일, tsconfig.json

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 매번 tsc 명령어를 돌리는 것보다는, 웹팩으로 반복적으로 사용하는 명령어를 자동화할 수 있다
</div>

ts 명령어를 쓸 때, 부가적인 옵션을 타입스크립트 설정파일 `🎳 tsconfig.json`에 설정해둘 수 있다.

```json
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": true,
    "noImplicitAny": true,
    "strict": true,
    "strictFunctionTypes": true
  },
  "include": ["./src/**/*"]
}
```

- `checkJs: true`  
  : @ts-check, 타입스크립트의 타입검사 기능을 사용하겠다.
- `noImplicitAny`  
  : 명시적으로 기본적인 타입인 any라도 정의해야 한다.
- `strict`  
  : 타입스크립트의 엄격한 타입검사를 강제한다.

## 스펙(구조)을 정의하는 인터페이스

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 객체의 형상을 일일이 나열해서 각각 타입을 정의하면, 중복코드가 많아진다.
</div>

<span style="font-weight:700;font-size:1.03rem;">🗽 중복된 코드를 줄이기 위해 객체의 스펙을 정의하면 된다.</span>

(1) 타입 별칭 `(type)` 사용  
 : 새로운 타입을 생성하는 것이 아닌, <span style='color:#0072BB;'>정의된 구조에 이름을 그저 부여하는 것</span>이다.  
 : 즉, type을 사용하는 것은 특정 타입이나, 인터페이스를 참조할 수 있는 <span style='color:#0072BB;'>타입변수를 만드는 것</span>이다.

```ts
var Name = string;
const myName: Name = "peter";

type Person = {
  name: string;
  age: number;
};
```

(2) 인터페이스 `(interface)` 사용  
 : <span style='color:#652DC1;'>새롭게 정의한 인터페이스 타입을 생성</span>한다.  
 : `extends` 키워드를 사용해 <span style='color:rgb(196,58,26);'>확장(상속)할 수 있다.</span>

```ts
interface Person {
  name: string;
  age: number;
}

interface Developer extends Person {
  skill: string;
}
```

<span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">인덱싱 방식을 인터페이스에 정의</span>할 수 있다.  
그때그때마다 속성(index, key)을 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">타입만 맞도록, 임의로 부여</span>해서 사용할 수 있다.

```ts
interface StringArr {
  [index: number]: string;
}

// 인터페이스 딕셔너리 패턴
interface StringRegDictionary {
  [key: string]: RegExp; // 정규표현식 타입
}

var obj: StringRegDictionary = {
  cssFile: /\.css$/,
  jsFile: /\.js$/
};
```

<span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">함수의 스펙(구조)에 대해서도 인터페이스에 정의</span>할 수 있다.

```ts
interface SumFn {
  (a: number, b: number): number;
}

var sum: SumFn = function (a: number, b: number): number {
  return a + b;
};
```

type은 상속이 안되지만, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">interface는 상속이 가능하다.</span>  
따라서, type보다는 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">interface를 사용하는 것을 권장한다.</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 좋은 소프트웨어는 언제나 확장이 용이해야한다는 원칙을 따른다.</span>

## 유니온 타입과 공통속성

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🌋 유니온 타입을 이용해서 하나 이상의 타입을 정의할 수 있다.
</div>

<span style="font-weight:700;font-size:1.03rem;">☄️ 유니온 타입 | 을 쓰면 공통속성에만 접근할 수 있다.</span>

```ts
interface Person {
  name: string;
  age: number;
}

interface Developer {
  name: string;
  skill: string;
}

function askSomeone(someone: Person | Developer) {
  console.log(someone.name); // (ㅇ)
  console.log(someone.skill); // error!
}
```

유니온을 이용해 타입을 정의한 인자에는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 확실하게 둘 중 어떤 타입이 들어올지 모른다.</span>  
혹여나, 공통속성이 아닌 다른 속성을 쓸 수 있게 한다면
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 타입세이프하지 않은, 즉 에러가 날 수 있는 여지가 있기 때문에</span> 이를 막아놨다.

만일, 나머지 속성을 이용하고 싶으면 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">타입가드에 대한 추가적인 처리</span>가 필요하다.

🍍 타입가드  
 : 특정타입으로 타입의 범위를 좁혀나가는 과정

`🗿 type-guard.ts`

```ts
function askSomeone(someone: Person | Developer) {
  console.log(someone.name);
  if (typeof someone == "Person") {
    console.log(someone.age);
  }
  if (typeof someone == "Developer") {
    console.log(someone.skill);
  }

  throw new TypeError("someone must be Person or Developer");
}
```

🍍 인터섹션 타입 `&`  
 : 인터섹션을 이용해 타입을 정의한 인자에는 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">정의한 타입이 가지고 있는 모든 속성을 포함하는 인자만 올 수 있다.</span>  
 : 결과적으로는, 두개 이상의 인터페이스를 합친 <span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">새로운 인터페이스 타입을 생성하게 된다.</span>

```ts
const nth: string & number & boolean; // :never

function askSomeone(someone: Person & Developer) {
  console.log(someone.name);
  console.log(someone.age);
  console.log(someone.skill);
}

const someone = {
  name: "peter",
  age: 30,
  skill: "ts"
};

askSomeone(someone);
```

## ES6, 클래스와 프로토타입의 관계

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 자바스크립트는 프로토타입 기반의 언어이다
</div>

<span style="font-weight:700;font-size:1.03rem;">🎯 클래스를 사용하는 이유</span>

`🐝 class-ex.js`

```js
const user_before = { name: "peter", age: 30 };
const admin_before = { name: "jack", age: 50, role: "admin" };

class User {
  constructor(name, age) {
    // 초기화 메서드 (생성자)
    console.log("--created--");
    this.name = name;
    this.age = age;
  }
}

class Admin extends User {
  constructor(name, age, role) {
    super(name, age);
    this.role = role;
  }
}

const user = new User("peter", 30);
const admin = new Admin("jack", 50, "admin");
```

클래스는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">상속</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">캡슐화</span>, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">다형성</span> 같은 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">객체 지향 프로그래밍</span>의 주요 개념을 지원하여
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">🏆 재사용 가능한 코드</span>를 작성할 수 있게 해준다.

또한, 관련있는 속성과 메서드를 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">한 곳에 모아 논리적 단위로 관리할 수 있고 (모듈화)</span>,  
접근자를 통해 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">데이터를 캡슐화할 수 있다.</span>

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">정적 메서드</span>를 이용해, 유틸리티 함수를 만드는데 유용하다.

`🐝 util.js`

```js
class MathUtils {
  static add(a, b) {
    return a + b;
  }
}

console.log(MathUtils.add(3, 5));
```

js에서 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">프로토타입은 클래스 문법이 도입되기 전부터 사용되었다.</span>  
<span style='color:#652DC1;'>모든 객체는 자신의 프로토타입을 참조</span>하고, 이를 통해 <span style='color:#652DC1;'>상속이 이루어진다.</span>

`🐝 prototype-ex.js`

```js
function User(name, age) {
  this.name = name;
  this.age = age;
}

// 모든 함수는 자동으로 prototype 속성을 가지며, 이를 통해 메서드를 정의한다.
User.prototype.greet = function () {
  console.log(`Hi, my name is ${this.name}`);
};

const user = new User("peter", 30);
user.greet();
```

객체는 `__proto__` 또는 `[[Prototype]]`을 통해 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">부모 객체(프로토타입)에 접근한다.</span>  
js의 객체는 내부적으로 `[[Prototype]]`이라는 링크를 가지고 있다.  
<span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">이 링크를 통해 상위 객체를 참조</span>하며, `__proto__`는 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">이 링크를 노출한 속성</span>이다.

```js
// 예시1
const obj = { a: 10 };
obj;
// a: 10
// [[Prototype]]: Object

// 기본적으로 객체를 만들면 상위 prototype이 Object라는 최상위 객체가 있다.
// 때문에 Object 객체 안의 속성 및 메서드를 사용할 수 있다.

const arr = [];
arr;
// length: 0
// [[Prototype]]: Array(0)

// 예시2
const user = { name: "peter", age: 30 };
const admin = {};

admin.__proto__ = user; // 프로토타입의 상위에 user 객체를 정의
amdin.role = "admin";

console.log(admin.name); // 상위 오브젝트 속성 및 메서드를 가져다 쓸 수 있다.

// 예시3
const parent = {
  greet() {
    return "Hello from parent!";
  }
};

const child = Object.create(parent); // child의 [[Prototype]]이 parent를 참조
console.log(child.greet()); // 프로토타입 체인을 따라 child.greet() 호출 가능
console.log(child.__proto__ === parent); // true
```

<span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">사실 js의 클래스도 내부적으로 프로토타입을 기반으로하고, 프로토타입 기반의 상속도 유지한다.</span>  
즉, <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">클래스</span>는 단순히 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">프로토타입을 간편하게 추상화한 문법적 설탕 (Syntactic Sugar)</span>이다.

## 타입스크립트 클래스 문법

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 타입스크립트 클래스 문법
</div>

ts의 클래스에서는 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">사용할 멤버변수를 정의</span>해야하며,  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">변수의 접근범위나 유효동작</span>을 지정할 수 있다. `default, public`

```ts
clas User {
  private name: string;
  private age: number;
  readonly log: string;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

다만, 점차 클래스 기반 코드를 볼일이 없어진다.

- <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">react</span>
  - 예전문법: 클래스 기반 코드 extends React.Component
  - 최신문법: hook 기반의 함수형 코드
- <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">vue Composition API</span>
