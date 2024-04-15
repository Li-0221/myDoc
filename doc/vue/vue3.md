# Vue3 基础

## 响应式

### Vue2 响应式

实现原理：

- 对象类型：通过 `Object.defineProperty()`对属性的读取、修改、拦截
- 数组类型：通过重写数组原型上的一系列方法实现拦截

存在问题：

- 新增属性、删除属性，界面不会更新
- 直接通过下标修改数组，界面不会更新

解决以上问题：

```javascript
// 给对象新增属性
this.$set(this.person,'sex','女')

// 删除对象的name属性
this.$delete(this.person,'name')

// 修改数组的第一个元素
this.$set(this.arr,0，'xxx')
```

### Vue3 响应式

实现原理：

- 通过 Proxy（代理）：拦截对象中任意属性的变化，包括属性的读写、添加、删除等
- 通过 Reflect（反射）：对被代理对象的属性进行操作

## 组件传值

!> 使用 v-model 后。子组件必须写 defineProps 否则子组件 emit 改变 modelValue 时会出现 bug。 因为 v-model 是数据双向绑定。不写 defineProps 会数据不流通。无法绑定成功

父组件：

```vue
<template>
  <div><child v-model="isShow"></child></div>
</template>

<script lang="ts" setup>
const isShow = ref(true);
</script>
```

子组件：

```vue
<template>
  <div>{{ modelValue }}</div>
</template>

<script lang="ts" setup>
const props = defineProps<{
  modelValue: boolean;
}>();

const emits = defineEmits(["update:model-value"]);

// 更新父组件的值
const submit = () => {
  emits("update:model-value", false);
};
</script>
```

## ref&reactive

### 数据定义

- 一般使用 `ref` 定义基本数据类型，通过 `Object.defineProperty()`的 `get`、`set` 方法实现响应式
- 一般使用 `reactive` 定义复杂数据类型，通过 Proxy 实现响应式，并通过 Reflect 操作源对象内部数据
- 如果使用 `ref` 定义复杂数据类型，内部会自动通过 reactive 转为代理对象

### 使用

- `ref` 定义的数据，操作数据要`.value`，在模板中直接使用，不需要`.value`
- 使用`ref`获取 dom

```vue
<template>
  <div ref="myRef">获取单个DOM元素</div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
export default {
  //这里的myRef名字要保持一致
  const myRef = ref(null);
  console.log(myRef.value);
 }
};
</script>
```

### reactive 存在的问题

!>直接给 reactive 定义的变量赋值，会失去响应式

```javascript
const arr = reactive([]);
const load = () => {
  const res = [2, 3, 4, 5]; //假设请求接口返回的数据
  // 方法1 失败，直接赋值丢失了响应性
  // arr = res;
  // 方法2 这样也是失败
  // arr.concat(res);
};
```

原因如下：

- 方法 1：arr = res 时，直接把一个 res 新数组赋值给了 arr。reactive 声明的响应式对象被 arr 代理，操作代理对象需要有代理对象的前缀，此时的 res 直接把值赋值给了 arr ，使得 arr 失去了响应式。在 vue3 使用 proxy，对于对象或数组都不能直接将整个数据赋值。

- 方法 2 时，把 arr 直接当成一个数组，经过 reactive 包装之后，arr 已经不是普通的数组了，所以方法 2 也失效。

> 解决方案

- 使用 ref 定义（推荐）
- 数组的 push 方法
- 创建一个响应式对象，对象的属性为数组

```javascript
// 1.ref定义
const arr = ref([]);
arr.value = [1, 2, 3];

// 2.push方法
let list = reactive([]);
let arr = [1, 2, 3];
list.push(...arr);

// 3.创建响应式对象
let state = reactive({
  list: []
});
let arr = [1, 2, 3];
state.list = arr;
```

简单粗暴的方式：可以除了定义对象，都用 ref 定义

另外 ref 都可以用 const，reactive 用 let

## computed

```javascript
import { computed } from "@vue";

// 简写
let fullName = computed(() => {
  return person.firstName + " " + person.lastName;
});

// 完整
let fullName = computed({
  get() {
    return person.firstName + " " + person.lastName;
  },
  set(val) {
    let names = val.split(" ");
    person.firstName = names[0];
    person.lastName = names[1];
  }
});
```

## watch

!>监视 `reactive` 定义的数据时，`oldValue `无法正确获取，开启了深度监视（deep 配置失效）  
监视 `reactive` 定义的数据的某个属性时，deep 配置有效

