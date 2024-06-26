```typescript
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import path from "path";
//@ts-ignore
import viteCompression from "vite-plugin-compression";

export default () =>
  defineConfig({
    // 开发或生产环境服务的公共基础路径
    // 根据不同的环境设置不同路径 base: mode === 'development' ? './' : '/app'
    base: "./",
    // 需要使用的插件列表
    plugins: [
      vue(),
      // gzip压缩 生产环境生成 .gz 文件
      viteCompression({
        verbose: true,
        disable: false,
        threshold: 10240,
        algorithm: "gzip",
        ext: ".gz"
      })
    ],
    // 强制预构建插件包
    optimizeDeps: {
      // 检测需要预构建的依赖项
      entries: [],
      // 默认情况下，不在 node_modules 中的，链接的包不会预构建
      include: ["axios"]
      // 排除在优化之外的包
      // exclude: ['your-package-name']
    },
    // 静态资源服务的文件夹
    publicDir: "public",
    // 静态资源处理
    // assetsInclude: '',
    // 控制台输出的级别 info 、warn、error、silent
    logLevel: "info",
    // 设为false 可以避免 vite 清屏而错过在终端中打印某些关键信息
    clearScreen: true,
    resolve: {
      // 配置别名
      alias: {
        "@": path.resolve(__dirname, "src")
      }
      // 导入时想要省略的扩展名列表，建议不写
      // 不建议忽略自定义导入类型的扩展名（例如：.vue），因为它会影响 IDE 和类型支持。
      // extensions: []
    },
    css: {
      // 选项将被传递给 postcss-modules
      modules: {},
      // postCss 配置
      postcss: {},
      // 指定传递给 css 预处理器的选项，相当于在main.ts里面引入了
      // 引入的文件最好只写 (scss) 变量，不要写css变量
      preprocessorOptions: {
        // 把文件 @/styles/variables.scss  交给scss处理器，并引入到全局css
        // scss: {
        //     additionalData: `@import '@/styles/variables.scss';`
        // }
      }
    },
    json: {
      // 是否支持从 .json 文件中进行按名导入
      namedExports: true,
      // 若设置为 true 导入的json会被转为 export default JSON.parse("..") 会比转译成对象字面量性能更好
      stringify: false
    },
    // 本地运行配置
    server: {
      host: "localhost",
      https: false, // 是否启用 https
      cors: true, // 为开发服务器配置 CORS , 默认启用并允许任何源
      open: true, // 服务启动时自动在浏览器中打开应用
      port: "9000", // 运行的端口
      strictPort: false // 设为true时端口被占用则直接退出，不会尝试下一个可用端口
      // 反向代理配置
      // proxy: {
      //     '/api': {
      //         target: 'https://xxxx.com/',
      //         changeOrigin: true,
      //         rewrite: (path) => path.replace(/^\/api/, '')
      //     }
      // }
    },
    // 打包配置
    build: {
      // 浏览器兼容性  "esnext"|"modules"
      target: "modules",
      // 指定输出路径
      outDir: "dist",
      // 生成静态资源的存放路径
      assetsDir: "assets",
      // 小于此阈值的导入或引用资源将内联为 base64 编码，以避免额外的 http 请求。设置为 0 可以完全禁用此项
      assetsInlineLimit: 4096,
      // 启用/禁用 CSS 代码拆分
      cssCodeSplit: true,
      // 当设置为 true，构建后将会生成 manifest.json 文件
      manifest: false,
      // 设置为 false 来禁用将构建后的文件写入磁盘
      write: true,
      // 默认情况下，若 outDir 在 root 目录下，则 Vite 会在构建时清空该目录。
      emptyOutDir: true,
      // 启用/禁用 brotli 压缩大小报告
      brotliSize: true,
      // chunk 大小警告的限制
      chunkSizeWarningLimit: 500,
      rollupOptions: {
        // 将打包出来超过chunkSizeWarningLimit的文件，拆分成多个小文件
        output: {
          manualChunks(id) {
            if (id.includes("node_modules")) {
              return id
                .toString()
                .split("node_modules/")[1]
                .split("/")[0]
                .toString();
            }
          }
        }
      },
      terserOptions: {
        //去除 console debugger
        compress: {
          drop_console: true,
          drop_debugger: true
        },
        // 去掉注释内容
        output: {
          comments: true
        }
      }
    }
  });
```
