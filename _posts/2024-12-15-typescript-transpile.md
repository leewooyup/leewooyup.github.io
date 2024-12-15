---
title: "[ts] 점진적으로 타입스크립트를 적용해 나가는 프로세스 및 프로젝트 환경 구성, 편리한 유지보수를 위한 유틸리티 타입"
date: 2024-12-15 23:40 +0900
lastmod: 2024-12-15 23:40 +0900
categories: ts
tags:
  [
    eslint,
    prettier,
    babel,
    npm,
    preset,
    index.d.ts,
    Definitely Typed,
    type assertions,
    optional chaining operator,
    DOM API,
    Partial,
    Pick,
    Mapped Type
  ]
---

## 프로젝트 환경 구성 (eslint + prettier, babel)

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 eslint란 js code에서 직접접인 에러와 에러가 날만한 잠재적인 부분들을 code level에서 줄여주는 코드 문법 보조 도구이다
</div>

`eslint`는 <span style='color:rgb(196,58,26);'>직/간접적인 에러를 체킹</span>해줄 뿐만 아니라, 코드 자동완성과 포맷팅까지 할 수 있다.  

하지만, eslint가 `code formatter`가 있음에도 불구하고, 별도의 `code formatter`인 `prettier`를 사용하는데,  
이는 `prettier`가 개인화된 포맷팅을 지정할 수 있기 때문이다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">prettier: an opinionated code formatter</span>  

🎯 prettier  
  : 팀단위 코딩 컨벤션을 만들어 코드를 정의할 때 자주 사용된다. <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">가독성🏆</span>  

즉, `eslint`를 `prettier`와 함께 쓰면, 코드를 `validation`하면서  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">개인화된 코드 컨벤션을 녹여낼 수 있다.</span>  

```
npm i -D eslint@8.57.0 eslint-plugin-prettier prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser

-- @: npm's organization
```

`.eslintrc.js`  
  : eslint 설정파일, <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">프로젝트 폴더 바로 아래에 작성</span>  
  `.` : 숨김파일  
  `rc` : 설정파일  

```js
module.exports = {
    root: true,
    env: {
        browser: true,
        node: true
    },
    extends: [
        'eslint:recommended',
        'plugin:@typescript-eslint/eslint-recommended',
        'plugin:@typescript-eslint/recommended',
    ],
    plugins: ['prettier', '@typescript-eslint'],
    rules: {
        'prettier/prettier': [ // prettier의 설정의 이해
            'error', // 정의된 규칙에 어긋날 시, error 처리 (다른옵션: warn, off)
            {
                singleQuote: true,
                semi: true,
                useTabs: false,
                tabWidth: 2,
                printWidth: 80,
                bracketSpacing: true,
                arrowParens: 'avoid',
                endOfLine: 'auto',
            },
        ],
    },
    parserOptions: {
        parser: '@typscript-eslint/parser',
    },
};
```

- `eslint:recommended`  
  : eslint에서 권장하는 rules, plugin 옵션들의 집합이 정의되어 있다.  

- `@typescript-eslint/eslint-plugin`, `@typescript-eslint/parser`  
  : eslint로 typscript의 <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">rule을 validation할 때 권장</span>하는 라이브러리  
  ☑️ tslint가 있는데도 eslint로 type checking하는 이유: <span style="background-color:#ffdce0;color:rgb(196,58,26);border-radius:5px;padding:2px 3px;">tslint 성능이슈</span>  

- `rules`  
  : eslint가 <span style='color:rgb(196,58,26);'>어떤 규칙을 가지고 파일을 validation할 지</span> 규칙을 정의하는 속성

`.eslintignore`  
  : estlint로 validation 제외 대상 정의  

```
node_modules
dist
```

`🪄 settings.json in vscode`  
  : vscode 설정 파일, `ctrl + shift + p` `Open settings`  
  `ctrl + ,` `Settings`에서 ui상으로 적용한 옵션들이 해당 파일에 업데이트 된다.  
  ※ `crtl + ,` `format on save` 해제, <span style='color:rgb(196,58,26);'>prettier의 format과 충돌나지 않기 위해서</span>

```json
{
    "editor.codeActionsOnSave": { 
        "source.fixAll.eslint": true,
    },
    "eslint.alwaysShowStatus": true,
    "eslint.workingDirectories": [
      ],
        {"mode": "auto"}
    "eslint.validate": [
        "javascript",
        "typescript"
    ],
}
```

