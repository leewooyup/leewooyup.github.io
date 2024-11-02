---
title: "[Issue__vuex] 새로고침 후에 Vuex store의 state가 초기화되는 현상 해결"
date: 2024-11-02 14:06 +0900
lastmod: 2024-11-02 14:06 +0900
categories: Issue
tags:
  [
    Vuex,
    vuex-persistedstate,
    state,
    localStorage,
    modules,
    Vue Instance Property,
    Helper Function
  ]
---

## Issue: vuex state 초기화

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐂 Issue: 새로고침 후에 Vuex 상태 유지 x
</div>

<span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">🧾 로그인 후, 관련 데이터를 vuex의 state에 저장하려는 상황</span>  
로그인 후에 얻은 `🪙 accessToken`을 `vuex store` `state`에 저장하고 페이지 접근 권한을 따질 때 사용하려 했지만,  
새로고침이나 페이지 이동 시, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 state 값이 초기화되는 문제에 직면</span>

🪶 새로고침을 하면 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">vue 인스턴스가 소멸했다가 다시 생성되는 vue의 라이프사이클</span> 때문에 발생

## resolve: vuex-persistedstate 라이브러리

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐃 resolve: vuex-persistedstate
</div>

`npm install --save vuex-persistedstate`

`vuex store`의 `state`에 저장된 값을 웹브라우저의 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🥟 localStorage</span>에 저장한다.  
이렇게 하면 새로고침을 해도 사라지지 않기 때문에, 로딩 시 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🥟 localStorage</span>에 있는 값을 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">state에 다시 동기화 시켜준다.</span>

다만, `vuex store`에 있는 모든 `state`값을 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">🥟 localStorage</span>에 저장하게 되면,  
성능이 크게 떨어질 수 있다. 따라서, `vuex store`를 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">모듈화하여 localStorage에 저장할 모듈만 path에 등록시키면 해당 성능문제의 영향을 줄일 수 있다.</span>

---

`🐝 store.js`  
 : vuex store 모듈화

```js
import { createStore } from "vuex";
import createPersistedState from "vuex-persistedstate";

const store = createStore({
  modules: {
    user: userStore,
    product: productStore
  },
  plugins: [
    createPersistedState({
      paths: ["user"]
    })
  ]
});

export default store;
```

`modules` 이라는 옵션을 통해 모듈을 등록하는데, 등록되는 방식을 보면  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">각 모듈이 단순한 객체 형태여야 한다.</span>

`🐝 userStore.js`  
 : vuex store의 각 모듈, 단순한 객체 형태

```
const userStore = {
    namespaced: true,
    state: () => ({
        accessToken: '',
    }),
    mutations: {
        setAccessToken(state, accessToken) {
            state.accessToken = accessToken;
        },
    },
    actions: {
        login({ commit }, accessToken) {
            commit('setAccessToken', accessToken);
        },
    },
    getters: {
        isAuthenticated(state) {
            return !!state.accessToken;
        }
    }
};

export default userStore
```

**`createStore`**는 애플리케이션 전체의 <span style="padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">중앙 스토어 인스턴스를 한번만 생성할 때 사용하는 함수</span>이고,  
vuex에서 모듈을 정의할 때는 각 모듈이 `state`, `mutations`, `actions`, `getters` 속성을 가지는 단순한 객체로 정의한다.

<span style="font-weight:700;font-size:1.03rem;">🍪 state를 화살표 함수로 작성한 이유는 모듈의 state가 독립적인 인스턴스로 유지되도록 하기 위해서이다.</span>  
 : state를 객체로 작성하면 해당 모듈이 여러 인스턴스로 사용될 때, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 모든 인스턴스가 동일한 객체를 공유하게 된다.</span>  
 하지만 화살표 함수로 작성하면 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">각 인스턴스가 함수 호출로 새로운 객체를 생성하게 되어 상태 공유를 방지할 수 있다.</span>

---

`🐝 main.js`

```js
import { createApp } from "vue";
import App from "./App.vue";
import store from "./store/store";

const app = createApp(App);
app.use(store);
app.mount("#app");
```

이렇게 설정해 놓으면 새로고침이나 페이지 이동에도 `vuex store`의 `state`가 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">초기화되지 않는다.</span>

## 모듈화된 store 호출법

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 모듈화된 store 호출법
</div>

- `this.$store` 호출법  
   : `$`: <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">vue instance property</span>를 나타내는 관례

  ```js
  this.$store.dispatch("MODULE_NAME/ACTIONS_NAME", ARGS);
  ```

- <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">import 식별자</span> 호출법  
  : `import store from './store/store'`

  ```js
  store.getters["MODULE_NAME/GETTERS_NAME"];
  ```

- <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">헬퍼함수</span> 호출법  
  : 🍍 vuex의 각 속성들을 더 쉽게 사용할 수 있게 하는 함수

  ```js
  ...mapState('MODULE_NAME', ['STATE_NAME']);
  ```
