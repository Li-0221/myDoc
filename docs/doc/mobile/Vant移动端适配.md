# 移动端适配方案

> 结论：目前移动端最佳适配方案： viweport + flex

移动端特有的 `meta` ：`<meta name="viewport" content="width=device-width, initial-scale=1.0" />`

viewport 的单位有： vw vh vmin vmax

vw 和 vh 类似于百分比单位

在 `flexible` 的[仓库](https://github.com/amfe/lib-flexible)，有如下一段话：  
由于 `viewport` 单位得到众多浏览器的兼容，`lib-flexible` 这个过渡方案已经可以放弃使用，不管是现在的版本还是以前的版本，都存有一定的问题。建议大家开始使用 `viewport` 来替代。

## viewport

### 引入插件

- 介绍：
  [postcss-px-to-viewport](https://github.com/evrone/postcss-px-to-viewport)，将 px 单位转换为视口单位的 (vw, vh, vmin, vmax) 的 PostCSS 插件。

- 安装：

```powershell
npm install postcss-px-to-viewport --save-dev
yarn add -D postcss-px-to-viewport
```

### vite 中

因为 vite 中已经内联了 postcss，所以并不需要额外的创建 postcss.config.js 文件，我们只需要在 vite.config.ts 中进行配置即可：具体配置如下:

!> 该配置中 `viewportWidth` 为 375 是为了兼容 vant 的设计稿宽度为 375。  
由于 vite 配置暂时不支持动态设置 `viewportWidth` 建议让 ui 设计稿设计为 375。

```javascript
//vite.config.ts
import { defineConfig } from "vite";
import postcsspxtoviewport from "postcss-px-to-viewport";

export default defineConfig({
  // ......
  css: {
    postcss: {
      plugins: [
        postcsspxtoviewport({
          viewportWidth: 375 // vant是375  因为目前vite不支持动态设置 建议ui也是375
        })
      ]
    }
  }
});
```

### vite 集成 tailwindcss

在 vite 中使用 tailwindcss 和 postcss-px-to-viewport 时，需要在 postcss.config.js 中配置，并在 vite.config.js 中移除 postcss。

> 由于 tailwindcss 使用的是 rem 为基本单位，所以写宽高字体大小等只能使用 `基类-[100px]` 的形式，如 `w-[100px]、text-[16px]`

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
    // postcss-px-to-viewport写在后面，不然不会转换tailwindcss的px单位
    "postcss-px-to-viewport": {
      unitToConvert: "px",
      viewportWidth: 375 //设计稿宽度  vant是375   建议ui也是375
    }
  }
};
```

### vue-cli 中

> 如果读取的是 vant 相关的文件，viewportWidth 就设为 375，如果是其他的文件，我们就按照我们 UI 的宽度来设置 viewportWidth。
> 详见：[移动端 postcss-px-to-viewport 兼容 vant](https://www.cnblogs.com/zhangnan35/p/12682925.html)

```javascript
// postcss.config.js
const path = require("path");

module.exports = ({ webpack }) => {
  const designWidth = webpack.resourcePath.includes(
    path.join("node_modules", "vant")
  )
    ? 375
    : 750;

  return {
    plugins: {
      autoprefixer: {},
      "postcss-px-to-viewport": {
        viewportWidth: designWidth // 设计稿的宽度
      }
    }
  };
};
```

## rem

原理：通过 `flexible` 设置根元素字体大小，使用 rem 来替代 px。实现了不同大小的屏幕都适应相同的样式了。

### 引入插件

```powershell
npm i -S amfe-flexible
npm install postcss postcss-pxtorem --save-dev
```

```javascript
// main.js
import "amfe-flexible/index.min.js";
```

### vite 中

!>没实践过，可能有问题

```javascript
//vite.config.ts
import { defineConfig } from "vite";
import postCssPxToRem from "postcss-pxtorem";

export default defineConfig({
  // ......
  css: {
    postcss: {
      plugins: [
        postCssPxToRem({
          unitToConvert: "px", // 要转化的单位
          viewportWidth: 375, // vant是375  因为目前vite不支持动态设置 建议ui也是375
          unitPrecision: 6, // 转换后的精度，即小数点位数
          mediaQuery: true, // 是否在媒体查询的css代码中也进行转换，默认false
          replace: true, // 是否转换后直接更换属性值
          exclude: [],
          landscape: false // 是否处理横屏情况
        })
      ]
    }
  }
});
```

### vue-cli 中

> 如果设计稿的尺寸不是 375，而是 750 或其他大小，可以将 rootValue 配置为:

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    // postcss-pxtorem 插件的版本需要 >= 5.0.0
    "postcss-pxtorem": {
      rootValue({ file }) {
        return file.indexOf("vant") !== -1 ? 37.5 : 75;
      },
      propList: ["*"]
    }
  }
};
```
