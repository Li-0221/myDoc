# 路由模式

## hash

- url 上带有#号

- 主要是通过 location.hash 来实现页面跳转

- hash 是 url 中# 后面的那部分，常用在锚点在页面内的导航，改变 url 中的 hash 不会刷新页面

- 通过 `window.addEventListener('hashchange'，function(){})`监听浏览器左右箭头的变化

## history

- url 上没有#号

- 主要通过 history.pushState 来实现页面跳转

- history 提供 pushState 和 replaceState 方法，这两个方法改变 url 的 path 部分不会刷新页面

- 通过`window.addEventListener('popstate'，function(){})`监听浏览器左右箭头的变化

# 编程式导航

!>如果提供了 path，params 会被忽略

```javascript
// 命名的路由，并加上参数
router.push({ name: "user", params: { username: "eduardo" } });

// 带查询参数
router.push({ path: "/register", query: { plan: "private" } });
```

# 历史记录

设置 `replace` 属性的话，当点击时，会调用 `router.replace()`，而不是 `router.push()`，所以导航后不会留下历史记录（浏览器不能通过左右箭头回退，快进）。

使用场景：用户登录后不想保留登录页信息

```html
<router-link to="/abc" replace></router-link>
```

# 命名视图

你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 router-view 没有设置名字，那么默认为 default。

```html
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>
<router-view class="view main-content"></router-view>
<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 components 配置 (带上 s)：

```javascript
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: "/",
      components: {
        // 它们与 `<router-view>` 上的 `name` 属性匹配
        //  放在对应的位置
        default: () => import("./Home.vue"),
        LeftSidebar: () => import("./LeftSidebar.vue"),
        RightSidebar: () => import("./RightSidebar.vue")
      }
    }
  ]
});
```

# 导航守卫

## beforeEach

> 守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于等待中。

```javascript
const router = createRouter({ ... })

router.beforeEach((to, from, next) => {
  if (to.name !== "Login" && !isAuthenticated) next({ name: "Login" });
  else next();
});
```

## next()

- `next()` : 放行
- `next('/login')` : 中断当前路由去 login

> 在 addRoutes()之后第一次访问被添加的路由会白屏，这是因为刚刚 addRoutes()就立刻访问被添加的路由，然而此时 addRoutes()没有执行结束，因而找不到刚刚被添加的路由导致白屏。因此需要从新访问一次路由才行。  
> 解决方法：`next({...to,replace:true})`

- `next({...to,replace:true})` :

  `...to` 如果 addRoutes 并未完成，路由守卫会重复执行去，直到 addRoutes 完成，找到对应的路由

  `replace:true` 在 to 执行的时候不允许用户点击回退，防止产生异常

## 组件内守卫

!>浏览器回退无法触发 onBeforeRouteLeave 函数

```javascript
onBeforeRouteLeave((to, from) => {
  const answer = window.confirm(
    "Do you really want to leave? you have unsaved changes!"
  );
  // 取消导航并停留在同一页面上
  if (!answer) return false;
});

const userData = ref();

onBeforeRouteUpdate(async (to, from) => {
  //仅当 id 更改时才获取用户，例如仅 query 或 hash 值已更改
  if (to.params.id !== from.params.id) {
    userData.value = await fetchUser(to.params.id);
  }
});
```

# 路由元信息

有时，你可能希望将任意信息附加到路由上，如过渡名称、谁可以访问路由等。这些事情可以通过接收属性对象的 meta 属性来实现，并且它可以在路由地址和导航守卫上都被访问到。定义路由的时候你可以这样配置 meta 字段：

```javascript
const routes = [
  {
    path: '/posts',
    component: PostsLayout,
    children: [
      {
        path: 'new',
        component: PostsNew,
        // 只有经过身份验证的用户才能创建帖子
        meta: { requiresAuth: true }
      },
      {
        path: ':id',
        component: PostsDetail
        // 任何人都可以阅读文章
        meta: { requiresAuth: false }
      }
    ]
  }
]
```

routes 配置中的每个路由对象为 路由记录。路由记录可以是嵌套的，因此，当一个路由匹配成功后，它可能匹配多个路由记录。

例如，根据上面的路由配置，`/posts/new` 这个 URL 将会匹配父路由记录 `(path: '/posts')` 以及子路由记录 `(path: 'new')`。

一个路由匹配到的所有路由记录会暴露为 `$route` 对象， Vue Router 提供了一个 `$route.meta`方法，它是一个非递归合并所有 meta 字段的方法。这意味着你可以简单地写

```javascript
router.beforeEach((to, from) => {
  if (to.meta.requiresAuth && !auth.isLoggedIn()) {
    // 此路由需要授权，请检查是否已登录
    // 如果没有，则重定向到登录页面
    return {
      path: "/login",
      // 保存我们所在的位置，以便以后再来
      query: { redirect: to.fullPath }
    };
  }
});
```

## TypeScript

可以通过扩展 `RouteMeta` 接口来输入 meta 字段：

```typescript
// typings.d.ts or router.ts
import "vue-router";