<span style="font-size:1.03rem;">🏴 editor.codeActionsOnSave - source.fixAll.eslint</span>  
  : 코드를 저장할 때 eslint가 돈다

<span style="font-size:1.03rem;">🏴 eslint.workingDirectories - mode: auto</span>
  : eslint code가 있는 디렉토리를 쫓아가서 validation  

---

🍍 babel  
  : js 최신문법이 최대한 많은 브라우저에 호환되는 형태로 변환해주는 `javascript compiler` `(transfiler)`  
  babel은 ts의 <span style='color:rgb(196,58,26);'>타입 시스템을 이해할 수 없기 때문에</span> 관련 라이브러리를 설치해야 한다.  

  ```
  npm i -D @babel/core @babel/preset-env @babel/preset-typescript
  ```

`preset` : `plugin + option`의 조합  


## 타입스크립트를 점진적으로 적용해 나가기

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 타입스크립트를 점진적으로 적용해 나가기
</div>

```
(1) 타입스크립트 환경 구성
(2) 명시적인 any 선언
(3) 구체적인 타입 정의
(4) 외부 라이브러리 모듈화
(5) 'strict' 옵션 추가 후 타입 정의
```

### 타입스크립트 환경 구성

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;font-weight:bold;">
    🥯 타입스크립트 환경 구성
</div>

`typescript` 라이브러리를 설치하고 관리하기 위해 우선, <span style='color:rgb(196,58,26);'>npm 초기화</span>부터 수행하여,  
`🧆 package.json` 파일을 생성한다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">※ package.json에 커스텀 명령어 "build": "tsc" 지정</span>  

🍍 package.json  
  : nodejs에서 사용하는 모듈들을 패키지로 만들어 관리하고 배포하는 역할을 한다.  

`typescript` 라이브러리를 개발용으로 설치한다.

```
npm init -y
npm i -D typescript
```

타입스크립트 설정 파일 `🎳 tsconfig.json` 을 생성한다.  

```json
{
    "compilerOptions": {
        "allowJs": true,
        "target": "ES5",
        "outDir": "./dist",
        "moduleResolution": "node",
        "lib": ["ES2015", "DOM", "DOM.Iterable"]
    },
    "include": ["./src/**/*"]
}

```

🏴 `"allowJs": true`  
  : js까지 ts가 검증  
  : ts를 점진적으로 적용하기 위해 js까지 컴파일한다.  

🏴 `"outDir": "./dist"`  
  : ts의 결과물이 어디에 들어갈 것인지를 지정

🏴 `"moduleResolution": "node"`  
  : Promise를 인식시켜 주기 위해 설정

🏴 `"include": ["./src/**/*"]`  
  : 어떤 파일을 대상으로 ts파일을 컴파일 할 것인지 지정  
  `["**/*"], default` 모든 폴더 밑에 있는 모든 파일을 대상으로 

🏴 `"exclude": ["node_modules", "bower_components", "json_packages"]`  
  : 어떤 파일을 ts 컴파일에서 제외 할 것인지 지정, `default`  

해당 환경을 구성한 뒤에,  
js파일을 하나씩 ts파일로 변환하고 빌드해본다.  

```
npm run build
```

타입에러가 났음에도 js파일이 생성된다.  
즉, <span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">타입에러와 런타임에러는 서로 독립적인 관계이다.</span>  

### 명시적인 any 선언

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;font-weight:bold;">
    🥯 명시적인 any 선언
</div>

`🎳 tsconfig.json` 파일에 `noImplicitAny: true` 옵션을 추가한다.  
우선 에러를 해결한다는 관점에서만 에러가 난 부분에 명시적으로 any타입을 정의해준다.  

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">※ 타입을 정하기 어려운 곳이 있으면 명시적으로 any롤 선언한다.</span>

### 외부 라이브러리 모듈화 axios, chart.js

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;font-weight:bold;">
    🥯 외부 라이브러리 모듈화 axios, chart.js
</div>

🍍 axios  
  : Promise based HTTP client for the browser and node.js  
  : `npm i axios`  

