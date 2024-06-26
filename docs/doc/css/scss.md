# Sass

## 简介

Sass 是一个 CSS 预处理器。

Sass 是 CSS 扩展语言，可以帮助我们减少 CSS 重复的代码，节省开发时间。

Sass 完全兼容所有版本的 CSS。

## 变量

### 基本使用

Sass 变量可以存储字符串、数字、颜色值、布尔值、列表、null。

> Sass 变量使用`$`符号

scss 代码：

```scss
$myFont: Helvetica, sans-serif;
$myColor: red;
$myFontSize: 18px;
$myWidth: 680px;

body {
  font-family: $myFont;
  font-size: $myFontSize;
  color: $myColor;
}

#container {
  width: $myWidth;
}
```

转化为 css 代码：

```css
body {
  font-family: Helvetica, sans-serif;
  font-size: 18px;
  color: red;
}

#container {
  width: 680px;
}
```

### 作用域

Sass 变量的作用域只能在当前的层级上有效果。

当然 Sass 中我们可以使用 `!global`关键词来设置变量是全局的。

scss 代码：

```scss
$myColor: red;

h1 {
  $myColor: green; // 只在 h1 里头有用，局部作用域
  $mySize: 17px !global; // 全局作用域
  color: $myColor;
}

p {
  color: $myColor;
  font-size: $mySize;
}
```

转化为 Css 代码：

```css
h1 {
  color: green;
}

p {
  color: red;
  font-size: 17px;
}
```

!>注意：所有的全局变量我们一般定义在同一个文件，如：`_globals.scss`，然后我们使用 @include 来包含该文件。

## 插值语句

> 通过 #{} 插值语句可以在选择器或属性名中使用变量：

```scss
$name: foo;
$attr: border;
p.#{$name} {
  #{$attr}-color: blue;
}
```

编译为

```css
p.foo {
  border-color: blue;
}
```

#{} 插值语句也可以在属性值中插入 SassScript，大多数情况下，这样可能还不如使用变量方便，但是使用 #{} 可以避免 Sass 运行运算表达式，直接编译 CSS。

```scss
p {
  $font-size: 12px;
  $line-height: 30px;
  font: #{$font-size}/#{$line-height};
}
```

编译为

```css
p {
  font: 12px/30px;
}
```

## 嵌套规则与属性

### 嵌套规则

scss 代码：

```scss
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }
  li {
    display: inline-block;
  }
  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
```

scss 转化为 Css 代码：

```css
nav ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
nav li {
  display: inline-block;
}
nav a {
  display: block;
  padding: 6px 12px;
  text-decoration: none;
}
```

### 嵌套属性

很多 CSS 属性都有同样的前缀，例如：font-family, font-size 和 font-weight ， text-align, text-transform 和 text-overflow。

在 scss 中，我们可以使用嵌套属性来编写它们：

scss 代码：

```scss
font: {
  family: Helvetica, sans-serif;
  size: 18px;
  weight: bold;
}

text: {
  align: center;
  transform: lowercase;
  overflow: hidden;
}
```

scss 转化为 Css 代码：

```css
font-family: Helvetica, sans-serif;
font-size: 18px;
font-weight: bold;

text-align: center;
text-transform: lowercase;
text-overflow: hidden;
```

### 父选择器&

> 在使用嵌套规则时，父选择器能对于嵌套规则如何解开提供更好的控制。
> scss 代码：

```scss
.box {
  color: blue;
  .min-box {
    &:hover {
      color: red;
    }
  }
}
```

转换为 css 代码：

```css
.box {
  color: blue;
}

.box .min-box:hover {
  color: red;
}
```

!>&在选择器前面时不能有任何的空格，不然就会变成 CSS 选择器中的后代选择器。  
&写在后面的，前面必须要有空格。

## @import 与 Partials

### 导入文件

Sass 可以帮助我们减少 CSS 重复的代码，节省开发时间。

我们可以安装不同的属性来创建一些代码文件，如：变量定义的文件、颜色相关的文件、字体相关的文件等。

> @import 指令可以让我们导入其他文件等内容。

CSS @import 指令在每次调用时，都会创建一个额外的 HTTP 请求。但 Sass `@import` 指令将文件包含在 CSS 中，不需要额外的 HTTP 请求。

!>注意：包含文件时不需要指定文件后缀，Sass 会自动添加后缀 .scss。

