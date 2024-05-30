# 前言

这一章记录的是在实际开发中可能会遇到的问题

# Prisma

## 表结构

1. 如果表中某个字段可能是 null 只需要在类型后面加 `?`
2. 如果某个`String`类型的字符串，他的长度很大,需要加上`@db.LongText `

```prisma

model User {
  id          String     @id @default(uuid())
  name        String
  phone       String     @unique
  password    String
  role        UserRole   @default(NORMAL_USER)
  loginCount  Int        @default(0)
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  lastLoginAt DateTime?  //可能为null的字段
  //注意 在生成token的时候要排除这个token 不然生成的时候会把旧token加密到新token里面
  token       String?    @db.LongText  //长字符串
  status      UserStatus @default(Normal)
  Finance     Finance?
}

```

## 查询

- 查询列表分页
- 模糊查询
- 根据时间范围查询
- 根据某一项排序

```ts
export class UserService {
  constructor(private readonly prisma: PrismaService) {}

  async find(userListDto: UserListDto) {
    const {
      pageNum,
      pageSize,
      lastLoginAt,
      createdAt,
      name,
      phone,
      role,
      status
    } = userListDto;

    const where = {
      // 模糊匹配字符串 如果是name是''，匹配所有name
      name: { contains: name },
      phone: { contains: phone },
      // 如果传的role是 '',设置成undefined，忽略该条件
      role: role ? role : undefined,
      status: status ? status : undefined,
      createdAt:
        createdAt && createdAt.length === 2 && createdAt[1]
          ? {
              gte: dayjs(createdAt[0]).startOf("day").toDate(),
              lt: dayjs(createdAt[1]).endOf("day").toDate()
            }
          : undefined,
      lastLoginAt:
        lastLoginAt && lastLoginAt.length === 2 && lastLoginAt[1]
          ? {
              gte: dayjs(lastLoginAt[0]).startOf("day").toDate(),
              lt: dayjs(lastLoginAt[1]).endOf("day").toDate()
            }
          : undefined
    };
    const skip = (pageNum - 1) * pageSize;
    const take = pageSize;

    const users = await this.prisma.user.findMany({
      where,
      skip,
      take,
      orderBy: {
        lastLoginAt: "desc"
      }
    });
    const total = await this.prisma.user.count({ where });
    return { users, total };
  }
}
```

# 缓存

前提 ：在 app.module.ts 中引入并使用了 `CacheModule`，`CacheInterceptor`

短时间请求同一个 get 请求，不会执行对应路由的 service，而是会直接读取缓存。可能会导致请求到的数据是旧数据。

参考 https://juejin.cn/post/7290795913136799800

```ts
@Module({
  imports: [
    // 如需使用redis https://docs.nestjs.com/techniques/caching#different-stores
    CacheModule.register({ max: 100, isGlobal: true })
  ],
  controllers: [AppController],
  providers: [
    // 他会缓存get请求 如果短时间对一个get请求多次 他会读缓存
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor
    },
    AppService
  ]
})
export class AppModule {}
```

# 文件上传

## 上传代码

1. file.controller.ts

```ts
@ApiBearerAuth()
@Controller("file")
@ApiTags("文件模块")
export class FileController {
  constructor(private readonly fileService: FileService) {}

  @Post("upload")
  @ApiOperation({ summary: "上传文件" })
  @UseInterceptors(FileInterceptor("file"))
  upload(@UploadedFile() fileDto: FileDto) {
    return this.fileService.upload(fileDto);
  }
}
```

2. file.service.ts

```ts
@Injectable()
export class FileService {
  constructor() {}

  async upload(file: any) {
    if (!file) throw new Error("No file uploaded");
    try {
      const { buffer, originalname } = file;
      // 生成要保存的文件路径
      const filePath = path.join(
        __dirname,
        "../../",
        "publicFile",
        originalname
      );
      const writeStream = createWriteStream(filePath);
      writeStream.write(buffer);

      return `publicFile/${originalname}`;
    } catch (error) {
      return { responseCode: 500, message: "文件存储错误" };
    }
  }
}
```

## 提供访问

main.ts 中

```ts
// 文件存放地址，浏览器输入 http://localhost:5600/publicFile/img.jpg 能访问到图片
// 如果同时存在前端和文件，那么都要添加prefix 不然就进前端的路由了
// 访问的有问题就清浏览器缓存
app.useStaticAssets(join(__dirname, "..", "publicFile"), {
  prefix: "/publicFile"
});
```

# 静态资源

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);

// 将静态资源托管到指定目录，前端打包出来的dist目录放到public下
// 前端需设置路由为hash模式（history需要额外配置），前端生产环境接口的baseURl设置为 /
// 如果设置了prefix 前端打包的base也要设置为'/app'
app.useStaticAssets(join(__dirname, "..", "public/dist"), {
  prefix: "/app"
});

// 文件存放地址，浏览器输入 http://localhost:5600/publicFile/img.jpg 能访问到图片
// 如果同时存在前端和文件，那么都要添加prefix 不然就进前端的路由了
// 访问的有问题就清浏览器缓存
app.useStaticAssets(join(__dirname, "..", "publicFile"), {
  prefix: "/publicFile"
});
```

# PM2

在服务器上使用 `PM2` 管理进程

1. 安装

```shell
npm install pm2@latest -g

yarn global add pm2
```

2.启动 npm yarn pnpm 应用

```shell
pm2 start -n demo npm -- run dev

pm2 start -n 进程名称 包管理器 -- 运行脚本
```

3. 启动 nuxt 应用

- 在项目根目录下新建 ecosystem.config.js

```js
module.exports = {
  apps: [
    {
      name: "ProjectName",
      script: "./node_modules/nuxt/bin/nuxt.js",
      args: "start -e production" // pm2执行其实就是 `nuxt start -e production`
    }
  ]
};
```

- 进入项目根目录执行命令

```shell
 pm2 start
```
