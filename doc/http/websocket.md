```typescript
import { ElMessage, ElNotification, ElMessageBox } from "element-plus";
import mitt from "mitt";
import EVENT_TYPE from "@/enum/event-type.enum";

let ws: WebSocket | undefined;

let reconnectCount = 0;
let reSendCount = 0;

export const emitter = mitt<{ msg: any }>();

export const connect = () => {
  if (reconnectCount++ > 10) {
    ws!.close();
    ElMessageBox.confirm("websocket连接失败，点击确认刷新页面！", "警告", {
      confirmButtonText: "OK",
      cancelButtonText: "Cancel",
      type: "warning"
    }).then(() => {
      location.reload();
    });

    return;
  }

  ws = new WebSocket(import.meta.env.VITE_WEBSOCKET_URL);

  ws.onopen = () => {
    reconnectCount = 0;
    ElMessage.success("连接成功");
  };

  ws!.onmessage = (message) => {
    const data = JSON.parse(message.data);

    const { event_type, msg } = data;
    if (event_type === EVENT_TYPE.NOTIFICATION) {
      ElNotification({
        title: data.title,
        message: msg,
        type: data.type
      });
      return;
    }
    if (event_type !== EVENT_TYPE.PING) {
      emitter.emit("msg", JSON.parse(message.data));
    }
  };

  ws.onerror = (err: any) => {
    ElMessage.error(err);
  };

  ws.onclose = () => {
    setTimeout(() => {
      reconnectCount++;
      connect();
    }, 3000);
  };
};

export const send = (data: any) => {
  if (reSendCount > 10) {
    ElMessage.error("请求失败！");
    return;
  }
  switch (ws?.readyState) {
    case 0:
      setTimeout(() => {
        reSendCount++;
        send(data);
      }, 1000);
      break;
    case 1:
      reSendCount = 0;
      ws.send(JSON.stringify(data));
      break;
    default:
      break;
  }
};

export const close = () => {
  ws!.close();
};
```
