# JS

## 1.变量

> 绝大部分 const，少用 let，不用 var

## 2.函数

```javascript
n => n + 1;
(a, b) => a + b;
(n1, n2, n3) => [n1, n2, n3];
(name, age) => ({ name, age }); //返回对象时，需要加上括号
```

## 3.高阶函数

```javascript
const f = a => b => a + b;
f(1)(2);

//函数 f2 的参数 fn 为函数
const f2 = fn => fn(1, 2);
f2((a, b) => a + b);
```

## 4.扩展操作符

```javascript
const user = { name: "lili", age: 18 };
const user2 = { ...user, age: 19 }; //age 被覆盖为 19

//定义一个 call 方法，第一个参数是函数，剩下的参数用 args 代替
//call 方法的返回值是 fn(...args)，fn 就是调用 call 方法的第一个参数（函数）
const call = (fn, ...args) => fn(...args);
call((a, b) => a + b, 1, 2);
```

# TS

## 1.类型声明

```typescript
//type 可以声明单个类型，也可以声明具体的值
//一般用 type 声明单个变量
type age = number;
type Name = "lili";

//interface 只能用于声明对象，也可以声明对象的属性的具体值
interface User {
  name: string;
  age?: number;
  sex: "男" | "女";
}
```

```typescript
type Name = string;
// type Name = "lili";
const str: Name = "lili";

type Id = string | number;
const id1: Id = "123";
const id2: Id = 123;

type Dir = "left" | "right" | "top" | "bottom";
const dir: Dir = "left";

type User = {
  name?: string;
  readonly id: string;
};
```

## 2.联合类型

```typescript
type A = {
  t: "a";
  specialA: string;
};
type B = {
  t: "a";
  specialB: number;
};
type C = A | B;

const c: C = {
  t: "a",
  specialA: "specialA"
};

//type D 错误，因为类型 A，B 有相同的键 t，t 不能为同时满足 a 和 b 的值
//当类型 A，B 没有相同的键时可以使用 &
//type D = A & B;
```

## 3.接口继承

```typescript
type Dog = "Dog";
type Cat = "Cat";
interface User {
  readonly id: string | number;
  name: string;
  age?: number;
}

//继承
interface UserWithDog extends User {
  dog: Dog;
}

const u1: UserWithDog = {
  name: "lili",
  age: 18,
  id: 1,
  dog: "Dog"
};

//泛型
interface UserWithPet<T> extends User {
  pet: T;
}
const u2: UserWithPet<Cat> = {
  name: "lili",
  age: 18,
  id: 1,
  pet: "Cat"
};
```
