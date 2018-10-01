---
title: 【译】ES2018 新特性：Rest/Spread 特性
date: 2018-07-09 21:10:24
tags:
- 翻译
- JavaScript
categories:
- 前端
---

Sebastian Markbåge 提出的 Rest/Spread Properties 提案包括两部分：

* 用于对象解构的 rest 操作符(...)。目前，这个操作符只能在数组解构和参数定义中使用
* 对象字面量中的 spread 操作符(...)。目前，这个操作符只能用于数组字面量和在函数方法中调用。

## 对象解构中的 rest 操作符(...)

在对象解构模式下，rest 操作符会将解构源的除了已经在对象字面量中指明的属性之外的，所有可枚举自有属性拷贝到它的运算对象中。

```
const obj = {foo: 1, bar: 2, baz: 3};
const {foo, ...rest} = obj;
    // Same as:
    // const foo = 1;
    // const rest = {bar: 2, baz: 3};
```

如果你正在使用对象解构来处理命名参数，rest 操作符让你可以收集所有剩余参数：

```
function func({param1, param2, ...rest}) { // rest operator
    console.log('All parameters: ',{param1, param2, ...rest}); // spread operator
    return param1 + param2;
}
```

#### 语法限制

在每个对象字面量的顶层，可以使用 rest 操作符最多一次，并且必须只能在末尾出现：

```
const {...rest, foo} = obj; // SyntaxError
const {foo, ...rest1, ...rest2} = obj; // SyntaxError
```

如果是嵌套结构，你可以多次使用 rest 操作符：

```
const obj = {
    foo: {
        a: 1,
        b: 2,
        c: 3,
    },
    bar: 4,
    baz: 5,
};

const {foo: {a, ...rest1}, ...rest2} = obj;

// Same as:
// const a = 1;
// const rest1 = {b: 2, c: 3};
// const rest2 = {bar: 4, baz: 5};

```

## 对象字面量中的 spread 操作符

对象字面量内部，spread 操作符将自身运算对象的所有可枚举的自有属性，插入到通过字面量创建的对象中：

```
> const obj = {foo: 1, bar: 2, baz: 3};
> {...obj, qux: 4}
{ foo: 1, bar: 2, baz: 3, qux: 4 }
```

要注意的是顺序问题，即使属性 key 并不冲突，因为对象会记录插入顺序：

```
> {qux: 4, ...obj}
{ qux: 4, foo: 1, bar: 2, baz: 3 }
```

如果 key 出现了冲突，后面的会覆盖前面的属性：

```
> const obj = {foo: 1, bar: 2, baz: 3};
> {...obj, foo: true}
{ foo: true, bar: 2, baz: 3 }
> {foo: true, ...obj}
{ foo: 1, bar: 2, baz: 3 }
```

## 对象 spread 操作符的使用场景

这一节，我们会看看 spread 操作符的使用场景。我也会用 Object.assign() 实现一遍，它和 spread 操作符很相似（之后我们会更详细地比较它们）。

#### 拷贝对象

拷贝对象 obj 的可枚举自有属性：

```
const clone1 = {...obj};
const clone2 = Object.assign({}, obj);
```

clone 对象们的原型都是 Object.prototype，它是所有通过对象字面量创建的对象的默认原型：

```
> Object.getPrototypeOf(clone1) === Object.prototype
true
> Object.getPrototypeOf(clone2) === Object.prototype
true
> Object.getPrototypeOf({}) === Object.prototype
true
```

拷贝一个对象 obj，包括它的原型：

```
const clone1 = {__proto__: Object.getPrototypeOf(obj), ...obj};
const clone2 = Object.assign(
    Object.create(Object.getPrototypeOf(obj)), obj);
```

注意，一般来说，对象字面量内部的 __proto__ 只是浏览器内置的特性，并非 JavaScript 引擎所有。

#### 对象的真拷贝

