# 目录

## components

组件：写在这里的 vue 文件会直接注册成组件，它们将在整个应用程序中自动可用，而无需显式地导入它们。

## pages

页面： 写在这里的 vue 文件会直接注册成页面 ，所有的页面都要写在这个文件夹里面，并且当前页面的路由路径就是文件名。

!> index.vue 对应路径为 '/'

## layouts

布局： 布局模版，写在这的 vue 文件会直接注册成布局。

## composables

工具方法：类似于 utils，里面的方法可以在 .js, .ts 和 .vue 文件中使用，无需导入。

## middleware

路由中间件：类似于导航守卫。

# 路由

## Navigation

### NuxtLink

类似于`<router-link></router-link>`。

```vue
<template>
  <header>
    <nav>
      <ul>
        <li><NuxtLink to="/about">About</NuxtLink></li>
        <li><NuxtLink to="/posts/1">Post 1</NuxtLink></li>
        <li><NuxtLink to="/posts/2">Post 2</NuxtLink></li>
      </ul>
    </nav>
  </header>
</template>
```

### NuxtPage

类似于 `<router-view></router-view>`。

## 动态路由

### 文件目录

pages 下面文件命名为 `[参数名].vue`，示例如下。

- `[id].vue`

- `shop-[name].vue`

### 路由跳转

```vue
<template>
  <div>
    <NuxtLink to="/99">去动态页面，参数id=99</NuxtLink>
    <NuxtLink to="/bag">去动态页面，参数name='bag'</NuxtLink>
  </div>
</template>
```

### 读取参数

```vue
<template>
  <div>动态参数为{{ route.params.id }}</div>
</template>

<script lang="ts" setup>
const route = useRoute();
</script>
```

## 嵌套路由/嵌套页面

### 文件目录

1. pages 下新建 `parent.vue` 文件、`parent` 文件夹。

2. 在 parent 文件夹内新建 `child-one.vue` 文件、`child-two.vue` 文件。

3. 嵌套页面的路由跳转

   ```vue
   <template>
     <div>
       <NuxtLink to="/parent/child-one">子页面one</NuxtLink>
       <NuxtLink to="/parent/child-two">子页面two</NuxtLink>

       <!-- 如果parent下有index.vue，路由如下 -->
       <NuxtLink to="/parent">子页面index</NuxtLink>
     </div>
   </template>
   ```

## Route Parameters

useRoute()组合可以在`<script setup>`块或 Vue 组件的 `setup()` 方法中使用，以访问当前路由细节。

- pages/post/[id].vue

```vue
<script setup>
const route = useRoute();
// 当路由地址为 /posts/1, 输出 1
console.log(route.params.id);
</script>
```

## Route Middleware

类似于 `routerBeforeEach`

```typescript
export default defineNuxtRouteMiddleware((to, from) => {
  // isAuthenticated() is an example method verifying if an user is authenticated
  if (isAuthenticated() === false) {
    return navigateTo("/login");
  }
});
```

# layouts

## 新建

layouts 下新建 `default.vue`，会自动注册名为 default 的模版。

```vue
<template>
  <div>
    <div>header</div>

    <slot></slot>
    <!-- 具名插槽 -->
    <slot name="one"></slot>
    <div>footer</div>
  </div>
</template>
```

## 使用

```vue
<template>
  <div>
    <NuxtLayout name="default">
      <div>页面主要内容</div>
      <template #one>放在插槽one中的内容</template>
    </NuxtLayout>
  </div>
</template>
```

# components

## 组件

- 目录

```
| components/
--| TheHeader.vue
--| TheFooter.vue
Copy to clipboard
layouts/default.vue
```

- layouts/default.vue

```vue
<template>
  <div>
    <TheHeader />
    <slot />
    <TheFooter />
  </div>
</template>
```

## 懒加载组件

要动态导入一个组件(也称为惰性加载组件)，你所需要做的就是在组件名称前添加 `Lazy` 前缀。

- layouts/default.vue

```vue
<template>
  <div>
    <TheHeader />
    <slot />
    <LazyTheFooter />
  </div>
</template>
```

# Composables

## 定义

带名字的导出

**Method 1**: Using named export

- composables/useFoo.ts

```typescript
export const useFoo = () => {
  return useState("foo", () => "bar");
};
```

**Method 2**: Using default export

- composables/use-foo.ts or composables/useFoo.ts

```typescript
// 它将作为 useFoo()导出 （驼峰不带扩展名的文件名大小写）
export default function () {
  return useState('foo', () => 'bar')
```

## 使用

- view/xxx.vue

```vue
<template>
  <div>
    {{ foo }}
  </div>
</template>
<script setup>
const foo = useFoo();
</script>
```

# Middleware

> 路由中间件是导航守卫，它接收当前路由和下一个路由作为参数。

```javascript
export default defineNuxtRouteMiddleware((to, from) => {
  if (to.params.id === "1") {
    return abortNavigation();
  }
  return navigateTo("/");
});
```

Nuxt 提供了两个全局可用的辅助函数，它们可以直接从中间件返回:

- navigateTo (to: RouteLocationRaw | undefined | null, options?: { replace: boolean, redirectCode: number, external: boolean ) - 在插件或中间件中重定向到给定的路由。也可以直接调用它来执行页面导航。

- abortNavigation (err?: string | Error) - 终止导航，并显示一条可选的错误消息。

# 过渡

Nuxt 利用 Vue 的<Transition>组件在页面和布局之间应用过渡。

## 开启过渡

- nuxt.config.ts

```typescript
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  app: {
    // 页面过渡
    pageTransition: { name: "page", mode: "out-in" },
    // 布局过渡
    layoutTransition: { name: "slide", mode: "out-in" }
  }
});
```

## 过渡样式

- app.vue

```vue
<style>
.page-enter-active,
.page-leave-active {
  transition: all 0.4s;
}
.page-enter-from,
.page-leave-to {
  opacity: 0;
  filter: blur(1rem);
}
</style>
```

# 数据获取

[参考最佳实践](https://nuxt.com.cn/docs/getting-started/data-fetching#%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)
