# Express 的基本使用

## 基础示例

一个接口里面只能有一个 res.send 生效，如果有多个 res.send 时，要注意使用 return。
换言之，不是写了 res.send 之后，后面的代码就不执行了

<img src="https://s1.ax1x.com/2022/04/18/LdewSx.md.png" height=270/>

## 路由模块化

<img src="https://s1.ax1x.com/2022/04/19/L0eo7D.png"/>

步骤 1-4 如下  
<img src="https://s1.ax1x.com/2022/04/19/L0ebhd.png"/>

导入并注册  
<img src="https://s1.ax1x.com/2022/04/19/L0e7Ae.png"/>

为 userRouter 所有的地址增加一个 api 前缀  
<img src="https://s1.ax1x.com/2022/04/19/L0eL9A.png"/>

## 中间件

!>一定要注意写中间件写的顺序

### 基本使用

<img src="https://s1.ax1x.com/2022/04/19/L0eO1I.md.png"/>

!>此处的[cb0,cb1,cb2]的中括号可以省略

<img src="https://s1.ax1x.com/2022/04/19/L0eXct.md.png"/>

<img src="https://s1.ax1x.com/2022/04/19/L0ejjP.md.png"/>

### 全局的中间件

> 调用所有的接口都会触发这个中间件

一个请求可能会有多个中间件，多个中间件共同享用的一份 res，req  
所以可以在前面的中间件中设置一些自定义属性或者方法，提供给下游的中间件或者路由使用  
可以有多个全局的中间件，按照注册的顺序执行

<img src="https://s1.ax1x.com/2022/04/18/LdeUYR.md.png" height=370/>

### 路由中间件

绑定在 router 上的中间件，只对这个路由实例有效

<img src="https://s1.ax1x.com/2022/04/18/LdeNk9.md.png" height=330/>

### 错误中间件

一般使用 app.use 全局注册，注册在路由后
捕获异常，防止项目要崩溃

<img src="https://s1.ax1x.com/2022/04/18/LdeYTJ.md.png" height=220/>

### 内置中间件

express.json 解析 json 格式的请求体数据  
express.urlencoded 解析 urlencoded 格式的请求体数据  
express.static 托管静态资源，如图片、html、css

<img src="https://s1.ax1x.com/2022/04/19/L0ezB8.md.png"/>

!>注意：不对请求体的数据做任何解析，直接拿 req.body 会 undefined

### 第三方中间件

<img src="https://s1.ax1x.com/2022/04/19/L0mSHS.md.png"/>

### 自定义中间件

监听 req 的 data 和 end  
<img src="https://s1.ax1x.com/2022/04/18/Ldeaf1.md.png" height=300/>

自定义一个响应中间件

!>放在身份认证前面  
踩坑：验证 token 的中间件写在了响应中间件前面，调用需要身份认证的接口时，如果 token 验证失败报错了，就直接跳过响应中间件进入错误处理中间件了，此时在错误中间件中使用响应中间件的内容会出问题

<img src="https://s1.ax1x.com/2022/04/18/LdeB6K.md.png" height=400/>

## cors 中间件解决跨域

### 使用方法

<img src="https://s1.ax1x.com/2022/04/19/L0exnf.md.png" />

### 请求头

<img src="https://s1.ax1x.com/2022/04/19/L0m9Ag.md.png" />
<img src="https://s1.ax1x.com/2022/04/19/L0mCNQ.md.png" />
<img src="https://s1.ax1x.com/2022/04/19/L0mF9s.md.png" />

## Express 操作 MySQL

### 安装并配置依赖

```powershell
npm install mysql
```

<img src="https://s1.ax1x.com/2022/04/18/LdeDOO.md.png"  height=400/>

### 增删改查

> 增加

插入对应的字段

<img src="https://s1.ax1x.com/2022/04/18/Lde6TH.md.png" height=300 />

如果对象的每个属性和数据表的字段对应，可简写

<img src="https://s1.ax1x.com/2022/04/18/LdesmD.md.png"   height=300/>

> 更新

更改指定的值

<img src="https://s1.ax1x.com/2022/04/18/Ldey0e.md.png"  height=300/>

如果数据对象的每个属性和数据库表字段对应，可简写

<img src="https://s1.ax1x.com/2022/04/18/Ldegkd.md.png"  height=300/>

> 删除

直接删除一条数据  
但是不建议这样做，数据一旦删除恢复就比较麻烦  
使用标记删除，可以在表中设置一个类似 state 的状态字段，使用 update 更改这条数据的状态

<img src="https://s1.ax1x.com/2022/04/18/Lde2tA.md.png"  height=300/>

## 身份认证

### JWT 认证（json web token）

<img src="https://s1.ax1x.com/2022/04/19/L0mAcq.png"  />

JWT 由三部分组成，分别是 Header（头部）、Payload（有效载荷）、Signature（签名），通常用 . 分隔开  
Header 和 Signature 是安全性相关的部分  
Payload 是用户信息经过加密之后的字符串

### 使用 JWT

```powershell
npm install jsonwebtoken express-jwt
```

基础使用

<img src="https://s1.ax1x.com/2022/04/19/L0eHtH.md.png"  height=1200/>

!>注意：访问带有权限的接口时，需要在请求头上面添加

<img src="https://s1.ax1x.com/2022/04/19/L0mPhj.md.png"  />

## 数据库加密

对用户的密码进行加密处理

```powershell
npm i bcryptjs
```

<img src="https://s1.ax1x.com/2022/04/18/LdeRfI.md.png"  height=120/>
<img src="https://s1.ax1x.com/2022/04/19/L0ehX6.md.png"  height=150/>
<img src="https://s1.ax1x.com/2022/04/19/L0e5nK.md.png"  height=150/>

## 表单数据验证

> 后端永远不要相信前端传来的数据，即使前端已经验证了
> 安装验证规则包

```powershell
npm install joi
```

安装中间件，实现对表单数据的验证

```powershell
npm install @escook/express-joi
```

新建用户信息模块？schema/user.js 定义验证规则

<img src="https://s1.ax1x.com/2022/04/19/L0eI0O.md.png" height=400 />

在需要验证表单的地方导入并使用  
错误中间件中添加表单验证失败的错误处理

<img src="https://s1.ax1x.com/2022/04/19/L0ef6x.md.png"  height=200/>

> 参考
> https://www.npmjs.com/package/@escook/express-joi/v/1.1.1
