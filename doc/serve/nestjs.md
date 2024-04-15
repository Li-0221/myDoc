# 快速生成

1. 命令 `nest g resource user`

2. 选择 REST API

3. 选择 curd

# REST API 规范

## 传统接口

```
http://localhost:8080/api/get_list?id=1

http://localhost:8080/api/delete_list?id=1

http://localhost:8080/api/update_list?id=1
```

## REST API 接口

一个接口就会完成 增删改差 他是通过不同的请求方式来区分的

查询 GET
提交 POST
更新 PUT PATCH
删除 DELETE

```
http://localhost:8080/api/get_list/1
```

# 状态码

200 OK

304 Not Modified 协商缓存了

400 Bad Request 参数错误

401 Unauthorized token 错误

403 Forbidden referer origin 验证失败

404 Not Found 接口不存在

500 Internal Server Error 服务端错误

502 Bad Gateway 上游接口有问题或者服务器问题

# Controller Request

作用：获取前端传过来的参数

| 装饰器                  | 对应值                          | 说明                                       |
| ----------------------- | ------------------------------- | ------------------------------------------ |
| @Request() / @Req       | req                             | 请求对象                                   |
| @Response() / @Res      | res                             | 响应对象                                   |
| @Next()                 | next                            |                                            |
| @Session()              |                                 | 服务端的 session 对象 （服务端的内存中）   |
| @Param(key?: string)    | req.params / req.params[key]    | 路由斜杠后面的参数对象 或 对象里的某个属性 |
| @Body(key?: string)     | req.body / req.body[key]        | 请求体 或 请求体里面的某个属性             |
| @Query(key?: string)    | req.query / req.query[key]      | 查询参数 或 查询参数里面的某个属性         |
| @Headers(name?: string) | req.headers / req.headers[name] | 请求头 或 请求头里面的某个属性             |

# 中间件

在处理路由程序之前调用的函数叫中间件

- 执行任何代码
- 对请求和响应对象进行更改
- 结束请求-响应循环
- 调用下一个中间件函数
- 如果当前中间件函数没有结束请求-响应循环，则必须调用 next() 方法将控制权传递给下一个中间件函数。否则，请求将被挂起。

## 定义中间件

logger.middleware.ts

```typescript
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response, NextFunction } from "express";

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`Request: ${req.method} ${req.url}`);
    next();
  }
}
```

app.module.ts

```typescript
import { Module, NestModule, MiddlewareConsumer } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { UserModule } from "./user/user.module";
import LoggerMiddleware from "./middleware/logger.middleware";

@Module({
  imports: [UserModule],
  controllers: [AppController],
  providers: [AppService]
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    //添加全局中间件
    consumer.apply(LoggerMiddleware).forRoutes("*");
    // consumer.apply(CorsMiddleware).forRoutes('*');

    //添加指定路由中间件  除了login和code路由，其他路由都需要验证
    // consumer
    //   .apply(AuthMiddleware)
    //   .exclude(
    //     { path: 'login', method: RequestMethod.ALL },
    //     { path: 'code', method: RequestMethod.ALL },
    //   )
    //   .forRoutes('');
  }
}
```

# 下载文件

- 文件下载功能更适合在一个独立的模块中处理，例如一个名为 file 的模块。

https://blog.csdn.net/qq1195566313/article/details/126880230?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168439504516800217252102%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=168439504516800217252102&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-13-126880230-null-null.blog_rank_default&utm_term=nest&spm=1018.2226.3001.4450

## 直接下载

## 文件流下载

# 响应拦截

我们想给他返回一个标准的 json 格式 就要给我们的数据做一个全局 format

```json

{
  data, //数据
  status:0,
  message:"成功",
  success:true
}
```

## 格式化数据

src/common/response

```typescript
import { Injectable, NestInterceptor, CallHandler } from "@nestjs/common";
import { map } from "rxjs/operators";
import { Observable } from "rxjs";

interface data<T> {
  data: T;
}

@Injectable()
export class Response<T = any> implements NestInterceptor {
  intercept(context, next: CallHandler): Observable<data<T>> {
    return next.handle().pipe(
      map((data) => {
        return {
          data,
          status: 0,
          success: true
        };
      })
    );
  }
}
```

## 全局注册

src/main.ts

```typescript
import { Response } from "./common/response";
//...
app.useGlobalInterceptors(new Response());
```

> 在 controller 里面就可以直接返回的数据会被放到 data 里面

# 异常过滤器

