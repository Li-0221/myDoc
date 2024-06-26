# 封装

!>在拦截器里面 , return 进入 then , reject 进入 catch

```typescript
import axios from "axios";
import type { AxiosRequestConfig, AxiosResponse } from "axios";
import { ElMessage, ElMessageBox, ElLoading } from "element-plus";
import { useMainStore } from "@/store/index";
import router from "@/router/index";
// import NProgress from "nprogress";

const store = useMainStore();
let requestount = 0;
let loadingInstance;
// 设置请求头和请求路径
const request = axios.create({
  baseURL: import.meta.env.VITE_BASE_URL as string | undefined,
  timeout: 10000
});

request.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    const { token } = store.user;
    requestount++;
    loadingInstance = ElLoading.service({
      lock: true,
      text: "加载中...",
      spinner: "el-icon-loading",
      background: "rgba(0, 0, 0, 0.7)"
    });
    if (token) {
      //@ts-ignore
      config.headers.Authorization = token;
    }
    return config;
  },
  (error) => {
    if (--requestount === 0) {
      loadingInstance.close();
    }
    return Promise.reject(error);
  }
);
// 响应拦截
request.interceptors.response.use(
  // 2xx 范围内的http状态码都会触发该函数。
  (response) => {
    if (--requestount === 0) {
      loadingInstance.close();
    }

    const res = response.data;
    // 此处code是后端自定义的码
    if (res.code && res.code !== "200") {
      ElMessage({
        message: res.msg || "Error",
        showClose: true,
        type: "error",
        duration: 5 * 1000
      });

      if (res.code && res.code === "401") {
        // to re-login
        ElMessageBox.confirm("会话失效，请重新登录", "Warning", {
          confirmButtonText: "确认",
          type: "warning"
        }).then(() => {
          router.push("/login");
        });
      }
      return Promise.reject(res || "Error");
    }
    return res;
  },
  // 超出 2xx 范围的http状态码都会触发该函数。
  (error) => {
    if (--requestount === 0) {
      loadingInstance.close();
    }
    ElMessage({
      message: error.message || "Error",
      showClose: true,
      type: "error",
      duration: 5 * 1000
    });
    return Promise.reject(error);
  }
);

export default request;
```

# 示例

## api

```typescript
import request, { HttpResponse } from "@/service/request";

export interface LoginData {
  username: string;
  password: string;
}

export interface LoginRes {
  token: string;
}

export function logout() {
  // res的类型默认是 AxiosResponse<any, any>
  return request.post("/api/user/logout");
}

export function login(data: LoginData) {
  // res的类型定义为HttpResponse<LoginRes>
  return request.post<LoginRes, HttpResponse<LoginRes>>(
    "/api/user/login",
    data
  );
}
```

## 调用

```typescript
try {
  // 上面设置了res类型为 HttpResponse<LoginRes>
  const res = await login({ username: "admin", password: "admin" });

  // 注意这个data，不要写漏 这里data的类型是 LoginRes
  const token = res.data.token;
} catch (error) {
  throw new Error(error);
}
```

也可以写成如下方式：

```typescript
try {
  const { data } = await login({ username: "admin", password: "admin" });
  const token = data.token;
} catch (error) {
  throw new Error(error);
}
```
