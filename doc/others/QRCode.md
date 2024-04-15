# 介绍

QRCode.js is javascript library for making QRCode. QRCode.js supports Cross-browser with HTML5 Canvas and table tag in DOM. QRCode.js has no dependencies.

# 使用

## 安装

```powershell
npm i qrcodejs2
```

## 基础使用

```html
<div id="qrcode"></div>
<script type="text/javascript">
  new QRCode(document.getElementById("qrcode"), "http://jindo.dev.naver.com/collie");
</script>
```

## 配置项使用

|     参数     |        默认值         |                                                  说明                                                  |                   备注                    |
| :----------: | :-------------------: | :----------------------------------------------------------------------------------------------------: | :---------------------------------------: |
|     text     |        string         | 二维码内容字符串 如果是 url 的话，为了微信和 QQ 可以识别，连接中的中文使用 encodeURIComponent 进行编码 |
|    width     |          256          |                                                图像宽度                                                |          单位像素（百分比不行）           |
|    height    |          256          |                                                图像高度                                                |          单位像素（百分比不行）           |
|  colorDark   |        '#000'         |                                              二维码前景色                                              | 英文\十六进制\rgb\rgba\transparent 都可以 |
|  colorLight  |        ‘#fff’         |                                              二维码背景色                                              | 英文\十六进制\rgb\rgba\transparent 都可以 |
| correctLevel | QRCode.CorrectLevel.L |                                                容错级别                                                |             由低到高 L M Q H              |

```html
<div id="qrcode"></div>
<script type="text/javascript">
  var qrcode = new QRCode(document.getElementById("qrcode"), {
    text: "http://jindo.dev.naver.com/collie",
    width: 128,
    height: 128,
    colorDark: "#000000",
    colorLight: "#ffffff",
    correctLevel: QRCode.CorrectLevel.H
  });

  qrcode.clear(); // clear the code.
  qrcode.makeCode("http://naver.com"); // make another code.
</script>
```

## 常见问题

### 长字符串显示模糊问题

原因：显示模糊的问题，是 canvas 的问题。由于字符串比较长，尤其是需要传一个连接地址，后面还加一些参数的时候，就会加大二维码的像素复杂度，而本身 canvas 在绘制的时候，就一直有像素模糊的问题，尤其是在手机上的时候

解决：先将生成的二维码进行倍数扩大，然后在 css 上面固定其显示宽高，这样就可以扩大显示像素精度。

js 放大

```javascript
var qrcode = new QRCode(document.getElementById("qrcode"), {
  text: "http://www.baidu.com",
  width: 128 * 5, // 相应宽高扩大5倍
  height: 128 * 5,
  colorDark: "#000",
  colorLight: "#fff",
  correctLevel: QRCode.CorrectLevel.H
});
```

css 固定宽高

```scss
#qrcode-out {
  width: 128px;
  height: 128px;
  canvas,
  img {
    width: 100%;
    height: 100%;
  }
}
```

### 因为 url 太长，导致二维码加载报错

一般都是容错率设置为最高导致的，此时把容错率调低一级便可以
`correctLevel : QRCode.CorrectLevel.Q`