```javascript
// 监听ref定义的数据
watch(
  sum,
  (newVal, oldVal) => {
    console.log(newVal, oldVal);
  },
  { immediate: true }
);

// 监听多个ref定义的数据
watch(
  [sum, msg],
  (newVal, oldVal) => {
    console.log(newVal, oldVal);
  },
  { immediate: true }
);

// 监听多个reactive定义的数据
watch(
  person,
  (newVal, oldVal) => {
    console.log(newVal, oldVal); //oldVal 无法正确获取
  },
  { immediate: true, deep: false } //deep配置失效
);

// 监听reactive定义的数据的某个属性
watch(
  () => person.job,
  (newVal, oldVal) => {
    console.log(newVal, oldVal);
  },
  { immediate: true, deep: true } //deep配置有效
);

// 监听reactive定义的数据的多个属性
watch(
  [() => person.job, () => person.job],
  (newVal, oldVal) => {
    console.log(newVal, oldVal);
  },
  { immediate: true, deep: true }
);
```

## watchEffect

> `watchEffect` 默认开启深度监听和立即执行

`watchEffect` 不用说明监听的是什么，只要他的回调函数用到了什么属性，就监听什么属性

```javascript
watchEffect(() => {
  const x = sum;
  const y = msg;
  console.log("触发了监听");
});
```

watchEffect 与 computed 的区别：

- computed 注重计算出来的值（回调函数的返回值），必须要写返回值
- watchEffect 注重过程（回调函数的函数体），不需要写返回值

## 生命周期

|   选项式 API    | Hook inside <small>setup</small> |
| :-------------: | :------------------------------: |
|  beforeCreate   |           Not needed\*           |
|     created     |           Not needed\*           |
|   beforeMount   |          onBeforeMount           |
|     mounted     |            onMounted             |
|  beforeUpdate   |          onBeforeUpdate          |
|     updated     |            onUpdated             |
|  beforeUnmount  |         onBeforeUnmount          |
|    unmounted    |           onUnmounted            |
|  errorCaptured  |         onErrorCaptured          |
|  renderTracked  |         onRenderTracked          |
| renderTriggered |        onRenderTriggered         |
|    activated    |           onActivated            |
|   deactivated   |          onDeactivated           |

```javascript
export default {
  setup() {
    // mounted
    onMounted(() => {
      console.log("Component is mounted!");
    });
  }
};
```

## 异步组件 Suspense

在大型应用中，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块。为了实现这个效果，Vue 有一个 defineAsyncComponent 方法。

```vue
<template>
  <div>
    {{ list }}
  </div>
</template>

<script setup lang="ts">
// 模拟http请求
const httpRequest = () => {
  return new Promise((resolve) => {
    setTimeout(() => {
      const arr = [1, 2, 3, 4];
      resolve(arr);
    }, 3000);
  });
};

// 在setup最外层使用await不需要用async
const list = await httpRequest();
</script>
```

```vue
<template>
  <div class="bg-slate-50 h-5">
    <!-- 使用Suspense包裹异步组件，组件A加载完才会显示 -->
    <Suspense>
      <!-- 组件加载成功后显示 -->
      <template #default> <A></A></template>

      <!-- 在A组件加载中时显示 -->
      <template #fallback>
        <div>loading...</div>
      </template>
    </Suspense>
  </div>
</template>

<script setup lang="ts">
import { defineAsyncComponent } from "vue";

// 将组件A以异步组件的方式导入
const A = defineAsyncComponent(() => import("./components/A.vue"));
</script>
```

## 传送组件 Teleport

`to` 的值必须是有效的查询选择器或 HTMLElement (如果在浏览器环境中使用)。指定将在其中移动 `<teleport>` 内容的目标元素

```html
<!-- 正确 -->
<teleport to="#some-id" />
<teleport to=".some-class" />
<teleport to="[data-teleport]" />

<!-- 错误 -->
<teleport to="h1" />
<teleport to="some-string" />

<!-- 将teleport内的东西传送到id为popup的元素下 -->
<teleport to="#popup" >
  <video src="./my-movie.mp4">
</teleport>
```

## keep-alive 缓存组件

当组件在` <keep-alive>` 内被切换时，它的 `mounted` 和 `unmounted` 生命周期钩子不会被调用，取而代之的是 `activated` 和 `deactivated`。(这会运用在 `<keep-alive>` 的直接子节点及其所有子孙节点。)

主要用于保留组件状态或避免重新渲染。

```html
<!-- 基本 -->
<keep-alive>
  <component :is="view"></component>
</keep-alive>

<!-- 多个条件判断的子组件 -->
<keep-alive>
  <comp-a v-if="a > 1"></comp-a>
  <comp-b v-else></comp-b>
</keep-alive>

<!-- 和 `<transition>` 一起使用 -->
<transition>
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>
</transition>
```