此外，你也可以导入 CSS 文件。
导入后我们就可以在主文件中使用导入文件等变量。

以下实例，导入 variables.scss、colors.scss 和 reset.scss 文件。

```scss
@import "variables";
@import "colors";
@import "reset";
```

### Partials

如果你不希望将一个 Sass 的代码文件编译到一个 CSS 文件，你可以在文件名的开头添加一个下划线。这将告诉 Sass 不要将其编译到 CSS 文件。

但是，在导入语句中我们不需要添加下划线。

Sass Partials 文件命名：`_filename.scss`

!>注意：请不要将带下划线与不带下划线的同名文件放置在同一个目录下，比如，\_colors.scss 和 colors.scss 不能同时存在于同一个目录下，否则带下划线的文件将会被忽略。

## @mixin 与 @include

> @mixin 指令允许我们定义一个可以在整个样式表中重复使用的样式。  
> @include 指令可以将混入（mixin）引入到文档中。

### 定义混入

创建一个名为"important-text" 的混入：

```scss
@mixin important-text {
  color: red;
  font-size: 25px;
  font-weight: bold;
  border: 1px solid blue;
}
```

!>注意 Sass 的连接符号 - 与下划线符号 \_ 是相同的，也就是 @mixin important-text { } 与 @mixin important_text { } 是一样的

### 使用混入

@include 指令可用于包含一混入：

```scss
.box {
  @include important-text;
  background-color: green;
}
```

将以上代码转换为 css：

```css
.box {
  color: red;
  font-size: 25px;
  font-weight: bold;
  border: 1px solid blue;
  background-color: green;
}
```

混入中也可以包含混入，如下所示：

```scss
@mixin special-text {
  @include important-text;
  @include link;
  @include special-border;
}
```

### 接收参数的混入

#### 基本使用

scss 混入可以接收参数，如下所示：

```scss
/* 混入接收两个参数 */
@mixin bordered($color, $width) {
  border: $width solid $color;
}

.myArticle {
  @include bordered(blue, 1px); // 调用混入，并传递两个参数
}

.myNotes {
  @include bordered(red, 2px); // 调用混入，并传递两个参数
}
```

将以上代码转换为 css：

```css
.myArticle {
  border: 1px solid blue;
}

.myNotes {
  border: 2px solid red;
}
```

#### 参数的默认值

混入的参数也可以定义默认值，语法格式如下：

```scss
@mixin sexy-border($color blue, $width: 1px) {
  border: {
    color: $color;
    width: $width;
    heigth: 100px;
  }
}
p {
  @include sexy-border(blue);
}
h1 {
  @include sexy-border(blue, 2px);
}
```

#### 可变参数

不能确定一个混入（mixin）或者一个函数（function）使用多少个参数，这时我们就可以使用 `... `来设置可变参数。

```scss
@mixin box-shadow($shadows...) {
  -moz-box-shadow: $shadows;
  -webkit-box-shadow: $shadows;
  box-shadow: $shadows;
}

.shadows {
  @include box-shadow(0px 4px 5px #666, 2px 6px 10px #999);
}
```

以上代码转换为 css：

```css
.shadows {
  -moz-box-shadow: 0px 4px 5px #666, 2px 6px 10px #999;
  -webkit-box-shadow: 0px 4px 5px #666, 2px 6px 10px #999;
  box-shadow: 0px 4px 5px #666, 2px 6px 10px #999;
}
```

## @extend 与继承

> @extend 指令告诉 Sass 一个选择器的样式从另一选择器继承。

如果一个样式与另外一个样式几乎相同，只有少量的区别，则使用 @extend 就显得很有用。

```scss
.button-basic {
  border: none;
  padding: 15px 30px;
  text-align: center;
  font-size: 16px;
  cursor: pointer;
}

.button-report {
  @extend .button-basic;
  background-color: red;
}

.button-submit {
  @extend .button-basic;
  background-color: green;
  color: white;
}
```

将以上代码转换为 CSS 代码，如下所示

```css
.button-basic,
.button-report,
.button-submit {
  border: none;
  padding: 15px 30px;
  text-align: center;
  font-size: 16px;
  cursor: pointer;
}

.button-report {
  background-color: red;
}

.button-submit {
  background-color: green;
  color: white;
}
```

## 函数

详见官方文档 [https://www.sass.hk/skill/sass26.html](https://www.sass.hk/skill/sass26.html)
