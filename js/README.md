## 1. 数据类型检查

### 1.1 typeof

- 直接在 计算机底层基于数据类型的值(二进制)进行检测
- `typeof null object` 对象存储在计算机 都是以 000 开头 null 也是 000 开头
- `typeof` 普通对象/数组对象/正则对象/日期对象 `object`

### 1.2 instanceof 检测当前实例是否属于这个类

- 底层机制: 只要当前类出现在实例的原型链上 结果都为 `true`
- 由于我们可以肆意的修改原型的指向 所以判断的结果是不准确的
- 不能检测基本类型

```js
/** 自定义 instanceof
 * @params example 实例
 * @params classFunc 类
 *
 * */
function instance_of(example, classFunc) {
  let classFuncPrototype = classFunc.prototype,
    proto = Object.getPrototypeOf(example); // example.__proto__
  while (true) {
    // 原型链顶端为 null 说明没有找到类
    if (proto == null) return false;
    // 在原型链上 找到了类
    if (proto == classFuncPrototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
}
```

### 1.3 constructor

- 同样原型的 `constructor` 可以更该 判断的结果也是不准确的

```js
let arr = [];
console.log(arr.constructor == Array);
console.log(arr.constructor == RegExp);
console.log(arr.constructor == Object);
Numbet.prototype.constructor = "AA";
let n = 1;
console.log(n.constructor == Number);
```

### 1.4 Object.prototype.toString.call([value])

- 标准的判断数据类型 `Object.prototype.toString` 不是转为字符串 而是返回当前实例的所属类的信息
- `toString` 方法执行 `this` 是谁 就是检测的它所属类的信息
- 只要把 `Object.prototype.toString` 执行 让它里面的 `this` 变为要检测的类型 那就能返回当前实例的所属类的信息

```js
(function () {
  /** jquery 封装的 判断数据类型
   *
   * */

  const class2type = {};
  const toString = class2type.toString; //Object.prototype.toString
  // 设定数据类型的映射表
  [
    "Boolean",
    "Number",
    "String",
    "Function",
    "Array",
    "Date",
    "RegExp",
    "Object",
    "Error",
    "Symbol",
  ].forEach((name) => {
    class2type[`[object ${name}]`] = name.toLowerCase();
  });

  function toType(obj) {
    if (obj == null) {
      return obj + "";
    }
    return typeof obj === "object" || typeof obj === "function"
      ? class2type[toString.call(obj)] || "object"
      : typeof obj;
  }
  window.toType = toType;
})();
```

## 2 JS 中循环的性能分析

### 2.1 for 和 while 性能分析

- 基于 `var` 声明的时候 `FOR` 和 `WHILE` 性能差不多 (不确定的时候使用 `while`)
- 基于 `let` 声明的时候 `FOR` 性能会更好一些 没有创造全局不释放的变量

### 2.2 forEach

- 会比 `for` `while` 性能差
- 不能退出循环 不能使用 `break` 退出循环

```js
let arr = new Arrar(9999999).fill(0);

console.time("FOR~~");
for (let i = 0; i < arr.length; i++) {}
console.timeEnd("FOR~~");

console.time("WHILE~~");

let i = 0;

while (i < arr.length) {
  i++;
}
console.timeEnd("WHILE~~");

arr.forEach((item) => {});
```

#### 2.2.1 手写 forEach

```js
let arr = [1, 2, 3, 4, 5];

Array.prototype.forEach = function (callback, context) {
  let _this = this,
    index = 0,
    len = _this.length;
  context = context == null ? window : context;
  for (; i < len; i++) {
    typeof callback === "function" ? callback.call(context, _this[i], i) : null;
  }
};
```

### 2.3 for in 循环

- 性能 最差 迭代当前对象中所有的可枚举的属性(私有属性大部分都是可枚举的 公有属性{出现在对象的原型链上的} 也有部分是可枚举的) 查找机制一定会 查找到原型链上
- 遍历顺序 是已数字优先
- 无法遍历 `Symbol` 属性
- 可以遍历到公有中可枚举的

