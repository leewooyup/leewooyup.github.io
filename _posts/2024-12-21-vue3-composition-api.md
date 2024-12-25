---
title: "[vue3] 로직의 재사용이 유용한 Composition API와 hook을 통한 관심사의 분리"
date: 2024-12-25 17:49 +0900
lastmod: 2024-12-25 17:49 +0900
categories: vue3
tags:
  [
    Composition API,
    module,
    setup,
    props,
    context,
    ref,
    reactivity,
    computed,
    lifecycle,
    watch,
    hook
  ]
---

## Vue2와 Vue3의 차이점

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 Vue2와 Vue3의 차이점
</div>

- `Composition API`  
`Composition API`는 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">Vue2</span>에서는 플러그인 형태로 사용가능했지만,  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">Vue3</span>부터 라이브러리 공식 API로 채택되었다.  

- `root element`  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">Vue2</span>에서는 `root element`가 하나여야 되는 한계점이 있지만,  
<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">Vue3</span>에서는 `root element`가 여러개여도 상관없다.  

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">Vue3</span>로 넘어가면서 라이브러리의 전반적인 로직을 타입스크립트로 재작성했다.  

## Composition API

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 Functional-based Composition API
</div>

`Option API`에서 나누어 작성했던 속성을 `Composition API`는 setup 속성에 한번에 작성한다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">Option API와 섞어쓸 수 있다..</span>  
즉, setup 안에서 변수와 함수를 선언해서 사용할 수 있게 만들었는데, <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">이러한 구조적인 이점은 타입추론을 용이하게 한다.</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">※ 함수나 변수를 정의할 때 타입을 추론하기 쉽다.</span>  

또한, `Composition API`를 이용하면 로직을 재사용할 수 있다.  

`useMessage.js`  

```js
import { ref } from "vue";

function useMessage() {
    const message = ref('hello');

    function changeMessage() {
        message.value = 'hi';
    }

    return { message, changeMessage } // setup 바깥에서 사용하기 위해 return
}

export { useMessage }
```

`🦇 App.vue`  

```vue
<template>
    <div>
        <p>{{ message }}</p>
        <button @click="changeMessage">change</button>
    </div>
</template>

<script>
import { useMessage } from './hooks/useMessage';

export default {
    setup() {
        const { message, changeMessage } = useMessage();
        return { message, changeMessage }; // setup 바깥에서 사용하기 위해 return
    }
}
</script>

```

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;">
<span style="font-weight:bold;">☑️ setup의 인자</span><br>
setup의 첫번째 인자는 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">props</span>이고, 두번째 인자는 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">context</span>이다.<br>
단, <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">props</span>의 속성에 접근할 때, Destructuring을 이용하면 안된다.<br>
Destructuring을 이용하는 순간 Reactivity가 사라진다. 이러면, 데이터 싱크를 맞추기 어려워지고 버그로 이어질 수 있다.<br><br>

<span style='color:rgb(196,58,26);'>Destructuring the `props` will cause the value to lose reactivity</span><br><br>

두번째 인자인 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">context</span>에는 emit (이벤트 발생)과 같은 함수가 들어있는데, 이는 호출하는 용도이므로, Destructuring 해도 상관없다.<br>
⚠️ <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">Composition API</span>에서는 this를 쓰지 않는다.
</div>

## ref API와 .value

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 ref API와 .value
</div>

- `ref` <span style="background-color:#E1FEE5;color:#345F53;border-radius:5px;padding:2px 3px;">vue3</span>  
  : 🗣️ 'Reactivity를 주입하는 변수를 선언하겠다.'  

- `.value`  
  : 해당 속성을 통해, <span style='color:rgb(196,58,26);'>변수의 실질적인 값을 제어</span>한다.

<span style="font-weight:700;font-size:1.03rem;">🍪 왜 .value를 통해 실질적인 값에 접근하게 만든걸까?</span>  
  : <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">vue는 자동으로 데이터의 변경사항을 감지</span>하고, 그에 따라 DOM을 업데이트한다.  
  즉, vue는 렌더링 중에 사용된 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">모든 참조를 추적한다.</span>  
  참조가 변경되면, <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">참조를 추적하는 구성요소에 대한 리렌더링을 트리거</span>한다.  
  &nbsp;  
  표준 js에서는 일반 변수의 접근이나 변경을 감지할 방법이 없다.  
  하지만, `getter`/`setter` 메서드를 이용하면 접근과 변경을 감지할 수 있는데,  
  여기서 `.value` 속성은 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">vue가 참조에 접근하거나 변경한 시점을 감지할 수 있는 기회를 제공한다.</span>  
  ref로 변수를 초기화 했을 때, 즉 `Reactivity`가 주입된 값에 대해서  
  `.value`를 통해 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">값에 접근하거나, 값이 변경됨에 따라 특정 동작을 수행할 수 있는 구조를 만들어 놓았다.</span>  

  ```js
  const myRef = {
    _value: 0,
    get value() {
        track()
        return this._value
    }

    set value(newValue) {
        this._value = newValue
        trigger()
    }
  }
  ```