<span style="font-size:1.03rem;">🍪 타입스크립트가 외부 라이브러리(모듈)를 해석하는 방식</span>  
  : js 라이브러리를 ts에 이식하려면 ts가 인식할 수 있게 중간에 타입을 정의해줘야 한다.  
  즉, 타입 정의 파일 `(index.d.ts)` 이 있어야 한다. <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">d: declaration type, 선언파일</span>  

  : <span style='color:#5F5F5F;'>(방법1)</span> 라이브러리 내부 자체에 있는 경우, 해당 폴더 하위의 `index.d.ts` 파일 참조  
  <span style='color:#5F5F5F;'>(방법2)</span> 타입 정의 라이브러리가 따로 있는 경우, 해당 라이브러리를 설치 in `Definitely Typed`  
    : `node_modules` 아래 `@types` 하위 폴더에 정의된 `index.d.ts` 파일 참조  

  : <span style='color:#5F5F5F;'>(방법3)</span> 타입 정의 라이브러리가 제공되지 않는 경우, `index.d.ts` 파일 직접 정의  
    : `🎳 tsconfig.json`에 `"typeRoot": ["./node_modules/@types", "./types"]` 지정  
    `./types` 하위에 라이브러리명으로 폴더를 만들고 `index.d.ts`파일 정의 `ex. ./types/chart.js/index.d.ts`  

    ```ts
    declare module 'chart.js';
    ```

    <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">※ 옛 버전의 chart.js에는 라이브러리 내부 자체 index.d.ts 파일이 없었다.</span>  

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;color:#34343c;">
<span style="font-weight:bold;">📘 Definitely Typed</span><br>
The repository for high quality<br>
Typescript type definitions.<br>
외부 라이브러리에 대해 이미 만들어논 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">오픈소스 타입 정의 라이브러리 저장소</span><br><br>

라이브러리 내부 자체에 타입정의 파일 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">index.d.ts</span>이 없는 경우<br>
Definitely Typed 저장소의 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">@types</span>에서 관련 타입정의 라이브러리를 찾아 설치한다.<br>
<span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">@types</span> : Definitely Typed 저장소 하위에 정의된 타이핑(typing) 라이브러리
</div>

### strict 옵션 추가 후 타입 정의

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;font-weight:bold;">
    🥯 strict 옵션 추가 후 타입 정의
</div>

`🎳 tsconfig.json` 파일에 `"strict": true` 옵션 추가  
좀 더 엄격하게 타입을 정의할 수 있게 동작을 점검한다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 추후에 일어날 수 있는 타입정의에 대한 오류를 막을 수 있다.</span>  

해당 옵션을 추가하면 아래의 옵션들을 추가한 효과가 생긴다.  

```
"alwaysStrict": true
"strictNullChecks": true
"strictBindCallApply": true
"strictFunctionTypes": true
"strictPropertyInitialization": true
"noImplicitThis": true
```

- `strictNullChecks`  
  : 🍪 해당 에러를 해결하는 4가지 방법  
    : - (1) `type guard`  

       ```ts
       const div = document.querySelector('div');
       funciton clearDiv() {
           if(!div) {
                return;
           }
           div.innerHTML = '';
       }
       ```

    : - `type assertions`  
      : ⚠️ 확신이 있을 때만 사용해야한다. 타입에러가 나지 않고, 객체의 경우 필요한 속성을 누락할 수 있다.  
      (2) `as`  
      (3) `non-null type assertion`: !  
      (4) `optional chaining operator`: ? <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">권고</span>  
      

      ```ts
      // (1)
      const div = document.querySelector('div') as HTMLDivElement;

      // (2) !: null이 아니다
      div!.innerHTML ='';

      // (3)
      div?.innerHTML = '';
      // 내부적으로 해당 동작과 같다
      // if (div === null || div === undefined) {
      //   return;
      // } else {
      //   div.innerHTML = '';
      // }
      ```
      <div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;color:#34343c;">
      <span style="font-weight:bold;">📕 타입스크립트 내부적으로 정의된 타입 간의 상하 관계</span><br>
      <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">HTMLDivElement</span> extends <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">HTMLElement</span> extends <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">Element</span><br>
      <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">MouseEvent</span> extends <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">UIEvent</span> extends <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">Event</span><br>
      </div>


- `strictFunctionTypes`  
  : <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 No overload matches this call.</span>  
  정의된 함수의 스펙과 넘겨받은 함수의 스펙이 일치하지 않을 때 해당 에러 발생

## 예제: DOM API 관련 유틸리티 함수 선언

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 예제: DOM API 관련 유틸리티 함수 선언
</div>

