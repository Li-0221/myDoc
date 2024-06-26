```vue
<template><div ref="xterm" class="w-full h-full" /></template>

<script lang="ts" setup>
import { Terminal } from "xterm";
import { FitAddon } from "xterm-addon-fit";
import "xterm/css/xterm.css";
import { debounce } from "lodash-es";
// websocket发送消息和接收消息的方法
import { send, emitter } from "@/service/websocket";
import SOCKET_EVENT from "@/enum/socket-event.enum";

const xterm = ref<HTMLDivElement>();
const route = useRoute();

const term = new Terminal({
  allowProposedApi: true,
  fontSize: 18,
  cursorBlink: true,
  disableStdin: false
});
const fitAddon = new FitAddon();

const fitScreen = () => {
  fitAddon.fit();
  // websocket 发送消息，告知xterm的尺寸
  send({ event_type: "resize", input: { cols: term.cols, rows: term.rows } });
};

const init = () => {
  term.open(xterm.value as HTMLDivElement);
  term.loadAddon(fitAddon);
  term.focus();
  term.resize(50, 50);
  fitScreen();
  window.addEventListener("resize", debounce(fitScreen, 50));
  // 每按一个字符触发一次
  term.onData((data: string) => {
    sendCMD(data);
  });
};

// websocket 发送在xterm中输入的命令
const sendCMD = (cmd?: string) => {
  send({ event_type: "cmd", input: cmd || "\r" });
};

emitter.on("msg", (res: any) => {
  const { event_type, out } = res;

  if (event_type === "cmd_res" && res.pty_id === route.query.pty_id) {
    term.write(out);
  }
});

onMounted(() => {
  init();
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", debounce(fitScreen, 50));
  fitAddon.dispose();
  term.dispose();
});
</script>
```
