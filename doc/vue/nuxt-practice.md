# 前言

这一章记录的是在实际开发中可能会遇到的问题

# 水合

水合可以理解为服务端生成 html 结构

# 配置

可以在 `.env` 和 `nuxt.config.ts` 中写环境变量

## 区分开发和生产

开发环境会读 `.env` 文件 生产环境不会，而且 `.env` 文件会覆盖 `runtimeConfig` 里面的内容。  
所以 `runtimeConfig` 可以直接写生产环境的配置，在 `.env` 文件里写开发环境的配置。

```
//env
NUXT_PUBLIC_API_BASE='http://localhost:5600'
```

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    apiSecret: "", // 可以由.env文件中的 NUXT_API_SECRET 环境变量覆盖
    public: {
      apiBase: "" // 可以由.env文件中的 NUXT_PUBLIC_API_BASE 环境变量覆盖
    }
  }
});
```

## 读取

vue 模板中，可以使用 `$config.public` 访问公开的运行时配置

!> 不能在 ts 文件顶部使用 `useRuntimeConfig()` , 要放在具体的方法里面。

```vue
<template>
  <div>
    {{ $config.public }}
  </div>
</template>

<script setup lang="ts">
const config = useRuntimeConfig();
console.log(config.public.apiBase);
</script>
```

# 执行

1. 服务端执行： `setup` 顶层、`process.serve`判断内
2. 客户端执行： `onMounted` 内、`process.client`判断内

```vue
<template>
  <div></div>
</template>

<script setup lang="ts">
// `setup` 顶层的代码会在服务端执行
if (process.serve) {
  //服务端执行
}

if (process.client) {
  //客户端执行
}
onMounted(() => {
  // `onMounted` 里面的代码会在客户端执行
});
</script>
```

# 数据请求

一般请求有分为两种

1. 服务端请求（服务端渲染） `useAsyncData` 搭配 `$fetch`
2. 客户端请求（客户端渲染） 直接使用`$fetch`

## 请求 API

```ts
// /composables/index.ts
//  获取更新记录
export const fetchRecord = () => {
  const config = useRuntimeConfig();
  return $fetch(`${config.public.apiBase}log/list`, {
    method: "post",
    body: { pageNum: 1, pageSize: 999 }
  });
};
```

## 服务端执行请求（服务端渲染）

如果想将请求好的数据 v-for 出来返回给客户端（客户端查看源码可以看到 v-for 出来的内容）  
必须要在顶层 await 请求，因为 nuxt 用了 vue 的 Suspense 组件，不在顶层会报水合错误

> 服务端请求 `useAsyncData` 搭配 `$fetch`

```vue
<template>
  <div>{{ record }}</div>
</template>

<script lang="ts" setup>
import { fetchRecord } from "~/composables";

// useAsyncData 的第一个参数是key 有缓存作用 可以根据请求的参数动态设置  如`logs:${id.value}`
// useAsyncData 返回值里面的data是响应式的
const { data: record }: any = await useAsyncData("logs", fetchRecord);
</script>
```

## 客户端执行请求（客户端渲染）

> 客户端请求 直接使用`$fetch`

```vue
<template>
  <div>{{ record }}</div>
</template>

<script lang="ts" setup>
import { fetchRecord } from "~/composables";
const record = ref<any>();

onMounted(async () => {
  const { data } = await fetchRecord();
  console.log(data);
  record.value = data;
});
</script>
```