```js
let obj = {
  name: "zs",
  age: 18,
  [symbol("A")]: "AA",
  0: 100,
  1: 200,
};

for (let key in obj) {
  // 解决遍历公有属性上 如果是公有属性 不是私有属性 则 break 出去
  if (!Object.hasOwnProperty(key)) break;
  console.log(key);
}

// 获取对象中的私有属性 非 symbol 外
let keys = Object.keys(obj);
// 如果浏览器支持 Symbol 则 加上对象中的symbol 属性
if (typeof Symbol !== "undefined")
  keys = keys.concat(Object.getOwnPropertySymbols(obj));

keys.forEach((key) => {
  console.log("属性名", key);
  console.log("属性值", obj[key]);
});
```

### 2.4 for of 循环

- `iterator` 迭代器
- 数组、部分类数组、`Set`、`Map` 对象没有
- `for of` 循环 是按照 迭代器规范遍历

```js
let arr = [1, 2, 3, 4, 5];
// 重写 iterator
arr[Symbol.iterator] = function () {
  let _this = this,
    index = 0;
  return {
    next() {
      // 必须具备 next 方法
      // done: false, value: 遍历的值
      if (index > _this.length - 1) {
        return {
          done: true,
          value: undefined,
        };
      }
      return {
        done: false,
        value: _this[index++],
      };
    },
  };
};
console.time("FOR OF~~");
for (const val of arr) {
  console.log(val);
}
console.timeEnd("FOR OF~~");

let obj = {
  0: 100,
  1: 200,
  2: 300,
  length: 3,
};

// 重写 obj 的 iterator
obj[Symbol.iterator] = Array.prototype[Symbol.iterator];
for (let val of obj) {
  console.log(val);
}
```

## 3. this 的理解以及应用场景

### 3.1 this 的理解分析

1. 函数执行 看方法前面是否有 "点" 没有 `this` 指向的 `window` (严格模式下是 `undefined`) 有 "点" 前面是谁 `this` 指向的就谁
2. 给当前元素的某个事件绑定方法 当事件行为触发 方法中 `this` 是当前元素本身
3. 构造函数的 this 指向的是当前类的实例
4. 箭头函数中没有执行主体 所用到 this 都是指向的所处上下文中的 `this`
5. 可以基于 `Function.prototype`上的 `call`/`apply`/`bind` 方法 改变 `this` 指向
6. call/apply/bind 的区别

```js
function func(x, y) {
  console.log(this, x, y);
}

const obj = {
  name: "obj",
};

/** 重写 call 方法
 * @params context 绑定的this 指向
 * @params params 参数
 * */

Function.prototype.call = function (context, ...params) {
  let _this = this,
    key = Symbol["KEY"],
    ret;
  context === null ? (context = window) : null;
  !/^(object|function)$/i.test(typeof context)
    ? (context = Object(context))
    : null;
  context[key] = _this; // 追加一个唯一值 = 被执行的函数
  ret = context[key](...params); // 执行函数
  delete context[key]; // 删除唯一值
  return ret;
};

// call apply 是立即执行函数
// call apply 接受参数方式不同
/**、
 * 1. 把 func 的 this 指向 obj
 * 2. 并把 params 接受的值当作实参传递给 func 函数
 * 3. 并把 func 函数 执行
 * */
func.call(obj, 10, 20);
func.apply(obj, [10, 20]);

/** 重写 bind 方法
 * @params context 绑定的this 指向
 * @params params 参数
 * */

Function.prototype.bind = function (context, ...params) {
  let _this = this;
  return function (...args) {
    _this.apply(context, params.concat(args));
  };
};

/**
 * 和 call apply 区别: 并不会立即执行
 * 1. 把 传递进来的 obj 10 20 等信息存储起来
 * 2. 执行bind返回一个新的函数 例如 proxy 把 proxy 绑定给 DOM 元素的事件 在 proxy 内部执行 func 函数 并把this 和值 都改变成 存储的内容
 * */
document.body.addEventListener("click", func.bind(obj, 10, 20));
```

### 3.2 应用场景 鸭子类型

```js
function func() {
  // arguments 变为数组
  let ret = Array.prototype.slice.call(arguments);
  console.log(ret);
}

func(10, 20, 30);
```
