---
title: "[Dev-core-frontend] 팀단위 코드 컨벤션 설정을 위한 eslint, prettier (with babel)"
date: 2025-01-01 23:40 +0900
lastmod: 2025-01-25 19:52 +0900
categories: Dev-core-frontend
tags:
  [
    eslint,
    prettier,
    typescript,
    settings.json,
    .eslintrc.js,
    .prettierrc.js,
    babel
  ]
---

## 프로젝트 환경 구성 (eslint + prettier, babel)

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 ESLint는 Linter이고, Prettier는 Formatter이다
</div>

`Linter`는 js 소스코드에 직접적인 에러와 에러가 날만한 잠재적인 부분에 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">flag를 달아주는 코드 문법 보조 도구</span>이다.  
이러한, `🏴 ESLint`는 <span style='color:rgb(196,58,26);'>직/간접적인 에러를 체킹</span>해줄 뿐만 아니라, 코드 자동완성과 포맷팅까지 할 수 있다.  
포맷팅을 해주는 `code formatter`란 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">소스코드를 일관된 스타일로 작성하도록 코드를 변환해주는 도구</span>이다.

하지만, `ESLint`가 `code formatter`가 있음에도 불구하고, 별도의 `code formatter`인 `prettier`를 사용하는데,  
이는 `prettier`가 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">개인화된 포맷팅을 지정할 수 있기 때문이다.</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎡 prettier: an opinionated code formatter</span>

🎡 prettier  
 : 팀단위 코딩 컨벤션을 만들어 코드를 정의할 때 자주 사용된다. <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">가독성🏆</span>  
 즉, `eslint`를 `prettier`와 함께 쓰면, 코드를 `validation`하면서  
 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">개인화된 코드 컨벤션을 녹여낼 수 있다.</span>

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">ESLint + prettier 설정</span> `🧆 package.json`

```json
devdependencies: {
  "eslint": "^8.57.1",
  "eslint-config-prettier": "^8.10.0",
  "eslint-plugin-prettier": "^5.0.0",
  "prettier": "^3.4.2"
}
```

- `🎡 eslint-config-prettier`: `Prettier`와 충돌하는 `ESLint` 규칙을 꺼준다.
- `🎡 eslint-plugin-prettier`: `Prettier`를 `ESLint` 규칙으로 실행시켜준다.

```
npm i -D eslint@8.57.0 eslint-plugin-prettier prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser

-- @: npm's organization
```

