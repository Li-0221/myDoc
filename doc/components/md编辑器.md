https://imzbf.github.io/md-editor-v3/en-US/index

```vue
<template>
  <div>
    <MdEditor v-model="text" theme="light" :toolbars="toolbars">
      <template #defToolbars>
        <Mark />
        <Emoji />
        <ExportPDF :model-value="text" height="700px" />
      </template>
    </MdEditor>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { MdEditor } from "md-editor-v3";
import type { ToolbarNames } from "md-editor-v3";
import { Emoji, Mark, ExportPDF } from "@vavt/v3-extension";
import "md-editor-v3/lib/style.css";
import "@vavt/v3-extension/lib/asset/style.css";

const toolbars: ToolbarNames[] = [
  "bold",
  "underline",
  "italic",
  "strikeThrough",
  "-",
  "title",
  "sub",
  "sup",
  "quote",
  "unorderedList",
  "orderedList",
  "task",
  "-",
  "codeRow",
  "code",
  "link",
  "image",
  "table",
  "mermaid",
  "katex",
  0,
  1,
  2,
  "-",
  "revoke",
  "next",
  "save",
  "=",
  "prettier",
  "pageFullscreen",
  "fullscreen",
  "preview",
  "previewOnly",
  "htmlPreview",
  "catalog"
];

const text = ref("Hello Editor!");
</script>

<style>
.md-editor * {
  box-sizing: content-box;
}
</style>
```
