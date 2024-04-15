需求：级联选择，选择一级菜单默认选择二级所有菜单

组件封装 TreeSelect

```vue
<template>
  <div v-for="(item, index) in options" class="check-box">
    <van-checkbox
      shape="square"
      :class="`check-header group${gruopStatus[index]}`"
      @click="changeGroupStatus(index)"
      >{{ item.title }}</van-checkbox
    >
    <van-checkbox-group
      v-model="data[index]"
      @change="changeBox"
      class="check-body"
    >
      <van-checkbox
        shape="square"
        class="check-item"
        v-for="obj in item.children"
        :name="obj.key"
      >
        <img alt="" :src="obj.icon" style="width: 15px; height: 15px" />
        {{ obj.title }}
      </van-checkbox>
    </van-checkbox-group>
  </div>
</template>

<script lang="ts" setup>
import { computed, ref, watch } from "vue";

const props = defineProps<{
  options: any[];
  value: any[];
}>();

const emits = defineEmits(["update:value"]);

const data = ref<any[]>([]);
const gruopStatus = computed(() => {
  let status: any = [];
  data.value.forEach((item, index: number) => {
    status[index] = getGroupStatus(item.length, index);
  });
  return status;
});

watch(
  () => props.value,
  (v) => {
    let arr: any[] = [];
    props.options.forEach((item, index) => {
      const checkedArr = v.filter((obj) => obj.startsWith(item.key));
      arr.push(checkedArr);
    });
    if (JSON.stringify(data.value) !== JSON.stringify(arr)) {
      data.value = arr;
    }
  },
  { deep: true, immediate: true }
);

/**
 * 叶子节点复选框改变事件
 */
const changeBox = () => {
  let arr: any[] = [];
  data.value.forEach((item, index: number) => {
    gruopStatus.value[index] = getGroupStatus(item.length, index);
    arr = arr.concat(item);
  });
  console.log(arr);

  emits("update:value", arr);
};

/**
 * 获取群组状态
 * @param checkedLen 叶子节点选中个数
 * @param index 选项组下标
 * @return 选项组状态 0-未选中 1-半选 2-全选
 */
const getGroupStatus = (checkedLen: number, index: number): number => {
  let status: number;
  const groupOptions = props.options[index].children;
  if (checkedLen === 0) {
    status = 0;
  } else if (checkedLen === groupOptions.length) {
    status = 2;
  } else {
    status = 1;
  }
  return status;
};

const changeGroupStatus = (index: number) => {
  const oldStatus = gruopStatus.value[index];
  let newStatus: number;
  if (oldStatus !== 2) {
    // 非全选先全选
    data.value[index] = Array.from(
      props.options[index].children,
      (item: any) => item.key
    );
  } else {
    // 已经全选则全不选
    data.value[index] = [];
  }
};
</script>

<style lang="less" scoped>
.check-box {
  .check-header {
    height: 35px;
    padding-left: 15px;
    background: #f2f3f7;

    :deep(.van-checkbox__label) {
      font-size: 14px;
      font-weight: 500;
      color: #202020;
      margin-left: 14px;
    }
  }

  .group2 {
    :deep(.van-icon) {
      color: var(--van-white);
      background-color: var(--van-checkbox-checked-icon-color);
      border-color: var(--van-checkbox-checked-icon-color);
    }
  }

  .group1 {
    :deep(.van-icon) {
      color: var(--van-white);
      background-color: var(--van-checkbox-checked-icon-color);
      border-color: var(--van-checkbox-checked-icon-color);
    }

    :deep(.van-icon-success:before) {
      content: "\964";
    }
  }

  :deep(.van-checkbox__icon) {
    font-size: 14px;
  }
}

.check-box:first-child {
  margin-top: 15px;
}

.check-body {
  padding: 10px 0 10px 45px;

  :deep(.van-checkbox) {
    height: 20px;
    margin-bottom: 10px;
  }

  :deep(.van-checkbox__label) {
    font-size: 14px;
    font-weight: 500;
    color: #202020;
    margin-left: 14px;
    display: flex;
    align-items: center;

    img {
      margin-right: 10px;
    }
  }
}
</style>
```

使用

```vue
<template>
  <van-dialog v-model:show="showLandType" title="土地类型" width="90%">
    <TreeSelect :options="landTypeData" v-model:value="landTypeValue" />
  </van-dialog>
</template>
<script lang="ts" setup>
import { ref } from "vue";
const landTypeValue = ref([]);
const landTypeData = [
  {
    key: "01",
    title: "耕地",
    children: [
      { key: "011", title: "水田" },
      { key: "012", title: "水浇地" },
      { key: "013", title: "旱地" }
    ]
  },
  {
    key: "02",
    title: "园地",
    children: [
      { key: "021", title: "果园" },
      { key: "022", title: "茶园" },
      { key: "023", title: "其它园地" }
    ]
  },
  {
    key: "03",
    title: "林地",
    children: [
      { key: "031", title: "有林地" },
      { key: "032", title: "灌木林地" },
      { key: "033", title: "其它林地" }
    ]
  },
  {
    key: "04",
    title: "草地",
    children: [
      { key: "041", title: "天然牧草地" },
      { key: "042", title: "人工草地" },
      { key: "043", title: "其他草地" }
    ]
  }
];
</script>
```
