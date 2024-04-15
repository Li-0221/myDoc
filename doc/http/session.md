# 注意事项

- 响应头里的 set cookie 会写入到浏览器的 cookie

- axios 设置 withCredentials 会允许向浏览器写入 cookie ，同时发送请求也会带上 cookie

- 同域的 XMLHttpRequest 不需要设置 withCredentials

# 实现步骤

## 响应头

后端响应头里面有 set-cookie 会写入到浏览器的 cookie ，如图：

<img src='https://s1.ax1x.com/2022/09/07/vHXLyd.png' width=500>

## set-cookie

- 在 axios 里面设置 `withCredentials` 允许接口写入 `cookie` 到浏览器，同时发送请求也会带上 cookie

```javascript
const service = axios.create({
  // baseURL: process.env.VUE_APP_BASE_URL,
  timeout: 8000,
  withCredentials: true
});
```

set-cookie 写入后：

<img src='https://s1.ax1x.com/2022/09/07/vHXbSe.png' width=500>

## 代理

本地开发时，如果服务端和客户端不同源，响应头的 set-cookie 可能会报警告，所以需要设置代理。

```javascript
module.exports = {
  devServer: {
    proxy: {
      "/api": {
        target: "<url>",
        ws: true, //websocket
        changeOrigin: true //跨域时，更改源
      }
    }
  }
};
```

## 发送 cookie

效果如下：

<img src='https://s1.ax1x.com/2022/09/07/vHXqQH.png' width=500>
