## 封装的属性

| 参数        | 说明                                | 类型    | 默认值            |
| ----------- | ----------------------------------- | ------- | ----------------- |
| total       | 总条目数                            | Number  | /                 |
| page        | 当前页数，支持 .sync 修饰符         | Number  | 1                 |
| limit       | 每页显示条目个数，支持 .sync 修饰符 | Number  | 20                |
| page-sizes  | 每页显示个数选择器的选项设置        | Number  | []10, 20, 30, 50] |
| hidden      | 是否隐藏                            | Boolean | false             |
| auto-scroll | 分页之后是否自动滚动到顶部          | Boolean | true              |

## 分页组件 pageination.vue

```vue
<template>
  <div :class="{ hidden: hidden }" class="pagination-container">
    <el-pagination
      :background="background"
      :current-page.sync="currentPage"
      :page-size.sync="pageSize"
      :layout="layout"
      :page-sizes="pageSizes"
      :total="total"
      v-bind="$attrs"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>
<script>
//详见工具方法/页面滚动
import { scrollTo } from "@/utils/scroll-to";
export default {
  name: "Pagination",
  props: {
    total: {
      required: true,
      type: Number
    },
    page: {
      type: Number,
      default: 1
    },
    limit: {
      type: Number,
      default: 20
    },
    pageSizes: {
      type: Array,
      default() {
        return [10, 20, 30, 50];
      }
    },
    layout: {
      type: String,
      default: "total, sizes, prev, pager, next, jumper"
    },
    background: {
      type: Boolean,
      default: true
    },
    autoScroll: {
      type: Boolean,
      default: true
    },
    hidden: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    currentPage: {
      get() {
        return this.page;
      },
      set(val) {
        this.$emit("update:page", val);
      }
    },
    pageSize: {
      get() {
        return this.limit;
      },
      set(val) {
        this.$emit("update:limit", val);
      }
    }
  },
  methods: {
    handleSizeChange(val) {
      this.$emit("pagination", { page: this.currentPage, limit: val });
      if (this.autoScroll) {
        scrollTo(0, 800);
      }
    },
    handleCurrentChange(val) {
      this.$emit("pagination", { page: val, limit: this.pageSize });
      if (this.autoScroll) {
        scrollTo(0, 800);
      }
    }
  }
};
</script>
<style scoped>
.pagination-container {
  background: #fff;
  padding: 32px 16px;
}
.pagination-container.hidden {
  display: none;
}
</style>
```