## transition 过渡组件

- 属性

  - enter-from-class 进入过渡的开始状态的类名
  - enter-to-class 进入过渡的结束状态的类名
  - enter-active-class 进入过渡生效时的状态的类名

  - leave-from-class 离开过渡的开始状态的类名
  - leave-active-class 离开过渡生效时的状态的类名
  - leave-to-class 离开过渡的结束状态的类名

- 搭配 animate.css 使用
  主要使用 enter-active-class leave-active-class

```vue
<template>
  <button @click="flag = !flag">切换按钮</button>

  <transition
    enter-active-class="animate__animated animate__rubberBand"
    leave-active-class="animate__animated animate__rubberBand"
  >
    <div class="w-20 h-20 bg-red-200" v-show="flag"></div>
  </transition>
</template>
```

## transition-group 列表过渡

用法与 transition 基本一致

`<transition-group> `提供了多个元素/组件的过渡效果。默认情况下，它不会渲染一个 DOM 元素包裹器，但是可以通过 tag attribute 来定义。

注意，每个 `<transition-group>` 的子节点必须有独立的 key，动画才能正常工作。

当一个子节点被更新，从屏幕上的位置发生变化，这个子节点就会产生动画效果。

```vue
<transition-group
  tag="ul"
  enter-active-class="animate__animated animate__rubberBand"
  leave-active-class="animate__animated animate__rubberBand"
>
  <li v-for="item in items" :key="item.id">
    {{ item.text }}
  </li>
</transition-group>
```

## Provide&Inject

父组件有一个 `provide` 选项来提供数据，子组件有一个 `inject` 选项来开始使用这些数据。
<img src='https://v3.cn.vuejs.org/images/components_provide.png'></img>

祖先组件中：

```vue
<script setup lang="ts">
import { provide, ref } from "vue";

let num = ref<number>(1);
// 为xxx提供一个响应式数据
provide("xxx", num);
</script>
```

孙组件中：

```vue
<script setup lang="ts">
import { inject, Ref } from "vue";

// 将数据xxx注入，并赋值给data
const data = inject("xxx");

// 点击孙组件的按钮，改变祖先组件的数据
const add = () => {
  (data as Ref<number>).value++;
};
</script>
```

## 全局函数&变量

由于 Vue3 `没有Prototype` 属性 使用 `app.config.globalProperties` 代替 然后去定义变量和函数

```javascript
// Vue 2
Vue.prototype.$http = () => {};

// Vue3
const app = createApp({});
app.config.globalProperties.$http = () => {};
```

举个例子：
main.ts 中定义

```typescript
// ts中需要类型声明
declare module "@vue/runtime-core" {
  export interface ComponentCustomProperties {
    $hello: () => string;
  }
}

// 挂载方法
app.config.globalProperties.$hello = () => {
  return "hello";
};
```

使用全局方法：

```vue
<template>
  <div>{{ $hello() }}</div>
</template>

<script lang="ts" setup>
import { getCurrentInstance, ComponentInternalInstance } from "vue";

//getCurrentInstance获取的是当前vue实例
const { proxy } = getCurrentInstance() as ComponentInternalInstance;
console.log(proxy?.$hello());
</script>
```

# 单文件组件`<script setup>`

`<script setup>` 是在单文件组件 (SFC) 中使用`组合式 API` 的编译时语法糖。相比于普通的 `<script>` 语法，它具有更多优势：

- 更少的样板内容，更简洁的代码。
- 能够使用纯 Typescript 声明 props 和抛出事件。
- 更好的运行时性能 (其模板会被编译成与其同一作用域的渲染函数，没有任何的中间代理)。
- 更好的 IDE 类型推断性能 (减少语言服务器从代码中抽离类型的工作)。

## 基本语法

要使用这个语法，需要将 setup attribute 添加到 `<script> `代码块上：

```vue
<script setup>
console.log("hello script setup");
</script>
```

> `<script setup>` 中的代码会在每次组件实例被创建的时候执行。

### 顶层的绑定会被暴露给模板

任何在 `<script setup>` 声明的顶层的绑定 (包括变量，函数声明，以及 import 引入的内容) 都能在模板中直接使用：

```vue
<script setup>
// 变量
const msg = "Hello!";

// 函数
function log() {
  console.log(msg);
}
</script>

<template>
  <div @click="log">{{ msg }}</div>
</template>
```

import 导入的内容也会以同样的方式暴露。意味着可以在模板表达式中直接使用导入的 helper 函数，并不需要通过 methods 选项来暴露它：

