# 更多用法

https://cn.vuejs.org/guide/extras/render-function.html

# vue 中使用 tsx

## 简单的 vue tsx 组件

```tsx
// Test.tsx
import { defineComponent, provide, ref, renderSlot } from "vue";
import { Plus } from "@element-plus/icons-vue";

const Tabs = defineComponent({
  name: "ElTabs",

  props: {
    name: String,
    showButton: { type: Boolean, default: true },
    msg: { type: String, required: true }
  },

  emits: ["tabAdd"],

  setup(props, { emit, slots, expose }) {
    const num = ref(0);
    const buttonRef = ref<HTMLButtonElement>();

    const handleClick = () => {
      num.value++;
      emit("tabAdd", num.value);
    };

    provide("Tab", { num });

    expose({ num, handleClick });

    return () => {
      const NewButton = props.showButton ? (
        <button ref={buttonRef} onClick={handleClick}>
          按钮
          <Plus />
        </button>
      ) : null;

      return (
        <>
          <header>
            {renderSlot(slots, "header")}
            {props.msg}
            {renderSlot(slots, "default")}
          </header>

          <div>
            {/* 使用当前tsx中定义的组件 */}
            {NewButton}
            {/* 使用ref定义的变量 */}
            {num.value}
          </div>
        </>
      );
    };
  }
});

export type TabsInstance = InstanceType<typeof Tabs>;

export default Tabs;
```

## 使用 tsx 组件

```vue
<template>
  <Test msg="这是一条消息" @tab-add="xx" :show-button="true">
    <template #default>到默认插槽</template>
    <template #header>到 header 插槽</template>
  </Test>
</template>

<script lang="ts" setup>
import Test from "./test.tsx";

const xx = (num: number) => {
  console.log(num);
};
</script>
```

# element-plus 中的应用

使用 element-plus 的虚拟化表格时，需要在表格项中自定义渲染内容（比如渲染一个 Tag 标签，渲染一个下拉框组件等需要使用 tsx 语法。

这是 Element Plus 的 虚拟列表的 columns 其中一项， 相关参考 https://element-plus.org/zh-CN/component/table-v2.html

```typescript
{
    key: "operations",
    title: "操作",
    fixed: TableV2FixedDir.RIGHT,
    cellRenderer: ({ rowData }) => (
      <>
        <ElPopover
          placement="left"
          title={rowData.owner_name}  // :title='rowData.owner_nam'
          width="300"
          trigger="click"
          v-slots={{
            reference: () => <ElButton size="small">预览</ElButton>, // reference插槽
            default: () => <div v-html={parseContent(JSON.parse(rowData.content))}></div>  // 默认插槽
          }}
        ></ElPopover>

        <ElButton size="small" type="primary" onClick={() => toChat(rowData)}>  // @click=toChat(rowData)
          跳转
        </ElButton>
      </>
    ),
    width: 150,
    align: "center"
  }
```

# 事件

## 点击事件

onClick 没有参数时的写法

```typescript
function onClick() {
  console.log("点击了按钮");
}

function render() {
  return (
    <div>
      <button onClick={onClick}>按钮</button>
    </div>
  );
}
```

!>有参数时必须要写成箭头函数，` <button onClick={onClick(1)}>按钮</button>` 这样会变成立即执行函数。

```typescript
function onClick(e: number) {
  console.log(`点击了按钮${e}`);
}

function render() {
  return (
    <div>
      <button onClick={() => onClick(1)}>按钮</button>
    </div>
  );
}
```
