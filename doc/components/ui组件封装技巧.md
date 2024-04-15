1. 想让写在 `MyInput` 上的属性传到 `el-input` 上
1. 想让写在 `MyInput` 上的插槽传到 `el-input` 上
1. 想让写在 `MyInput` 上的 ref 能获取到 `el-input` 的属性和方法

```vue
<template>
  <div class="w-52">
    <MyInput ref="myInputRef" v-model="data" :a="1" placeholder="请输入">
      <template #suffix>
        <el-icon class="el-input__icon"><calendar /></el-icon>
      </template>
      <template #prefix>
        <el-icon class="el-input__icon"><search /></el-icon>
      </template>
    </MyInput>
  </div>
</template>

<script lang="ts" setup>
import MyInput from "./components/MyInput.vue";
const data = ref("");
const myInputRef = ref("");
</script>
```

```vue
<template>
  <el-input ref="inp" v-bind="attrs">
    <template v-for="(value, name) in slots" #[name]="slotData">
      <slot :name="name" v-bind="slotData || {}" />
    </template>
  </el-input>
</template>

<script lang="ts" setup>
const instance = getCurrentInstance();

const attrs = useAttrs(); //拿到props中没有定义的属性
const slots = useSlots() || {}; //拿到插槽
const inp = ref();

onMounted(() => {
  const entries = Object.entries(inp.value);
  for (const [key, value] of entries) {
    instance![key] = value;
  }
});
</script>
```
