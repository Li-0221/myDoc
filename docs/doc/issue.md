!>持续更新。。。

# vite

## base

vite.config.js 中有一个 base 说用来指定打包路径的，如果部署上服务器出现页面 404，百分之九十是 base 错了

直接看不同的 base 打包出来的结果：

- 默认 base

  <img src='https://s1.ax1x.com/2022/07/22/jOaC40.jpg'>

  <img src='https://s1.ax1x.com/2022/07/22/jOaF3T.jpg'>

- 自定义 base（一般以 / 开头）

  <img src='https://s1.ax1x.com/2022/07/22/jOaiCV.jpg'>

  <img src='https://s1.ax1x.com/2022/07/22/jOa9Nq.jpg'>

## public

public 下的文件不会参与打包，不会生成文件 hash，直接会放到 dist。

可以在 public 中存放一些静态文件，如 JSON 等。

## build

!> 开发环境正常，但是到了生产环境出现了奇怪的 bug，如样式被覆盖。

解决办法：删除 build 下的拆分打包，因为把包拆的太多了，vite 引入包的顺序出问题了，导致样式被覆盖

vite.config.ts

```typescript
export default defineConfig({
  // 生产环境打包配置
  build: {
    chunkSizeWarningLimit: 1000,
    rollupOptions: {
      // 将打包出来的文件，拆分成多个小文件
      output: {
        manualChunks(id) {
          if (id.includes("node_modules")) {
            return id
              .toString()
              .split("node_modules/")[1]
              .split("/")[0]
              .toString();
          }
        }
      }
    }
  }
});
```

# HTTP FormData

FormData 指的是表单数据。

!> 使用 post 请求传递 formData 时，请求体里面放 data 是无效的

1. 可以使用 query 传参。
2. 可以使用以下方式，将数据全部放入 formData

```typescript
const formData = new FormData();
formData.append("scene", "sqlite");
formData.append("file", chunk);
formData.append("index", `${index}`);
formData.append("total", `${total}`);
formData.append("name", name);
formData.append("ext", "db");

formData.forEach((value, key) => {
  console.log(key, value);
});
```

# cookie

使用 js-cookie 存储数据时，其他操作无误的情况下，却没有存进去，大概率是由于 cookie 的大小限制。

# TS

## XX 上不存在属性 xxx

给 `window` 新增 `tinyMce` 属性 ts 报错：类型“Window & typeof globalThis”上不存在属性 ‘tinyMce’

声明 Window ，在根目录下新建一个 `xxx.d.ts` 文件（ts 会自动读取全局的 .d.ts 文件）

```typescript
declare interface Window {
  tinyMce: any;
}
```

## string 不能索引 object

元素隐式具有 “any“ 类型，因为类型为 “string“ 的表达式不能用于索引类型 “Object“。 在类型 “Object“ 上找不到具有类型为 “string“ 的参数的索引签名

解决：key 值的类型不是 string，在 javascript 中是默认给你转好的，而在 Typescript 中则不是，因此要么转，要么声明，要么忽略…

1. 方案一：忽略
   在 tsconfig.json 中 compilerOptions 里面新增忽略的代码，如下所示，添加后则不会报错

```json
"suppressImplicitAnyIndexErrors": true
```

2. 方案二：声明
   在定义的 Interface 里对其进行声明，如下所示，声明过后，也不会再报错

```typescript
interface Myobject {
  [key: string]: boolean; // 字段扩展声明
}
```

# tailwind

## button 样式覆盖

使用 vant 和 element-plus 的 button 组件时，tailwind 的 button 样式会覆盖掉 vant 和 element-plus 的 button 样式
解决办法：

1. 手动引入 button 样式
   main.js

```typescript
// vant
import "vant/es/button/style";

// element-plus
import "element-plus/es/components/button/style/css";
```

2. 移除 vite 的打包拆分配置
   vite.config.ts

```typescript
// 删除以下打包拆分
build: {
    rollupOptions: {
      // 将打包出来的文件，拆分成多个小文件
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            return id.toString().split('node_modules/')[1].split('/')[0].toString()
          }
        }
      }
    }
  }

```

# css

## 去除点击文字的竖线

```css
div {
  -webkit-user-select: none; /*谷歌 /Chrome*/
  -moz-user-select: none; /*火狐/Firefox*/
  -ms-user-select: none; /*IE 10+*/
  user-select: none;
}
```

## 文本两侧对齐

text-align 的属性值常用的有：right，left，center（行内内容居中