```vue
<script setup>
import { capitalize } from "./helpers";
</script>

<template>
  <div>{{ capitalize("hello") }}</div>
</template>
```

## 响应式

响应式状态需要明确使用响应式`APIs`来创建。和从 `setup()` 函数中返回值一样，ref 值在模板中使用的时候会自动解包：

```vue
<script setup>
import { ref } from "vue";

const count = ref(0);
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

## 使用组件

`<script setup>` 范围里的值也能被直接作为自定义组件的标签名使用：

```vue
<script setup>
import MyComponent from "./MyComponent.vue";
</script>

<template>
  <MyComponent />
</template>
```

将 `MyComponent` 看做被一个变量所引用。如果你使用过 JSX，在这里的使用它的心智模型是一样的。其 kebab-case 格式的 `<my-component>`同样能在模板中使用。不过，我们强烈建议使用 PascalCase 格式以保持一致性。同时也有助于区分原生的自定义元素。

### 动态组件

由于组件被引用为变量而不是作为字符串键来注册的，在 `<script setup>` 中要使用动态组件的时候，就应该使用动态的 `:is` 来绑定：

```vue
<script setup>
import Foo from "./Foo.vue";
import Bar from "./Bar.vue";
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```

> 请注意组件是如何在三元表达式中被当做变量使用的。

### 递归组件

一个单文件组件可以通过它的文件名被其自己所引用。例如：名为 `FooBar.vue` 的组件可以在其模板中用 `<FooBar/>` 引用它自己。

请注意这种方式相比于 import 导入的组件优先级更低。如果有命名的 import 导入和组件的推断名冲突了，可以使用 import 别名导入：

```javascript
import { FooBar as FooBarChild } from "./components";
```

### 命名空间组件

可以使用带点的组件标记，例如 `<Foo.Bar>` 来引用嵌套在对象属性中的组件。这在需要从单个文件中导入多个组件的时候非常有用：

```vue
<script setup>
import * as Form from "./form-components";
</script>

<template>
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
```

## 自定义指令

全局注册的自定义指令将以符合预期的方式工作，且本地注册的指令可以直接在模板中使用，就像上文所提及的组件一样。

但这里有一个需要注意的限制：必须以 `vNameOfDirective` 的形式来命名本地自定义指令，以使得它们可以直接在模板中使用。do'c

```vue
<script setup>
<template>
  <div>
    <div v-BackGround="{ background: 'red' }">1111</div>
  </div>
</template>

<script setup lang="ts">
import { Directive, DirectiveBinding } from 'vue';
const vBackGround: Directive = {
  // el是指令所在的html元素，binding是指令传递的值
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    el.style.background = binding.value.background
  },
  updated() { },
  unmounted() { }
}
</script>
```

```vue
<script lang="ts">
// 你可能想在 mounted 和 updated 时触发相同行为，而不关心其他的钩子函数，那么你可以通过将这个回调函数传递给指令来实现
const vBackGround2: Directive = (
  el: HTMLElement,
  binding: DirectiveBinding
) => {
  el.style.background = binding.value.background;
};
</script>
```

## defineProps 和 defineEmits

在 `<script setup>` 中必须使用 `defineProps` 和 `defineEmits` API 来声明 `props` 和 `emits` ，它们具备完整的类型推断并且在 `<script setup>` 中是直接可用的：

```vue
<script setup>
const props = defineProps({
  foo: String
});

const emit = defineEmits(["change", "delete"]);
// setup code
</script>
```

- `defineProps` 和 `defineEmits` 都是只在 `<script setup>` 中才能使用的编译器宏。他们不需要导入且会随着 `<script setup>` 处理过程一同被编译掉。

- `defineProps` 接收与 `props` 选项相同的值，`defineEmits` 也接收 `emits` 选项相同的值。

- `defineProps` 和 `defineEmits` 在选项传入后，会提供恰当的类型推断。

- 传入到 `defineProps` 和 `defineEmits` 的选项会从 `setup` 中提升到模块的范围。因此，传入的选项不能引用在 setup 范围中声明的局部变量。这样做会引起编译错误。但是，它可以引用导入的绑定，因为它们也在模块范围内。

> 如果使用了 Typescript，<a href="/Vue/Vue3?id=%e4%bb%85%e9%99%90-typescript-%e7%9a%84%e5%8a%9f%e8%83%bd">使用纯类型声明来声明 prop 和 emits </a> 也是可以的。

## defineExpose

使用 `<script setup>` 的组件是默认关闭的，也即通过模板 ref 或者 `$parent `链获取到的组件的公开实例，不会暴露任何在 `<script setup> `中声明的绑定。

为了在 `<script setup>` 组件中明确要暴露出去的属性，使用 `defineExpose` 编译器宏：

```vue
<script setup>
import { ref } from "vue";