```ts
function getUnixTimestamp(date: string | Date) {
  return new Date(date).getTime();
}

funtion $<T extends HTMLElement = HTMLDiveElement>(selector: string) {
  const element = document.querySelector(selector);
  return element as T;
}
// Generic으로 타입까지 넘겨받으면 해당 함수 안에서 type assertion까지 할 수 있다.
// 넘겨받는 타입이 없을 경우를 대비하여, default타입을 위와 같이 정의할 수 있다.

const canvas = $<HTMLCanvasElement>('.canvasEle');
```

## Utility Type

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 기존에 정의한 타입을 변환할 때 사용할 수 있는 타입 문법이다
</div>

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 유틸리티 타입을 이용하면 불필요하게 중복되는 타입 코드들을 줄여나갈 수 있다.</span>  

- `Partial`  
  : 특정 타입의 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">부분집합을 만족하는 타입을 정의할 수 있다.</span>  

  ```ts
  interface Address {
    email: string;
    address: string;
  }

  type MayHaveEmail = Partial<Address>;
  const me: MayHaveEmail = {}; // (ㅇ)
  const you: MayHaveEmail = { email: 'abc@abc.com' }; // (ㅇ)
  const all: MayHaveEmail = { email: 'peter@abc.com', address: 'Seoul' }; // (ㅇ)
  ```

- `Pick`  
  : 특정 타입에서 지정된 속성만 골라 타입을 정의  

  ```ts
  interface Product {
    id: number;
    name: string;
    price: number;
    brand: string;
    stock: number;
  }

  interface ProductOutline {
    id: number;
    name: string;
    price: number;
  }

  type ShoppingItem = Pick<Produce, 'id' | 'name' | 'price'>;
  ```

- `Omit`  
  : 특정 타입에서 지정된 속성만 제거한 타입을 정의  

  ```ts
  interface AddressBook {
    name: string;
    phone: number;
    address: string;
    company: string;
  }

  const chingtao: Omit<AddressBook, 'address' | 'company'> = {
    name: '중국집',
    phone: '1234123412'
  }
  ```

## Mapped Type

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 Mapped Type
</div>

기존에 정의된 타입을 변환할 때 사용하는 문법인데,  
js의 `map API` 함수를 타입에 적용한 것과 같은 효과를 가진다.  

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 내부 구현체로 많이 활용된다.</span>  

<span style="font-weight:700;font-size:1.03rem;">🍪 Mapped Type 기본 문법</span>  

```ts
{ [ P in K ]: T }
{ [ P in K ]?: T }
{ readonly [ P in K ]: T}
{ readonly [ P in K ]?: T }
```

```ts
type Person = 'Peter' | 'Jack' | 'Lami';
type PersonAges = { [K in Person]: number }
// 기존에 정의된 타입을 Mapped Type 문법을 이용해 새로운 타입으로 변환(?)

// 아래와 동일한 타입 코드이다
// type PersonAges = {
//   Peter: number;
//   Jack: number;
//   Lami: number;
// }

const participantsAges: PersonAges = {
  Peter: 30,
  Jack: 45,
  Lami: 27
}
```

## Utility Type 내부동작 구현

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🌋 Mapped Type으로 구현된 Partial 유틸리티 함수는 다음과 같이 정의된다
</div>

```ts
interface UserProfile {
  username: string;
  email: string;
  profilePhotoUrl: string;
}

type UserProfileUpdate = Particial<UserProfile>;
// 아래와 동일한 타입코드이다.
// type UserProfileUpdate = {
//   username?: string;
//   email?: string;
//   profilePhotoUrl?: string;
// }

// #1
type UserProfileUpdate = {
  username?: UserProfile['username'];
  email?: UserProfile['email'];
  profilePhotoUrl?: UserProfile['profilePhotoUrl'];
}

// #2, Mapped Type 문법 이용
type UserProfileUpdate = {
  [p in 'username' | 'email' | 'profilePhotoUrl']?: UserProfile[p]
}

// #3, keyof 문법 이용
type UserProfileUpdate = {
  [p in keyof UserProfile]?: UserProfile[p]
}

// #3, 기존에 정의된 다른 타입을 넘겨받을 수 있게, Generic 이용
type Subset<T> = {
  [p in keyof T]?: T[p]
}

// Partial은 내부적으로 이렇게 정의되어 있다.
```