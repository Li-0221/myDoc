## 下载

```powershell
npm install clipboard --save
```

## 新增工具方法 clipboard.js

```javascript
import Vue from "vue";
import Clipboard from "clipboard";
function clipboardSuccess() {
  Vue.prototype.$message({
    message: "Copy successfully",
    type: "success",
    duration: 1500
  });
}
function clipboardError() {
  Vue.prototype.$message({
    message: "Copy failed",
    type: "error"
  });
}
export default function handleClipboard(text, event) {
  const clipboard = new Clipboard(event.target, {
    text: () => text
  });
  clipboard.on("success", () => {
    clipboardSuccess();
    clipboard.destroy();
  });
  clipboard.on("error", () => {
    clipboardError();
    clipboard.destroy();
  });
  clipboard.onClick(event);
}
```

## html 内容

```html
<el-input
  v-model="inputData"
  placeholder="Please input"
  style="width:400px;max-width:100%;"
/>
<el-button @click="handleCopy(inputData,$event)">copy</el-button>
```

## script 内容

```javascript
import clip from '@/utils/clipboard' // use clipboard directly
 data() {
   return {
     inputData: 'https://github.com/PanJiaChen/vue-element-admin'
    }
  },
methods: {
handleCopy(text, event) {
clip(text, event)
}
}
```
