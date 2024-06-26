> 双 token 核心：在响应拦截器面进入 errHandle，在 errHandle 里面刷新 token，重新请求之前失效的请求

```javascript
import axios from "axios";
import errHandle from "@/utils /errHandle";

const baseUrl = process.env.VUE_APP_BASE_URL;

const request = axios.create({
  //请求的网址=baseURL+url   url 是调用 request 做请求传入的
  baseURL: baseUrl, // timeout: 5000,
  headers: { "X-Custom-Header": "foobar" }
});

// 添加请求拦截器
request.interceptors.request.use(
  function (config) {
    // 为所有的 request 请求都添加请求头，内容是 token
    const token = localStorage.getItem("token");
    if (token) config.headers.Authorization = `Bearer ${token}`;
    return config;
  },
  function (error) {
    return Promise.reject(error);
  }
);

// 添加响应拦截器
request.interceptors.response.use(
  function (response) {
    return response;
  },
  function (error) {
    const result = errHandle(error);
    return result ? Promise.resolve(result) : Promise.reject(error);
  }
);

export default request;
```

```javascript
import axios from "axios";
import request from "@/utils/request";
import store from "@/store/index";
import router from "@/router/index";

const baseUrl = process.env.VUE_APP_BASE_URL;

const instance = axios.create({
  baseURL: baseUrl,
  headers: { "X-Custom-Header": "foobar" }
});

const errHandle = async error => {
  // status===401 时 token 过期
  if (error.response.status === 401) {
    const refreshToken = localStorage.getItem("refreshToken");
    try {
      // 请求刷新 token 的接口
      // 不可以使用之前的 axios 实例，不然会进入拦截器，发现 token 失效，又进入到 error
      const result = await instance.post("/login/refresh", null, {
        headers: { Authorization: `Bearer ${refreshToken}` }
      });
      if (result) {
        store.commit("setToken", result.data.token); // 重新请求 token 过期报 401 的接口，接口信息在 error.config 里面
        return request(error.config);
      }
    } catch (error) {
      // 刷新 token 失败
      // localStorage.clear()
      // store.commit('setToken','')
      // store.commit('setUserInfo',{})
      // store.commit('setIsLogin',false)
      return router.push({ name: "login" });
    }
  }
};

export default errHandle;
```