const a = 1;
const b = ref(2);

defineExpose({
  a,
  b
});
</script>
```

当父组件通过模板 ref 的方式获取到当前组件的实例，获取到的实例会像这样 `{ a: number, b: number }` (ref 会和在普通实例中一样被自动解包)

## useSlots 和 useAttrs

在 `<script setup>` 使用 `slots` 和 `attrs` 的情况应该是很罕见的，因为可以在模板中通过 `$slots` 和 `$attrs` 来访问它们。在你的确需要使用它们的罕见场景中，可以分别用 `useSlots` 和 `useAttrs` 两个辅助函数：

```vue
<script setup>
import { useSlots, useAttrs } from "vue";

const slots = useSlots();
const attrs = useAttrs();
</script>
```

`useSlots` 和 `useAttrs` 是真实的运行时函数，它会返回与 `setupContext.slots`和 `setupContext.attrs` 等价的值，同样也能在普通的组合式 API 中使用。

## 与普通的 `<script>` 一起使用

`<script setup>` 可以和普通的 `<script>` 一起使用。普通的 `<script>` 在有这些需要的情况下或许会被使用到：

- 无法在 `<script setup>` 声明的选项，例如 `inheritAttrs` 或通过插件启用的自定义的选项。
- 声明命名导出。
- 运行副作用或者创建只需要执行一次的对象。

```vue
<script>
// 普通 <script>, 在模块范围下执行(只执行一次)
runSideEffectOnce();

// 声明额外的选项
export default {
  inheritAttrs: false,
  customOptions: {}
};
</script>

<script setup>
// 在 setup() 作用域中执行 (对每个实例皆如此)
</script>
```

!>WARNING
该场景下不支持使用 `render` 函数。请使用一个普通的 `<script>` 结合 `setup` 选项来代替。

## 顶层 await

`<script setup>` 中可以使用顶层 `await`。结果代码会被编译成 `async setup()`：

```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json());
</script>
```

另外，`await` 的表达式会自动编译成在 `await` 之后保留当前组件实例上下文的格式。

!>注意`async setup()` 必须与 `Suspense` `组合使用，Suspense` 目前还是处于实验阶段的特性。

## 仅限 TypeScript 的功能

### 仅限类型的 props/emit 声明

props 和 emits 都可以使用传递字面量类型的纯类型语法做为参数给 `defineProps` 和 `defineEmits` 来声明：

```typescript
const props = defineProps<{
  foo: string;
  bar?: number;
}>();

const emit = defineEmits<{
  (e: "change", id: number): void;
  (e: "update", value: string): void;
}>();
```

- `defineProps` 或 `defineEmits` 只能是要么使用运行时声明，要么使用类型声明。同时使用两种声明方式会导致编译报错。

- 使用类型声明的时候，静态分析会自动生成等效的运行时声明，以消除双重声明的需要并仍然确保正确的运行时行为。

  - 在开发环境下，编译器会试着从类型来推断对应的运行时验证。例如这里从 `foo: string` 类型中推断出 `foo: String`。如果类型是对导入类型的引用，这里的推断结果会是 `foo: null` (与 `any` 类型相等)，因为编译器没有外部文件的信息。

  - 在生产模式下，编译器会生成数组格式的声明来减少打包体积 (这里的 props 会被编译成 `['foo', 'bar'])`。

  - 生成的代码仍然是有着类型的 Typescript 代码，它会在后续的流程中被其它工具处理。

- 截至目前，类型声明参数必须是以下内容之一，以确保正确的静态分析：

  - 类型字面量

  - 在同一文件中的接口或类型字面量的引用

  现在还不支持复杂的类型和从其它文件进行类型导入。理论上来说，将来是可能实现类型导入的。

### 使用类型声明时的默认 props 值

仅限类型的 `defineProps` 声明的不足之处在于，它没有可以给 props 提供默认值的方式。为了解决这个问题，提供了 `withDefaults` 编译器宏：

```typescript
interface Props {
  msg?: string;
  labels?: string[];
}

const props = withDefaults(defineProps<Props>(), {
  msg: "hello",
  labels: () => ["one", "two"]
});
```

上面代码会被编译为等价的运行时 props 的 `default` 选项。此外，`withDefaults `辅助函数提供了对默认值的类型检查，并确保返回的 `props` 的类型删除了已声明默认值的属性的可选标志。