`🏴 .eslintrc.js`  
 : ESLint 설정파일, <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">프로젝트 폴더 바로 아래에 작성</span>  
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
    // 'plugin:vue/vue3-essential', // in vue 프로젝트
    "eslint:recommended",
    // '@vue/typescript', // in vue 프로젝트
    "plugin:@typescript-eslint/eslint-plugin",
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended"
  ],
  plugins: ["@typescript-eslint", "prettier"],
  // parser: 'vue-eslint-parser', // in vue 프로젝트
  parserOptions: { // 선택한 parser에 전달할 세부설정을 정의
    // 특별한 구문을 분석하려면 적합한 parser(구문분석기) 설정, (for ts코드 분석)
    parser: "@typscript-eslint/parser",
    ecmaVersion: 2020, // 최신 ecmascript 사용 가능
    sourceType: 'module', // es module 지원
    project: './tsconfig.json',
  }
  rules: {
    "prettier/prettier": [
      // prettier의 설정의 이해
      "error", // 정의된 규칙에 어긋날 시, error 처리 (다른옵션: warn, off)
      {
        singleQuote: true,
        semi: true,
        useTabs: false,
        tabWidth: 2,
        printWidth: 80,
        bracketSpacing: true,
        arrowParens: "avoid",
        endOfLine: "auto"
      }
    ]
  },
};
```

- `🎠 eslint:recommended`  
  : eslint에서 권장하는 <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">rules</span>, <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">plugin</span> 옵션들의 집합이 정의되어 있다.

- `🎠 @typescript-eslint/recommended`, `🎠 @typescript-eslint/eslint-plugin`, `🎠 @typescript-eslint/parser`  
  : eslint로 typscript의 <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">rule을 validation할 때 권장</span>하는 라이브러리  
  ☑️ tslint가 있는데도 eslint로 type checking하는 이유: <span style="background-color:#ffdce0;color:rgb(196,58,26);border-radius:5px;padding:2px 3px;">tslint 성능이슈</span>

- `🎠 prettier/recommended`  
  : `eslint`와 `prettier`를 통합하여 코드포맷팅 관리 및 권장 설정 적용

- `rules`  
  : eslint가 <span style='color:rgb(196,58,26);'>어떤 규칙을 가지고 파일을 validation할 지</span> 규칙을 정의하는 속성  
  prettier와 관련된 규칙은 해당 파일 말고, `🎡 .prettierrc.js` 파일에도 정의 가능
  ```js
  module.exports = {
    singleQuote: true,
    semi: true,
    useTabs: false,
    tabWidth: 2,
    printWidth: 80,
    bracketSpacing: true,
    arrowParens: "always",
    endOfLine: "auto"
  };
  ```

`.eslintignore`  
 : estlint로 validation 제외 대상 정의

```
node_modules
dist
```

`🪄 settings.json in vscode`  
 : vscode 설정 파일, `ctrl + shift + p` `Open settings`  
 `ctrl + ,` `Settings`에서 ui상으로 적용한 옵션들이 해당 파일에 업데이트 된다.

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

<span style="font-size:1.03rem;">`🏴 editor.codeActionsOnSave - source.fixAll.eslint`</span>  
 : 코드를 저장할 때 eslint가 돈다

<span style="font-size:1.03rem;">`🏴 eslint.workingDirectories - mode: auto`</span>
: eslint code가 있는 디렉토리를 쫓아가서 validation

---

🍍 babel  
 : js 최신문법이 최대한 많은 브라우저에 호환되는 형태로 변환해주는 `javascript compiler` `(transfiler)`  
 babel은 ts의 <span style='color:rgb(196,58,26);'>타입 시스템을 이해할 수 없기 때문에</span> 관련 라이브러리를 설치해야 한다.

```
npm i -D @babel/core @babel/preset-env @babel/preset-typescript
```

`preset` : `plugin + option`의 조합

## 예제: Vue3 프로젝트, package.json's devDependencies

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 예제: Vue3 프로젝트, package.json's devdependencies
</div>

```json
"devDependencies": {
    "@babel/core": "^7.12.16",
    "@babel/eslint-parser": "^7.12.16",

    "@typescript-eslint/eslint-plugin": "^5.4.0",
    "@typescript-eslint/parser": "^5.4.0",

    "@vue/cli-plugin-babel": "~5.0.0",
    "@vue/cli-plugin-eslint": "^5.0.0",
    "@vue/cli-plugin-typescript": "^5.0.0",
    "@vue/cli-service": "~5.0.0",

    "@vue/eslint-config-prettier": "^7.1.0",
    "@vue/eslint-config-typescript": "^9.1.0",

    "eslint": "^8.57.1",
    "eslint-config-prettier": "^8.10.0",
    "eslint-plugin-prettier": "^5.0.0",
    "eslint-plugin-vue": "^8.0.3",

    "prettier": "^3.4.2",
    "typescript": "^4.5.5"
}
```

- `@babel/eslint-parser`  
  : babel을 사용하는 프로젝트에서 `🏴 ESLint`를 정상적으로 동작하게 해주는 parser  
  즉, 최신 js문법을 `🏴 ESLint`가 이해할 수 있게 만든다.  
  <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">eslint의 정적분석은, babel 변환 이후가 아닌 이전에 시행된다.</span>
- `@typescript-eslint/eslint-plugin`  
  : typescript 전용 규칙을 추가해 코드 스타일 및 문제를 분석한다. <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">감독관</span>
  - `ex. @typescript-eslint/no-explicit-any`
- `@typescript-eslint/parser`
  : typescript 코드를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">AST(Abstract Syntax Tree, 코드의 문법구조를 트리형태로 표현)</span>로 변환하여, `🏴 ESLint`가 이해할 수 있도록 한다. <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">통역사</span>
- `@vue/cli-plugin-typescript`  
  : `🎳 tsconfig.json` 설정파일 생성
- `@vue/eslint-config-prettier`  
  : vue에서 `🏴 ESLint`와 `🎡 Prettier`가 충돌하지 않도록 도와주는 설정  
  `eslint-config-prettier`의 규칙들을 포함한 미리 정의된 설정  
  <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">(즉, @vue/eslint-config-prettier를 설치할 때 <span style='color:rgb(196,58,26);'>자동으로 의존성 설치</span>)</span>
- `@vue/eslint-config-typescript`  
  : vue에서 typescript와 관련된 ESLint 규칙을 제공한다.  
  `@typescript-eslint/eslint-plugin`의 규칙들을 포함한 미리 정의된 설정 (vue와 ts간의 코드 스타일 충돌방지 규칙 설정)
  <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">(즉, @vue/eslint-config-typescript를 설치할 때 <span style='color:rgb(196,58,26);'>자동으로 의존성 설치</span>)</span>
- `eslint-plugin-vue`  
  : `🏴 ESLint`를 사용하여 vue 파일의 코드 품질 관리 및 스타일 유지
