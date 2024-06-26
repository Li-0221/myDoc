# iconfont 字体图标

## symbol 引用

> 这是一种全新的使用方式，应该说这才是未来的主流。

- 支持多色图标了，不再受单色限制。
- 通过一些技巧，支持像字体那样，通过 font-size, color 来调整样式。
- 兼容性较差，支持 IE9+，及现代浏览器。
- 浏览器渲染 SVG 的性能一般，还不如 png。

第一步：引入项目下面生成的 symbol 代码。

```html
<script src="./iconfont.js"></script>
```

第二步：加入通用 css（引入一次即可）。

```html
<style>
  .icon {
    width: 1em;
    height: 1em;
    vertical-align: -0.15em;
    fill: currentColor;
    overflow: hidden;
  }
</style>
```

第三步：挑选相应图标并获取类名，应用于页面。

```html
<svg class="icon" aria-hidden="true">
  <use xlink:href="#icon-xxx"></use>
</svg>
```

## font-class 引用

> font-class 是 Unicode 使用方式的一种变种，主要是解决 Unicode 书写不直观，语意不明确的问题。

- 相比于 Unicode 语意明确，书写更直观。可以很容易分辨这个 icon 是什么。
- 因为使用 class 来定义图标，所以当要替换图标时，只需要修改 class 里面的 Unicode 引用。

第一步：引入项目下面生成的 fontclass 代码：

```html
<link rel="stylesheet" href="./iconfont.css" />
```

第二步：挑选相应图标并获取类名，应用于页面：

```html
<span class="iconfont icon-xxx"></span>
```