## 定义异常过滤器

src/common/filter

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException
} from "@nestjs/common";

import { Request, Response } from "express";

@Catch(HttpException)
export class HttpFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest<Request>();
    const response = ctx.getResponse<Response>();

    const status = exception.getStatus();

    response.status(status).json({
      data: exception.message,
      time: new Date().getTime(),
      success: false,
      path: request.url,
      status
    });
  }
}
```

## 全局注册

```typescript
import { HttpFilter } from "./common/filter";
// ...
app.useGlobalFilters(new HttpFilter());
```

> 接口异常时就会走这个过滤器

# 管道

作用：

1.转换，可以将前端传入的数据转成成我们需要的数据

2.验证 类似于前端的 rules 配置验证规则

ValidationPipe  
ParseIntPipe  
ParseFloatPipe  
ParseBoolPipe  
ParseArrayPipe  
ParseUUIDPipe  
ParseEnumPipe  
DefaultValuePipe

## 转换

user.controller.ts

```typescript
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    console.log(typeof id); //number
    return this.userService.findOne(+id);
  }

```

## 验证 dto

user.service.ts

```typescript
import { Controller, Post, Body } from "@nestjs/common";
import { UserService } from "./user.service";
import { CreateUserDto } from "./dto/create-user.dto";

@Controller("user")
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  // 如果传入的参数不是 CreateUserDto 中装饰器类型的，就会报400
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }
}
```

create-user.dto.ts

```typescript
import { IsString, IsInt } from "class-validator";

export class CreateUserDto {
  // 装饰器验证传入的字段
  @IsString()
  name: string;

  @IsInt()
  age: number;
}
```

# 守卫

守卫在每个中间件之后执行，但在任何拦截器或管道之前执行

主要是用于处理路由权限，如某个路由需要 admin 权限才能访问，如果他是 user，则返回 403

但是守卫的作用，目前可以通过前端控制，后面需要再学

https://www.bilibili.com/video/BV1NG41187Bs?p=21&vd_source=f0dd6fca300bb22c9819d00a25af348a
https://www.bilibili.com/video/BV1NG41187Bs?p=22&vd_source=f0dd6fca300bb22c9819d00a25af348a

# swagger

npm install @nestjs/swagger swagger-ui-express

## 引入

- main.ts

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  //swagger
  const options = new DocumentBuilder()
    .addBearerAuth()
    .setTitle("接口文档标题")
    .setDescription("描述，。。。")
    .setVersion("1")
    .build();
  const document = SwaggerModule.createDocument(app, options);
  // localhost:5700/api-docs 访问接口文档
  SwaggerModule.setup("/api-docs", app, document);
  await app.listen(5700);
}
```

## 常用装饰器

| 装饰器        | 作用                             |
| ------------- | -------------------------------- |
| ApiBearerAuth | 需要带 token，请求时会带上 token |
| ApiOperation  | 接口描述                         |
| ApiProperty   | post 请求用 结合 dto 配置        |
| ApiParam      | param 描述                       |
| ApiQuery      | query 参数描述                   |
| ApiTags       | 标签分组                         |

## 举例

- user.controller.ts

```typescript
import { CreateUserDto } from "./dto/create-user.dto";
import {
  ApiBearerAuth,
  ApiOperation,
  ApiParam,
  ApiQuery,
  ApiTags
} from "@nestjs/swagger";

@Controller("user")
@ApiBearerAuth()
@ApiTags("用户模块")
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  @Get()
  @ApiOperation({ summary: "获取所有用户", description: "描述xxxxx" })
  findAll() {
    return this.userService.findAll();
  }

  @Get(":id")
  @ApiParam({
    name: "id",
    description: "用户id",
    required: true,
    type: "number"
  })
  // @ApiQuery({
  //   name: 'id',
  //   description: '用户id',
  //   required: true,
  //   type: 'number',
  // })
  findOne(@Param() id: number) {
    return this.userService.findOne(+id);
  }
}
```

- create-user.dto.ts

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { IsString, IsInt } from "class-validator";

export class CreateUserDto {
  @IsString()
  @ApiProperty({ description: "用户名", example: "user1", required: false })
  name: string;

  @IsInt()
  @ApiProperty({ description: "年龄", example: 18 })
  age: number;
}
```

# Prisma

## 安装

```
npm install prisma --save-dev