有时候，你需要老老实实地拷贝对象的所有自有属性(properties)和特性(writable, enumerable, …)，包括 getters 和 setters。这时候 Object.assign() 和 spread 操作符就回天乏术了。你需要使用属性描述符([property descriptors](http://speakingjs.com/es5/ch17.html#property_attributes))：
```
const clone1 = Object.defineProperties({},
    Object.getOwnPropertyDescriptors(obj));
```
如果还希望保留 obj 的原型，可以用 [`Object.create()`](http://speakingjs.com/es5/ch17.html#Object.create)：
```
const clone2 = Object.create(
    Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj));
```
“探索 ES2016 and ES2017” 里介绍了 [`Object.getOwnPropertyDescriptors()`](http://exploringjs.com/es2016-es2017/ch_object-getownpropertydescriptors.html)
#### 陷阱：总是浅拷贝
我们之前见过的所有拷贝对象的方式，都是浅拷贝：如果原始属性值是一个对象，拷贝的对象将指向同一个对象，它不会（递归的、深度的）拷贝自身：
```
const original = { prop: {} };
const clone = Object.assign({}, original);

console.log(original.prop === clone.prop); // true
original.prop.foo = 'abc';
console.log(clone.prop.foo); // abc
```
#### 其他使用场景
合并 obj1 和 obj2 两个对象：
```
const merged = {...obj1, ...obj2};
const merged = Object.assign({}, obj1, obj2);
```
给用户数据填充默认值
```
const DEFAULTS = {foo: 'a', bar: 'b'};
const userData = {foo: 1};

const data = {...DEFAULTS, ...userData};
const data = Object.assign({}, DEFAULTS, userData);
    // {foo: 1, bar: 'b'}
```
安全地更新属性 foo:
```
const obj = {foo: 'a', bar: 'b'};
const obj2 = {...obj, foo: 1};
const obj2 = Object.assign({}, obj, {foo: 1});
    // {foo: 1, bar: 'b'}
```
指定属性 foo  和 bar 的默认值：
```
const userData = {foo: 1};
const data = {foo: 'a', bar: 'b', ...userData};
const data = Object.assign({}, {foo:'a', bar:'b'}, userData);
    // {foo: 1, bar: 'b'}
```
## 展开对象 VS Object.assign()
spread 操作符和 Object.assign() 很相似。主要的区别在于前者定义了新属性，而后者还进行了赋值。稍后将解释这究竟意味着什么。
#### Object.assign() 的两种使用方式
Object.assign() 有两种使用方式：
第一种，带有破坏性的（修改已有对象）：
```
Object.assign(target, source1, source2);
```
这里的 target 对象被修改了；source1 和 source2 被拷贝进去了。
第二种，非破坏性的（已有对象不会被修改）：
```
const result = Object.assign({}, source1, source2);
```
新对象是通过将 source1 和 source2 拷贝进一个空对象而生成的。最终，这个新对象被返回并赋值给 result。
spread 操作符类似于 Object.assign() 的第二种方式。接下来，我们来看看两者的相似和不同之处。
#### 都是通过 "get" 操作符读值
在写对象之前，两者都使用了 ”get“ 操作符去读取源对象的属性值。这一过程会将 getter 被转换成正常的数据属性。
来看个例子：
```
const original = {
    get foo() {
        return 123;
    }
};
```
original 有一个 foo getter(它的属性描述符有 get 和 set 属性)
```
> Object.getOwnPropertyDescriptor(original, 'foo')
{ get: [Function: foo],
  set: undefined,
  enumerable: true,
  configurable: true }
```
但是在它拷贝的结果 clone1 和 clone2 里，foo 是一个正常的数据属性（属性描述符有value 和 writable 属性）：
```
> const clone1 = {...original};
> Object.getOwnPropertyDescriptor(clone1, 'foo')
{ value: 123,
  writable: true,
  enumerable: true,
  configurable: true }

> const clone2 = Object.assign({}, original);
> Object.getOwnPropertyDescriptor(clone2, 'foo')
{ value: 123,
  writable: true,
  enumerable: true,
  configurable: true }
 ```
#### spread 定义属性，Object.assign() 设置属性
spread 操作符在目标对象上定义了新的属性，而Object.assign() 使用了一个 "set" 操作符来创建属性。这会导致两个结果：
###### 目标对象带有 setter
首先，Object.assign() 触发 setter，而 spread 不会：
```
Object.defineProperty(Object.prototype, 'foo', {
    set(value) {
        console.log('SET', value);
    },
});
const obj = {foo: 123};
```
以上代码段设置了一个 foo setter，它会被所有普通对象继承。
如果我们通过 Object.assign() 拷贝 obj，继承的 setter  会被触发：
```
> Object.assign({}, obj)
SET 123
{}
```
而 spread 就不会：
```
> { ...obj }
{ foo: 123 }
```
Object.assign() 在拷贝时还会触发自有 setter，这里并没有发生重写。
###### 目标对象带有只读属性
第二，你可以通过继承只读属性，来阻止 Object.assign() 创建自有属性，但 spread 上这是做不到的：
```
Object.defineProperty(Object.prototype, 'bar', {
    writable: false,
    value: 'abc',
});
```
以上代码设置了只读属性 bar，它会被所有普通对象继承。
这样，你就再也不能使用赋值语句去创建自有属性 bar（严格模式下会抛一个异常，宽松模式会静默失败）：
```
> const tmp = {};
> tmp.bar = 123;
TypeError: Cannot assign to read only property 'bar'
```
下列代码，我们使用对象字面量成功地创建了属性 bar。因为对象字面量没有设置属性，它只是定义了它们：
```
const obj = {bar: 123};
```
然而，Object.assign() 使用赋值语句创建属性，这就是不能拷贝 obj 的原因：
```
> Object.assign({}, obj)
TypeError: Cannot assign to read only property 'bar'
```
通过 spread 操作符拷贝没有问题：
```
> { ...obj }
{ bar: 123 }
```
#### spread 和 Object.assign() 都只拷贝自有可枚举属性
它们都会忽略所有继承的属性和不可枚举的自有属性。
对象 obj 从 proto 继承了一个可枚举属性，并且有两个自有属性：
```
const proto = {
    inheritedEnumerable: 1,
};
const obj = Object.create(proto, {
    ownEnumerable: {
        value: 2,
        enumerable: true,
    },
    ownNonEnumerable: {
        value: 3,
        enumerable: false,
    },
});
```
如果拷贝 obj，结果将只有属性 ownEnumerable。属性 inheritedEnumerable 和 ownNonEnumerable 没有被拷贝：
```
> {...obj}
{ ownEnumerable: 2 }
> Object.assign({}, obj)
{ ownEnumerable: 2 }
```



-----------
原文：http://exploringjs.com/es2018-es2019/ch_rest-spread-properties.html