하지만, 기존 vue2에서의 개발경험을 해치지 않기 위해 setup 바깥에서는 .value없이 접근할 수 있게 고안되었다.

## Composition Style Based API, vue3

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 vue 라이브러리에서 제공하는 API를 가져와서 호출하여 Option API의 속성을 구현한다
</div>

- `computed API`  
  : 특정 연산을 담아두는 속성으로, <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">값을 원하는 형태로 변환하는 익명함수</span>를 작성한다.  

  ```vue
  <template>
    <h1>{{ appTitle }}</h1>
    <h4>{{ newTitle }}</h4>
  </template>

  <script>
  import { computed } from 'vue';

  export default {
    props: ['appTitle'],
    setup(props) {
        const newTitle = computed(() => {
            return props.appTitle + '!!';
        })

        return { newTitle }
    }
  }
  </script>
  ```

---

- `Lifecycle API`  
  : 라이프사이클 훅 제공  
  <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">vue 인스턴스 라이프사이클</span>이란 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">인스턴스가 생성되어 소멸되기까지 거치는 과정을 의미한다.</span>  
  인스턴스가 생성되면 라이브러리 내부적으로 다음 과정이 진행된다.  

  ```
  - data속성의 초기화 및 관찰(Reactivity 주입)
  - vue 템플릿 코드(템플릿 표현식) 컴파일 (virtual DOM -> DOM 변환)
  - 인스턴스를 DOM에 부착
  ```

  ![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/mrm0QyCI9u.png "Optional title"){: width="800" class="normal"}  

  <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">※ 컴포넌트 생성 직후 바로 setup이 호출된다, 이 때 데이터가 동기화된다.</span>  

  🌩️ 네이밍 컨벤션 변경  

  | vue2 | vue3 |
  | --------- | ------------------------------------ |
  | beforeCreate | x, 대신 setup 사용 |
  | created | x, 대신 setup 사용 |
  | beforeMount | onBeforeMount |
  | mounted | onMounted |
  | beforeUpdate | onBeforeUpdate |
  | updated | onUpdated |
  | beforeDestroy | onBeforeUnmount |
  | destroyed | onUnmounted |
  | errorCaptured | onErrorCaptured |

---

- `watch API`  
  : 특정 데이터가 변했을 때, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">해당 변화를 감지해서 부가적인 동작을 호출</span>한다.  
  ※ watch는 최대한 지양, 개발자가 데이터 추적이 어려워진다.  

  ```js
  import { ref, watch } from 'vue';

  watch(message, (newValue, oldValue) => {
    console.log({newValue});
  })
  ```

## Custom Composition Function (hook)과 관심사의 분리

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 Composition API의 강점은 컴포넌트에 남아있어야 할 로직과 그러지 않은 로직을 별도의 파일로 나눌 수 있다는 점이다
</div>

<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">컴포넌트의 동작을 알아차릴 수 있는 로직</span>만 남겨 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">동작을 명시적으로 추상화</span>시키는 것이 좋다.  
별도의 파일로 분리된 세부적인 로직은, 컴포넌트에 남아있는 로직에서 변수 및 함수명으로 유추할 수 있게 네이밍한다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">단, 별도의 파일로 분리되었을 때 장점이 명확한 경우에만 분리한다.</span>  

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🎯 setup 당위성</span>  
  : 많은 컴포넌트에서 <span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">동일한 코드가 반복되는 경우</span>  
  하나의 파일에서 해당 컴포넌트들에 뿌려질 수 있는 형태로 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">모듈화</span>, <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">관심사의 분리</span>  

`🐝 useTodo.js`  

```js
import { ref } from "vue";

function useTodo() {
  const todoItems = ref([]);

  function fetchTodos() {
    // 의존성이 생기지 않게 해당 함수 내에서만 사용하는 변수 선언 (의존성 제거)
    // 함수 내의 동작이 외부 코드와 연결될 때 '의존성이 생긴다' 라고 표현한다.
    const result = [];
    for(let i = 0; i < localStorage.length; i++) {
      const todoItem = localStorage.key(i);
      result.push(todoItem);
    }
    return result;
  }

  function addTodoItem(todo) {
    todoItems.value.push(todo);
    localStorage.setItem(todo, todo);
  }

  return { todoItems, fetchTodos, addTodoItem }
}

export { useTodo }
```

`🦇 App.vue`  

```js
<script>
  import { useTodo } from './hooks/useTodo';

  export default {
    setup() {
      const { todoItems, fetchTodos, addTodoItem } = useTodo();

      /*...생략 */
    }
  }
</script>
```