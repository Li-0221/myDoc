## login.vue

```vue
<template>
  <div ref="login" class="login-body">
    <div class="login-content">
      <img class="login-logo" src="https://s1.ax1x.com/2022/06/29/jnLR2V.png" alt="" />
      <div class="login-input">
        <div class="login-title">欢迎登录</div>
        <el-form
          ref="formRef"
          :model="loginForm"
          :disabled="store.state.isLoading"
          class="login-form"
        >
          <el-form-item
            prop="phone"
            :rules="[
              {
                required: true,
                message: '请输入账号',
                trigger: 'blur'
              }
            ]"
          >
            <el-input v-model="loginForm.phone" maxlength="11" placeholder="请输入账号">
              <template #prefix>
                <img class="input-icon" src="https://s1.ax1x.com/2022/06/29/jnL280.png" alt="" />
              </template>
            </el-input>
          </el-form-item>
          <el-form-item
            prop="password"
            :rules="[
              {
                required: true,
                message: '请输入密码',
                trigger: 'blur'
              }
            ]"
          >
            <el-input v-model="loginForm.password" type="password" placeholder="请输入密码">
              <template #prefix>
                <img class="input-icon" src="https://s1.ax1x.com/2022/06/29/jnLgCq.png" alt="" />
              </template>
            </el-input>
          </el-form-item>
          <div class="codeFlex">
            <el-form-item
              prop="code"
              :rules="[
                {
                  required: true,
                  message: '请输入验证码',
                  trigger: 'blur'
                },
                { validator: checkCode, trigger: 'blur' }
              ]"
            >
              <el-input
                v-model="loginForm.code"
                maxlength="4"
                placeholder="请输入验证码"
                @keyup.enter="submitForm(formRef)"
              >
                <template #prefix>
                  <img class="input-icon" src="https://s1.ax1x.com/2022/06/29/jnL65n.png" alt="" />
                </template>
              </el-input>
            </el-form-item>
            <div class="code_box" @click="changeCode">
              <canvas id="modifyCanvas" ref="modifyCanvas" width="106" height="43"></canvas>
            </div>
          </div>
          <el-form-item>
            <el-button type="success" :loading="store.state.isLoading" @click="submitForm(formRef)"
              >登录
            </el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from "vue";
import type { ElForm } from "element-plus";
import { useStore } from "vuex";
type FormInstance = InstanceType<typeof ElForm>;
const formRef = ref<FormInstance>();
const verificationCode = ref<string[]>([]);

const loginForm = reactive<{
  phone: string;
  password: string;
  code: string;
}>({
  phone: "",
  password: "",
  code: ""
});
const store: any = useStore();
//刷新验证码
const changeCode = async () => {
  draw(verificationCode.value);
  localStorage.setItem("time", JSON.stringify(new Date().getTime()));
};
const checkCode = (rule: any, value: any, callback: any) => {
  const oldtime: string | null = localStorage.getItem("time");
  if (new Date().getTime() > Number(oldtime) + 5 * 60 * 1000) {
    callback(new Error("验证码已过期！请重新获取"));
    return false;
  }
  if (value.toLowerCase() !== verificationCode.value.join("")) {
    callback(new Error("验证码输入错误！"));
  } else {
    callback();
  }
};
//生成并渲染出验证码图形
const draw = (show_num: string[] = ["4", "r", "9", "s"]) => {
  let canvas = document.getElementById("modifyCanvas") as HTMLCanvasElement; //获取到canvas的对象
  let canvas_width = canvas.width;
  let canvas_height = canvas.height;
  let context = canvas.getContext("2d") as CanvasRenderingContext2D; //获取到canvas画图的环境，演员表演的舞台
  canvas.width = canvas_width;
  canvas.height = canvas_height;
  let sCode =
    "a,b,c,d,e,f,g,h,i,j,k,m,n,p,q,r,s,t,u,v,w,x,y,z,A,B,C,E,F,G,H,J,K,L,M,N,P,Q,R,S,T,W,X,Y,Z,1,2,3,4,5,6,7,8,9,0";
  let aCode = sCode.split(",");
  let aLength = aCode.length; //获取到数组的长度
  for (let i = 0; i < 4; i++) {
    //这里的for循环可以控制验证码位数（如果想显示6位数，4改成6即可）
    let j = Math.floor(Math.random() * aLength); //获取到随机的索引值
    // let deg = Math.random() * 30 * Math.PI / 180;//产生0~30之间的随机弧度
    let deg = Math.random() - 0.5; //产生一个随机弧度
    let txt = aCode[j]; //得到随机的一个内容
    show_num[i] = txt.toLowerCase(); //show_num得到的是verificationCode数组的地址，在这给verificationCode赋值了
    let x = 4 + i * 20; //文字在canvas上的x坐标
    let y = 30 + Math.random() * 8; //文字在canvas上的y坐标
    context.font = "bold 23px 微软雅黑";
    context.translate(x, y);
    context.rotate(deg);
    context.fillStyle = randomColor();
    context.fillText(txt, 0, 0);
    context.rotate(-deg);
    context.translate(-x, -y);
  }
  for (let i = 0; i <= 30; i++) {
    //验证码上显示小点
    context.strokeStyle = randomColor();
    context.beginPath();
    let x = Math.random() * canvas_width;
    let y = Math.random() * canvas_height;
    context.moveTo(x, y);
    context.lineTo(x + 1, y + 1);
    context.stroke();
  }
  for (var i = 0; i <= 5; i++) {
    //验证码上显示线条
    context.strokeStyle = randomColor();
    context.beginPath();
    context.moveTo(Math.random() * canvas_width, Math.random() * canvas_height);
    context.lineTo(Math.random() * canvas_width, Math.random() * canvas_height);
    context.stroke();
  }
};
//得到随机的颜色值
const randomColor = (): string => {
  let r = Math.floor(Math.random() * 256);
  let g = Math.floor(Math.random() * 256);
  let b = Math.floor(Math.random() * 256);
  return "rgb(" + r + "," + g + "," + b + ")";
};
// 登录
const submitForm = (formEl: FormInstance | undefined) => {
  console.log(formEl);

  if (!formEl) return;

  formEl.validate(valid => {
    if (valid) {
      // 调用登录接口
    }
  });
};
onMounted(() => {
  changeCode();
  localStorage.setItem("time", JSON.stringify(new Date().getTime()));
});
</script>
<style lang="less" scoped>
div {
  box-sizing: border-box;
}

.login-body {
  width: 100%;
  height: 100%;
  overflow: auto;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  background: url("https://s1.ax1x.com/2022/06/29/jnLWvT.md.png") no-repeat;
  background-size: cover;
  min-width: 800px;
  min-height: 500px;

  .login-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 1200px;
    height: 640px;
    padding: 119px 100px 96px 100px;
    flex-shrink: 0;
    background-color: #ffffff;
    box-shadow: 0px 3px 14px 0px rgba(0, 42, 75, 0.39);
    border-radius: 18px;

    .login-logo {
      width: 449px;
      height: 425px;
      margin-right: 180px;
    }

    .login-input {
      .login-title {
        width: 364px;
        font-size: 47px;
        font-style: italic;
        font-stretch: normal;
        margin-bottom: 48px;
        letter-spacing: 4px;
        color: #45a0b2;
        display: flex;
        align-items: center;
        justify-content: center;
      }

      .login-form {
        width: 364px;

        .el-form-item {
          height: 53px;

          .el-input {
            height: 43px;

            :deep(.el-input__inner) {
              height: 43px;
              border-radius: 2px;
            }

            :deep(.el-input__prefix) {
              display: flex;
              align-items: center;
              justify-content: center;
            }

            .input-icon {
              width: 16px;
              height: 22px;
            }
          }

          .el-button--success {
            width: 364px;
            height: 44px;
            background-color: #45a0b2;
            border-radius: 2px;
            font-size: 16px;
            border: none;
          }
        }

        .codeFlex {
          display: flex;
          width: 364px;
          align-items: center;
          justify-content: space-between;

          :deep(.el-form-item) {
            width: 100%;
            margin-right: 10px;
          }

          .code_box {
            cursor: pointer;
            height: 43px;
            width: 106px;
            border: solid 1px #b2c7df;
            margin-bottom: 20px;
          }
        }
      }
    }
  }
}
</style>
<style>
/* 去除浏览器input自动填充的默认样式 */
input:-internal-autofill-selected {
    /* 这里#fff是底色 */
    -webkit-box-shadow: 0 0 0px 1000px #fff inset;
    -webkit-text-fill-color: #333;
}
</style>
```
