# 虚拟列表

适用于大数据渲染列表的情况，当数据过多时，一次渲染所有数据会导致页面卡顿，使用虚拟列表可以只渲染可视区域的数据，从而提高渲染性能。

列表始终只渲染了一定的元素，当滚动时，列表会将不可见的元素移除，同时将可见区域的元素添加进来。

## VirtualizedList.tsx

```tsx
import { defineComponent, ref, nextTick, watch, computed } from "vue";
import { ElScrollbar } from "element-plus";

// 根据实际情况定义列表数据类型 必须要含有id字段 或者自行修改tsx中使用id的地方
interface ListData extends Record<string, any> {
  id: number;
}

type LoadType = "Prev" | "Next";

const VirtualizedList = defineComponent({
  name: "VirtualizedList",

  props: {
    list: { type: Array<ListData>, default: () => [] },
    size: { type: Number, default: 50 },
    cache: { type: Number, default: 10 }
  },

  emits: ["loadNext", "loadPrev"],

  setup(props, { emit, slots, expose }) {
    const scrollbarRef = ref<InstanceType<typeof ElScrollbar>>();
    const loading = ref(false);
    const currentList = ref<ListData[]>([]);

    const SIZE = ref(50);
    const CACHE = ref(10);

    watch(
      () => props.list,
      (val) => {
        if (val.length === 0) return;

        if (val.length < SIZE.value) {
          currentList.value = val;
          return;
        }
        if (val.length > SIZE.value) {
          currentList.value = val.slice(0, SIZE.value);
          return;
        }
      }
    );

    watch(
      () => props.size,
      (val) => (SIZE.value = val)
    );

    watch(
      () => props.cache,
      (val) => (CACHE.value = val)
    );

    const start = computed(() =>
      props.list.findIndex((item) => item.id === currentList.value[0].id)
    );
    const end = computed(() =>
      props.list.findIndex(
        (item) => item.id === currentList.value[currentList.value.length - 1].id
      )
    );

    const load = async (loadType: LoadType) => {
      if (!currentList.value) return;
      const list = JSON.parse(JSON.stringify(props.list));
      const lenght = list.length;
      loading.value = true;
      if (loadType === "Prev") {
        const cache = start.value + 1 > CACHE.value ? CACHE.value : start.value;
        currentList.value = list.slice(
          start.value - cache,
          end.value - cache + 1
        );
      } else {
        const cache =
          lenght - end.value - 1 > CACHE.value
            ? CACHE.value
            : lenght - end.value - 1;
        currentList.value = list.slice(
          start.value + cache,
          end.value + cache + 1
        );
      }
      emit(`load${loadType}`);
      await nextTick();
      loading.value = false;
    };

    const scroll = async ({ scrollTop }: any) => {
      if (loading.value) return;

      const scrollWrap = scrollbarRef.value!.wrapRef;
      const contentHeight = scrollWrap!.children[0].clientHeight;
      const wrapHeight = scrollWrap!.clientHeight;
      const scrollBottom = contentHeight - wrapHeight - scrollTop;

      let loadType: LoadType = "Next";
      if (scrollTop < 100 && start.value > 0) {
        loadType = "Prev";
        load(loadType);
      }
      if (scrollBottom < 100 && end.value < props.list.length - 1) {
        loadType = "Next";
        load(loadType);
      }
    };

    const renderList = () =>
      currentList.value
        ? currentList.value.map(({ id }) => {
            return (
              <li key={id} class="py-2 text-center leading-10 h-10 ">
                {id}
              </li>
            );
          })
        : null;

    return () => {
      return (
        <ElScrollbar ref={scrollbarRef} onScroll={scroll} class="h-full">
          {renderList}
        </ElScrollbar>
      );
    };
  }
});

export type TabsInstance = InstanceType<typeof VirtualizedList>;

export default VirtualizedList;
```

## 使用

```vue
<template>
  <div class="h-full">
    <VirtualizedList
      :list="list"
      class="h-full"
      @load-next="loadNext"
      @load-prev="loadPrev"
    />
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import VirtualizedList from "./VirtualizedList";

const list = ref<any>([]);

const generateList = (length = 200, prefix = "list-") =>
  Array.from({ length }).map((_, index) => ({
    id: `${prefix}${index}`
  }));

const loadNext = () => {
  console.log("loadNext");
};
const loadPrev = () => {
  console.log("loadPrev");
};

setTimeout(() => {
  list.value = generateList(106);
}, 1000);
</script>
```
