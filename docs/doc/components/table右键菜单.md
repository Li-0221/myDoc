index.vue

```vue
<template>
  <div class="content">
    <el-table
      :data="tableData"
      style="width: 100%"
      @row-contextmenu="handContextmenu"
    >
      <el-table-column prop="date" label="Date" width="180" />
      <el-table-column prop="name" label="Name" width="180" />
      <el-table-column prop="address" label="Address" />
    </el-table>
    <Menu v-if="show" ref="menuRef" @close="show = false" />
  </div>
</template>

<script lang="ts" setup>
import Menu from "./Menu.vue";

const tableData = [
  {
    date: "2016-05-03",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles"
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles"
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles"
  },
  {
    date: "2016-05-01",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles"
  }
];

const show = ref(false);
const menuRef = ref();
const handContextmenu = (row: any, column: any, event: any) => {
  const { x, y } = event;
  event.preventDefault();
  show.value = true;
  nextTick(() => {
    menuRef.value.init(row, column, event);
  });
};
</script>

<style scoped>
.content {
  width: 700px;
  border: 1px solid #ccc;
  margin: 0 auto;
}
</style>
```

Menu.vue

```vue
<template>
  <div id="contextmenu" class="contextmenu">
    <div class="contextmenu__item" @click="handleOne">菜单一</div>
    <div class="contextmenu__item" @click="handleThree">菜单三</div>
  </div>
</template>

<script lang="ts" setup>
const emit = defineEmits(["close"]);

const init = (row: any, column: any, event: any) => {
  const { x, y } = event;

  const content = document.querySelector(".content") as HTMLDivElement;
  const menu = document.querySelector("#contextmenu") as HTMLDivElement;

  const menuWidth = menu.offsetWidth;
  const menuHeight = menu.offsetHeight;
  const contentWidth = content.offsetWidth;
  const contentHeight = content.offsetHeight;

  const X = x - content.offsetLeft; //上边距
  const Y = y - content.offsetTop; //下边距

  menu.style.left = x + "px";
  menu.style.top = y + "px";

  if (X > contentWidth - menuWidth) {
    menu.style.left = x - menuWidth + "px";
  }
  if (Y > contentHeight - menuHeight) {
    menu.style.top = y - menuHeight + "px";
  }

  document.addEventListener("click", foo);
};

const foo = () => {
  emit("close");
};
const handleOne = () => {};
const handleThree = () => {};
onBeforeUnmount(() => {
  document.removeEventListener("click", foo);
});
defineExpose({ init });
</script>

<style scoped>
.contextmenu__item {
  display: block;
  line-height: 34px;
  text-align: center;
}

.contextmenu__item:not(:last-child) {
  border-bottom: 1px solid rgba(64, 158, 255, 0.2);
}

.contextmenu {
  position: absolute;
  background-color: #ecf5ff;
  width: 100px;
  /*height: 106px;*/
  font-size: 12px;
  color: #409eff;
  border-radius: 4px;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  border: 1px solid rgba(64, 158, 255, 0.2);
  white-space: nowrap;
  z-index: 1000;
}

.contextmenu__item:hover {
  cursor: pointer;
  background: #66b1ff;
  border-color: #66b1ff;
  color: #fff;
}
</style>
```