declare module "vue-router" {
  interface RouteMeta {
    // 是可选的
    isAdmin?: boolean;
    // 每个路由都必须声明
    requiresAuth: boolean;
  }
}
```

# 组合式 api

## 在 setup 中访问路由和当前路由

因为我们在 setup 里面没有访问 this，所以我们不能再直接访问 this.$router 或 this.$route。作为替代，我们使用 useRouter 函数：

```javascript
import { useRouter, useRoute } from "vue-router";

export default {
  setup() {
    const router = useRouter();
    const route = useRoute();

    function pushWithQuery(query) {
      router.push({
        name: "search",
        query: {
          ...route.query
        }
      });
    }
  }
};
```

route 对象是一个响应式对象，所以它的任何属性都可以被监听，但你应该避免监听整个 route 对象。在大多数情况下，你应该直接监听你期望改变的参数。

```javascript
import { useRoute } from "vue-router";
import { ref, watch } from "vue";

export default {
  setup() {
    const route = useRoute();
    const userData = ref();

    // 当参数更改时获取用户信息
    watch(
      () => route.params.id,
      async (newId) => {
        userData.value = await fetchUser(newId);
      }
    );
  }
};
```

# 过渡动效

> 以下示例均搭配 animate.css，基于本站 `transition 过渡组件`

所有的路由使用同一种过渡效果

```html
<router-view v-slot="{ Component }">
  <transition enter-active-class="animate__animated animate__fadeIn">
    <component :is="Component" />
  </transition>
</router-view>
```

## 单个路由过渡

如果你想让每个路由的组件有不同的过渡，你可以将元信息和不同的 amimate 结合在一起，放在<transition> 上：

```javascript
const routes = [
  {
    path: "/custom-transition",
    component: PanelLeft,
    meta: { transition: "animate__fadeIn" }
  },
  {
    path: "/other-transition",
    component: PanelRight,
    meta: { transition: "animate__fadeInDown" }
  }
];
```

```html
<!-- router-view支持插槽，可以解出Component, route -->
<router-view v-slot="{ Component, route }">
  <transition enter-active-class="animate__animated ${route.meta.transition}">
    <component :is="Component" />
  </transition>
</router-view>
```

## 强制过渡

Vue 可能会自动复用看起来相似的组件，从而忽略了任何过渡。幸运的是，可以添加一个 key 属性来强制过渡。这也允许你在相同路由上使用不同的参数触发过渡：

```html
<router-view v-slot="{ Component, route }">
  <transition enter-active-class="animate__animated ${route.meta.transition}">
    <component :is="Component" :key="route.path" />
  </transition>
</router-view>
```

# 滚动行为

!>这个功能只在支持 history.pushState 的浏览器中可用。

当创建一个 Router 实例，你可以提供一个 scrollBehavior 方法：

```javascript
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      //如果当前页面向下滚动了300px,则下一个页面离顶部也是300px
      return savedPosition;
      // 如果滚动出现异常，尝试添加 behavior
      // return { ...savedPosition, behavior: "smooth" };
    } else {
      return { top: 0, left: 0 };
    }
  }
});
```

## 延迟滚动

有时候，我们需要在页面中滚动之前稍作等待。例如，当处理过渡时，我们希望等待过渡结束后再滚动。要做到这一点，你可以返回一个 Promise，它可以返回所需的位置描述符。下面是一个例子，我们在滚动前等待 500ms：

```javascript
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve({ left: 0, top: 0 });
      }, 500);
    });
  }
});
```

# 动态路由

一般使用动态路由都是后台返回一个路由表，前端通过`router.addRoutes()`添加路由。

## 添加路由

```javascript
router.addRoute({ path: "/about", component: About });
```

## 添加嵌套路由

要将嵌套路由添加到现有的路由中，可以将路由的 name 作为第一个参数传递给 `router.addRoute()`，这将有效地添加路由，就像通过 children 添加的一样：

```javascript
router.addRoute({ name: "admin", path: "/admin", component: Admin });
router.addRoute("admin", { path: "settings", component: AdminSettings });
```

这等效于：

```javascript
router.addRoute({
  name: "admin",
  path: "/admin",
  component: Admin,
  children: [{ path: "settings", component: AdminSettings }]
});
```

## 删除路由

有几个不同的方法来删除现有的路由：

- 通过添加一个名称冲突的路由。如果添加与现有途径名称相同的途径，会先删除路由，再添加路由：

```javascript
router.addRoute({ path: "/about", name: "about", component: About });
// 这将会删除之前已经添加的路由，因为他们具有相同的名字且名字必须是唯一的
router.addRoute({ path: "/other", name: "about", component: Other });
```

- 通过调用 router.addRoute() 返回的回调：

```javascript
const removeRoute = router.addRoute(routeRecord);
removeRoute(); // 删除路由如果存在的话
```

> 当路由没有名称时，这很有用。

- 通过使用 router.removeRoute() 按名称删除路由：

```javascript
router.addRoute({ path: "/about", name: "about", component: About });
// 删除路由
router.removeRoute("about");
```

需要注意的是，如果你想使用这个功能，但又想避免名字的冲突，可以在路由中使用 `Symbol` 作为名字。
当路由被删除时，所有的别名和子路由也会被同时删除