justify（向两侧对齐，最后一行无效），justify-all（与 justify 一致，强制最后一行两端对齐）等

# Vant

## Image 图片

### 找不到本地资源

引入本地图片可能找不到资源，解决方法：

1. 使用 vite 的 `new URL`，写相对路径。

```vue
<template>
  <van-image src="url" class="w-full" />
</template>

<script setup lang="ts">
const url = new URL("./assets/image.png", import.meta.url).href;
</script>
```

2. 使用 webpack 的 `require`

```vue
<template>
  <van-image :src="require('./image.png')" />
</template>
```

### 加载提示高度异常

image 提供的加载中提示，宽高是父容器的 100% ，给父容器设置固定宽高

### lazyload

image 的懒加载在 swiper 中存在问题，建议使用 swiper 自带的 [懒加载](https://swiper.com.cn/api/lazy/213.html)，注意看文档最开始的文字。

## 组件注册

在 `<script setup>` 中可以直接使用 Vant 组件，不需要进行组件注册。

官方文档的示例中加了 van ，如`<van-count-down :time="time" />`， 在 setup 中不需要。
有些特殊的组件必须要 van，如 dialog。
具体情况见每个组件的组件调用（setup）部分。

```vue
<template>
  <count-down time="60000" />

  <van-dialog v-model:show="show" title="标题" show-cancel-button>
    <img src="https://fastly.jsdelivr.net/npm/@vant/assets/apple-3.jpeg" />
  <van-dialog>
</template>

<script lang="ts" setup>
import { CountDown，Dialog } from 'vant';
 const VanDialog = Dialog.Component;

const show = ref(true);
</script>
```

# element

## el-form

### resetField 失效

大概率是 el-from 还没挂载，from 上的 data 就赋值了。因为 resetField 是将其值重置为初始值，初始值指的是创建 form 时 data 的值。

解决办法是在 el-from 渲染之后再去赋值，可以使用 nexttick，这种 bug 尤其出现在 el-from 写在 dialog 里面的情况。因为 dialog 挂在页面上时，为了性能，默认是不加载 dialog 里面的内容，只有打开 dialog 之后才加载

## 数据

使用 element plus 时，出现奇怪的问题。可能是数据定义的问题，导致 eleemnt plus 赋值的时候出现了问题 导致失去响应式，尝试切换 ref reactive。

## 样式

覆盖 element 样式时 直接覆盖对应的样式变量，如想让 el-button 的颜色变化。

```scss
.el-button--success.is-disabled {
  --el-button-disabled-bg-color: #fff;
  --el-button-disabled-border-color: #ccc;
}
```

## 树形控件

element Plus 的树形控件支持自定义渲染字段，可以根据层级渲染，可以根据节点数据渲染。

```vue
<template>
  <el-tree
    :data="departmentTree"
    show-checkbox
    ref="treeRef"
    @check-change="handleCheckChange"
    node-key="sid"
    default-expand-all
    :expand-on-click-node="false"
    :props="{ class: customNodeClass, label: customLabel }"
  />
</template>

<script lang="ts" setup>
import type Node from "element-plus/es/components/tree/src/model/node";

// 根据数据：如果当前节点的数据有staffId 表示当前节点应该渲染员工名字，否则渲染部门名字
const customLabel = (data: any) => {
  if (data.staffId) {
    return data.staffName;
  }
  return data.name;
};

// 根据层级：如果当前节点属于第一级，...
const customLabel = (data: any, node: Node) => {
  if (node.level === 1) {
    return "xxx";
  }
};
</script>
```

根据节点数据渲染结果如下（黑色是部门，绿色是员工）：

<img src='https://s1.ax1x.com/2022/10/13/xa0XZT.png'>

## 选择框

!> 使用 el-select 的时候，如果 value 要为一个对象，必须要设置 value-key，值为 value 对象里面的某个属性

```vue
<template>
  <!-- value用reactive定义好像有问题？ -->
  <el-select v-model="value" value-key="id">
    <el-option
      v-for="item in options"
      :key="item.id"
      :label="item.label"
      :value="item"
    />
  </el-select>
</template>
```

## 表单校验

### 基本

- 需要定义 rules 表单校验规则
- 每个 `el-form-item` 上要写 prop，对应得是 rules 对象内的规则
- 触发校验 input 一般用 blur 选择框用 change

<img src='https://s1.ax1x.com/2022/07/04/jYAo6A.png'>

### 自定义校验

- 在自定义的校验规则里面，通过校验必须要使用 `callback()` 表示校验通过，否则校验不会出结果

<img src='https://s1.ax1x.com/2022/07/04/jYATOI.png'>

# VueUse

## 暗黑模式失效

- 场景：使用暗黑模式正常情况下点击按钮，html 标签上会有 class 为 dark 的切换
- 问题：点击按钮只能切换一次
- 原因：`@click="toggleDark` 没有加 `()`，不加括号时该方法的第一个参数是 `event`而不是 `isDark`

```vue
<template>
  <!-- toggleDark() -->
  <button @click="toggleDark">
    {{ isDark ? "Dark" : "Light" }}
  </button>
</template>

<script setup lang="ts">
import { useDark, useToggle } from "@vueuse/core";

const isDark = useDark();
const toggleDark = useToggle(isDark);
</script>
```

# TS & axios

在使用 TS 时，将响应的 res 赋值给变量时报错：

[![XrBsje.png](https://s1.ax1x.com/2022/06/08/XrBsje.png)](https://imgtu.com/i/XrBsje)

[![XrB6nH.png](https://s1.ax1x.com/2022/06/08/XrB6nH.png)](https://imgtu.com/i/XrB6nH)

原因是 `AxiosResponse` 上并没有自己规定返回的一些字段，所以 ts 会报错，所以我们要定义一下类型

## 方法 1（推荐）：

详见 axios ts 封装

- [ts&axios](http://www.lidoc.top/Http/axios&ts)

- 如果链接打不开，见导航栏 HTTP 下的 axios&ts

## 方法 2：

在 src 目录下创建一个 compiler-vue.d.ts 文件

```typescript
// compiler-vue.d.ts

import { AxiosRequestConfig } from "axios";

declare module "axios" {
  interface AxiosInstance {
    (config: AxiosRequestConfig): Promise<any>;
  }
}
```

# vue-router

## next()

- `next()` : 放行
- `next('/login')` : 中断当前路由去 login

> 在 addRoutes()之后第一次访问被添加的路由会白屏，这是因为刚刚 addRoutes()就立刻访问被添加的路由，然而此时 addRoutes()没有执行结束，因而找不到刚刚被添加的路由导致白屏。因此需要从新访问一次路由才行。  
> 解决方法：`next({...to,replace:true})`

- `next({...to,replace:true})` :

  `...to` 如果 addRoutes 并未完成，路由守卫会重复执行去，直到 addRoutes 完成，找到对应的路由

  `replace:true` 在 to 执行的时候不允许用户点击回退，防止产生异常

## onBeforeRouteLeave

!>浏览器的回退按钮无法触发该守卫

官方 issue 的回答是拦不住，用户多次点击回退不能把他困在当前页面

## 嵌套路由

```javascript

  routes: [
    {
      path: "/",
      redirect: "/login"
    },
    {
      path: "/login",
      name: "login",
      component: () => import("@/views/login/login.vue")
    },
    //  url为 /map 会显示PageLayout及子路由map
    //  虽然输入 / 重定向到 login ，但是输入 /map 会进入map
    {
      path: "/",
      component: PageLayout,
      redirect: "/map",
      children: [
        {
        path: "map",
        component: () => import("@/views/map/index.vue")
        }
      ]
    }
 
```

## 路由触发同名方法

情景：

- 在 vue-router 中的路由里面，有重定向到 `'/login'` 的情况
- 在请求接口的方法里面有一个 `login` 的方法

  <img src="https://s1.ax1x.com/2022/07/06/jUvm0U.png">

  <img src="https://s1.ax1x.com/2022/07/06/jUvVXV.png">

问题：

!>本地运行时正常，打包发布后，进入 `/login` 页面，只要一进入就调用 `login` 方法

解决：产生问题的原因可能是由于编译之后，方法名与 router 的 path 名称一致，改掉方法名

<img src="https://s1.ax1x.com/2022/07/06/jUvemT.png">

# async&await

## 问题 1：

- 做链式的异步请求时，async 写在了在外面的方法名前面
- await 后面跟的返回值不是 Promise，而是一个值

<img src='https://s1.ax1x.com/2022/04/28/LXSlSH.png' height=500></img>

## 问题 2：

!>await 只能拿到 then 里面的值，如果请求出错 promise 会直接抛出异常，如果不需要做额外处理，可以不写 try catch

```javascript
const getUserList = async () => {
  const res: any = await userList({});
  state.userList = res;
};
```

# setTimeout 0

将 `setTimeout` 的延迟时间设置为 0 ，是为了将函数放在异步队列，等同步任务完成才执行

```javascript
setTimeout(() => {}, 0);
```

# forEach 能否改变数组

## 基本类型

- 数组元素是基本类型，不改变原数组

```javascript
const array = [1, 2, 3, 4];
array.forEach((ele) => {
  ele = ele * 3;
});
console.log(array); // [1,2,3,4]
```

## 引用类型

- 数组元素是引用类型，改变原数组

```javascript
const objArr = [
  {
    name: "wxw",
    age: 22
  },
  {
    name: "wxw2",
    age: 33
  }
];
objArr.forEach((ele) => {
  if (ele.name === "wxw2") {
    ele.age = 88;
  }
});
console.log(objArr); // [{name: "wxw", age: 22},{name: "wxw2", age: 88}]
```

- 改变数组的整个元素，不改变原数组

```javascript
const changeItemArr = [
  {
    name: "wxw",
    age: 22
  },
  {
    name: "wxw2",
    age: 33
  }
];
changeItemArr.forEach((ele) => {
  if (ele.name === "wxw2") {
    ele = {
      name: "change",
      age: 77
    };
  }
});
console.log(changeItemArr); // [{name: "wxw", age: 22},{name: "wxw2", age: 33}]
```

## 总结

- 基本类型我们当次循环拿到的`ele`，只是`forEach`给在另一个地方复制创建新元素，是和原数组这个元素没有半毛钱联系的！所以，我们使命给循环拿到的 ele 赋值都是无用功！
- 引用类型真正的数据是保存在堆内存，栈内只保存了对象的变量以及对应的堆的地址，所以操作 Object 其实就是直接操作了原数组对象本身。
- `forEach` 的基本原理也是 for 循环，使用`arr[index]`的形式赋值改变，无论什么就都可以改变了。

# 对象

```javascript
// 对象的键可以是数字、字符串、变量、symbol，但是都会被转化为字符串
const xx = Symbol("xx");

const name = {
  1: "小李",
  2: "小虞",
  3: "张三",
  A: "aa",
  [xx]: "王五"
};
console.log(name[1]); // 小李
console.log(name["3"]); // 张三
console.log(name["A"]); // aa
console.log(name.A); // aa
console.log(name[xx]); // 王五
```

# deep 样式穿透

使用`:deep()`时,他的父级必须是当前 vue 文件定义的某个选择器，如下列的`.my-dialog `

```scss
.my-dialog {
    :deep(.el-dialog__header) {
        border-bottom: 1px solid #d8d8d8;
    }
}
```

# 深拷贝

遇到数组、对象（引用类型）赋值时一定要考虑是否需要深拷贝

问题：红圈处开始时直接赋值，在后面 `arrTemp` 使用了 `splice` 方法，由于此时赋值的时内存地址，导致 `taskData.value` 也发生了改变

<img src='https://s1.ax1x.com/2022/07/22/jOagVs.jpg' >

# try catch

问题：使用 try catch 捕获异常，抛出错误时 报错 `(local var) error: unknown`，或者其他错误。

对 Promise 使用 async awiat 时不需要使用 try catch。

```typescript
try {
  // ...
} catch (error) {
  throw new Error(error);
}
```

解决：在 ts.config.json 中配置：

```json
 "compilerOptions": {
  //  ...
    "useUnknownInCatchVariables": false,
  //  ...
  }
```

# iframe 页面通信

## 子页面触发父页面事件

```javascript
// iframe
window.parent.postMessage(data, "*");

//父级
window.addEventListener(
  "message",
  function (e) {
    console.log(e);
  },
  false
);
```

## 子页面改变父页面 url

```typescript
(top as Window).window.location.href = "url";
```

# 浏览器预设样式 user agent stylesheet

场景: 使用 element 的输入框，当 Google 自动填充密码时，会出现背景色

原因：由于浏览器的预设样式，导致输入框的背景色变化

<img src='https://s1.ax1x.com/2022/07/02/j1NhlV.png'>

<img src='https://s1.ax1x.com/2022/07/02/j1NfS0.png'>

- 写入如下 css 覆盖

```css
/* 去除浏览器input自动填充的默认样式 */
input:-internal-autofill-selected {
    /* 这里#fff是底色 */
    -webkit-box-shadow: 0 0 0px 1000px #fff inset;
    -webkit-text-fill-color: #333;
}
```

- 更改之后的输入框

<img src='https://s1.ax1x.com/2022/07/02/j1UOEQ.png'>