npx prisma init --datasource-provider mysql
```

## 建模

prisma/schema.prisma

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  // @relation(fields: [authorId], references: [id])
  // 表示Post中的authorId字段关联到User表中的id字段
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

> `@relation(fields: [authorId], references: [id])`
> 表示 Post 中的 authorId 字段关联到 User 表中的 id 字段

新增 Post 的时候，

```typescript
import { prisma } from "../../prisma/prisma.service";

const createPost = () => {
  try {
    const newPost = await prisma.post.create({
      data: {
        title: "Post标题",
        content: "Post内容",
        published: false,
        author: {
          connect: {
            id: 1 // 设置对应的作者ID
          }
        }
      }
    });

    console.log("创建的Post:", newPost);
  } catch (error) {
    console.error("创建Post时发生错误:", error);
  } finally {
    await prisma.$disconnect();
  }
};

createPost();
```

### 常见的类型

| 类型     | 含义                                             |
| -------- | ------------------------------------------------ |
| String   | 字符串类型，用于存储文本数据。                   |
| Int      | 整数类型，用于存储整数值。                       |
| Float    | 浮点数类型，用于存储带有小数点的数值。           |
| Boolean  | 布尔类型，用于存储 true 或 false 值。            |
| DateTime | 日期时间类型，用于存储日期和时间信息。           |
| Json     | JSON 类型，用于存储结构化的 JSON 数据。          |
| Enum     | 枚举类型，用于存储预定义的枚举值列表。           |
| Bytes    | 字节数组类型，用于存储二进制数据。               |
| BigInt   | 非常大的整数类型，用于存储超出整数范围的大数字。 |
| Decimal  | 十进制数类型，用于存储精确的小数值。             |
| Map      | 键值对映射类型，用于存储任意键和值的映射。       |

### 常见的元数据

| 类型              | 含义                                                                                                           |
| ----------------- | -------------------------------------------------------------------------------------------------------------- |
| @default(value)   | 指定字段的默认值。value 是默认值的具体取值，可以是一个固定的值或一个 MySQL 函数。例如：@default("John Doe")。  |
| @unique           | 指定字段的值在表中是唯一的，不能重复。                                                                         |
| @id               | 指定字段为主键（唯一标识一条记录的字段）。                                                                     |
| @relation         | 指定字段之间的关系（关联关系）。用于定义模型之间的关联，如一对一关系、一对多关系等。                           |
| @updatedAt        | 指定字段在更新时自动设置为当前时间戳，用于跟踪最后更新的时间。可以使用 MySQL 的 CURRENT_TIMESTAMP 函数来实现。 |
| @createdAt        | 指定字段在创建时自动设置为当前时间戳，用于跟踪创建时间。同样可以使用 CURRENT_TIMESTAMP 函数来实现。            |
| @map              | 指定字段在数据库中的实际列名或表名。用于将模型字段映射到特定的 MySQL 列或表。                                  |
| @charset(value)   | 指定字段的字符集。value 是字符集的名称，如 utf8mb4。                                                           |
| @collation(value) | 指定字段的排序规则。value 是排序规则的名称，如 utf8mb4_unicode_ci。                                            |

### 常见的@default 值

@default(now()): 设置默认值为当前日期和时间。
@default(current_timestamp()): 设置默认值为当前时间戳。
@default(current_date()): 设置默认值为当前日期。
@default(current_time()): 设置默认值为当前时间。

@default(autoincrement()): 设置默认值为自动递增的值。用于将字段设置为自动增加的主键。

@default(uuid()): 设置默认值为生成的 UUID（通用唯一标识符）。

@default(0): 设置默认值为整数 0。
@default(0.0): 设置默认值为浮点数 0.0。
@default(""): 设置默认值为空字符串。

## 生成表

```
npx prisma migrate dev --name init
```

## 读取数据

```typescript
import { Injectable } from "@nestjs/common";
import { prisma } from "../../prisma/prisma.service";

@Injectable()
export class UserService {
  async findAll() {
    // 获取user表里面的所有数据 类似于 select * from user
    const data = await prisma.user.findMany();
    return data;
  }
}
```

## 增加数据

!>要使用 async await，否则插入不成功

```typescript
import { prisma } from "../../prisma/prisma.service";

async function createUser() {
  try {
    const createdUser = await prisma.user.create({
      data: {
        name: "Alice",
        email: "222@qq.com"
      }
    });
    console.log("插入的用户数据:", createdUser);
  } catch (error) {
    console.error("发生错误:", error);
  } finally {
    await prisma.$disconnect();
  }
}

createUser();
```
