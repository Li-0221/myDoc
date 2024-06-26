## node 版本管理

```powershell
//安装n模块
sudo npm install n -g

//安装稳定版
sudo n stable

//安装最新版
sudo n latest

//安装指定版本
sudo n 版本号   // 8.16.0   /   12.8.0
```

## 依赖更新

```powershell
//下载ncu
npm install -g npm-check-updates

//检查package.json中的依赖是否有更新
ncu

//更新package.json中的依赖（需要删除node_modules重新install）
ncu -u

```
