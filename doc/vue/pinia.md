# Pinia

> Pinia 是 Vue 的存储库，它允许您跨组件/页面共享状态。

## 安装

```powershell
npm install pinia
```

## main.ts 中

```typescript
import { createApp } from "vue";
import App from "./App.vue";
import { createPinia } from "pinia";

const app = createApp(App);

app.use(createPinia());
app.mount("#app");
```

## store/index

```typescript
import { defineStore } from "pinia";

export const useStore = defineStore("main", {
  state: () => {
    return {
      name: "yuchunyi",
      age: 18
    };
  },
  getters: {
    ageDouble(state) {
      return state.age * 2;
    },
    doubleCountPlusOne() {
      // 可以通过this访问到useStore
      return this.doubleCount + 1;
    }
  },
  actions: {
    increment() {
      this.age++;
    },
    //异步操作
    async registerUser(login, password) {
      const res = await login(login, password);
      return res.data;
    }
  }
});
```

## state

```vue
<template>
  <div>{{ store.name }}</div>
   
  <div>{{ store.age }}</div>
   
  <div>{{ name }}</div>
   
  <div>{{ age }}</div>
</template>

<script lang="ts" setup>
import { useStore } from "@/store/index";
import { storeToRefs } from "pinia";
// 普通使用，需要store.name、store.age
const store = useStore();
// 使用解构，直接用name age，如果不用storeToRefs，就没响应式
const { name, age } = storeToRefs(useStore());

onMounted(() => {
  store.increment();
});
</script>
```

## 修改 state

- 方法 1 直接通过 store.xxx 修改

```typescript
const handleClick = () => {
  store.name = "lili";
  store.age++;
};
```

- 方法 2 $patch 对象写法

```typescript
const handleClickPatch = () => {
  store.$patch({
    name:"lili"
    age: store.age ++;
  });
};
```

- 方法 3 $patch 函数写法

```typescript
const handleClickMethod = () => {
  store.$patch((state) => {
    state.age++;
    state.name = "lili";
  });
};
```

- 方法 4 在 store 的 action 里写好了再调用
  src\store\index.ts 文件里，在 actions 里写一个 changeState( )方法，用来改变数据状态
  在用 actions 的时候，不能使用箭头函数，因为箭头函数绑定是外部的 this

```typescript
actions:{
    changeState(){
        this.age++
        this.name='lili'
    }
}
```

调用

```typescript

//在 template
<button @click="store.changeState">按钮</button>

//在 script
store.changeState()

```

## getters

与计算属性类似，有缓存效果，依赖项的变化重新触发方法
getters 里面的方法相对于 state 里面变量

- src\store\index.ts 文件里的 getters

```typescript
  getters: {
    getAge(state) {
      return state.age + '岁'
    }
  }
```

- 使用

```html
 
<div>{{ store.getAge }}</div>

<!-- const { getAge } = storeToRefs(useStore());   -->
<div>{{ getAge }}</div>
```

## actions

- src\store\index.ts 文件里的 actions

```typescript
  actions: {
    increment() {
      this.age++;
    },
    //异步操作
    async registerUser(login, password) {
      const res = await login(login, password);
      return res.data;
    }
  }
```

- 使用

```javascript
import { useStore } from "@/store/index";

const store = useStore();

onMounted(() => {
  store.increment();
  const data = store.registerUser();
});
```

## 问题

pinia 在外部 ts 文件中使用时。要创建 pinia 实例，然后传入 useStore 方法中，否则会报错。

```typescript
import { createPinia } from "pinia";
import { useMainStore } from "@/store/index";

const store = useMainStore(createPinia());
```
