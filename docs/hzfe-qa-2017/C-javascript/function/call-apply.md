```bash
# 此页面贡献者：年轻的小铲、树
```
## hzfe小剧场

村长玉香有一炒锅，而年轻的阿盖没有。没关系，那就借玉香的。锅还是那口锅，只是用的人不一样了，这大概，就是物是人非吧。使用 `apply` 的伪代码： `玉香.炒锅.apply(阿盖, [菜, 肉肉])` 。

而 `call` 的功效跟 `apply` 是一样的，仅仅传递参数的方式不同： `玉香.炒锅.call(阿盖, 菜, 肉肉)` 。

记不住哪个用 `数组/类数组对象` ，哪个用 `参数列表` 怎么办。铲式记忆法： `apply` 比 `call` 长，所以用的时候也多了个 `[]` 。

## Apply

> func.apply(thisArg, [argsArray])

The apply() method calls a function with a given `this` value, and `arguments` provided as an array (or an array-like object).

## Call

> function.call(thisArg, arg1, arg2, ...)

The call() method calls a function with a given `this` value, and `arguments` provided individually.

## 性能对比

借助 `benchmark` 对以下代码进行测试，会发现 `call` 比 `apply` 快很多。

```javascript
var test1 = function (str) {
  return [].slice.apply(str)
}

var test2 = function (str) {
  return [].slice.call(str)
}

test1('tefgjhklst')
test2('tefgjhklst')
```

## 规范

> When the **apply** method is called on an object func with arguments `thisArg` and `argArray`, the following steps are taken:
  1. If IsCallable(func) is false, throw a TypeError exception.
  2. If `argArray` is undefined or null, then
    a. Perform PrepareForTailCall().
    b. Return ? `Call(func, thisArg)`.
  3. Let `argList` be ? `CreateListFromArrayLike(argArray)`.
  4. Perform PrepareForTailCall().
  5. Return ? `Call(func, thisArg, argList)`.

> When the **call** method is called on an object func with argument, `thisArg` and zero or more `args`, the following steps are taken:
  1. If IsCallable(func) is false, throw a TypeError exception.
  2. Let `argList` be a new empty List.
  3. If this method was called with more than one argument, then in left to right order, starting with the second argument, append each argument as the last element of `argList`.
  4. Perform PrepareForTailCall().
  5. Return ? `Call(func, thisArg, argList)`.

不难看出，两者的区别主要是 `apply` 调用了 `CreateListFromArrayLike` 操作符。规范：

> The abstract operation `CreateListFromArrayLike` is used to create a `List` value whose elements are provided by the indexed properties of an array-like object, obj. The optional argument elementTypes is a List containing the names of ECMAScript Language Types that are allowed for element values of the List that is created. This abstract operation performs the following steps:

## Demo

在实现javascript的对象继承的时候，除了使用原型链的方式外，还可以使用`call/apply`。举个栗子：

```JavaScript
function Person (name, age) {
  this.age  = age
  this.name = name
  this.sayName = function(){ console.log(this.name) }
}
function Student (name, age, grade) {
  Person.call(this, name, age, grade)
  // Person.apply(this, [name, age, grade])
  // Person.apply(this, arguments)
  this.grade = grade
}
```

```JavaScript
var student = new Student("goodboy",22,"little school")
console.log(`name: ${student.name} age: ${student.age} grade: ${student.grade}`) // name: goodboy age: 22 grade: little school
student.sayName() // goodboy
```

`Student`函数对象本身是没有`sayName`等方法属性的，而`Person.call(this)`让`Student`对象可以调用`Person`的方法以及属性。并且，`call/apply`二者的作用完全一样，只是接受参数的方式不太一样。

如果某个函数的参数数量不固定，用`apply`，用数组传递。也可以通过`arguments`来获得所有参数。如果某个函数的参数数量固定，可以用`call`。

再举个栗子：
```html
<input type="text" id="inputTest" value="input val">
```
```JavaScript
var value = "global val"
function setVal(){ this.value = "local val" }
function getVal(){ console.log(this.value) }
```
```JavaScript
window.getVal() // global val
getVal.call(window) // global val
getVal.call(new setVal) // local val
getVal.call(document.getElementById('inputTest')) // input text
```
也就是说`call/apply`都是为了改变某个函数运行时的`context`，即上下文而存在的，即改变函数体内部`this`的指向。

再举个栗子加深理解：
```JavaScript
var member = [
  { species: 'hzfe', name: 'goodhome' },
  { species: 'hzfe', name: 'taotao' }
]
for (var i = 0; i < member.length; i++) {
  (function (i) {
    this.print = function(){ console.log(`#${i} ${this.species}: ${this.name}`) }
    this.print()
  }).call(member[i], i)
}

// #0 hzfe: goodhome
// #1 hzfe: taotao
```

百度出来的古董面试题：定义一个`log`方法，让它可以代理`console.log`方法，常见的解决方法是：
```JavaScript
function log (msg)　{ console.log(msg) }
log(1)    // 1
log(1, 2) // 1

// 传入参数的个数是不确定的，所以应该改成这样：
function log () { console.log.apply(console, arguments) }
log(1)    // 1
log(1, 2) // 1 2

// 假如有些自定义的标识前缀，比如令log("hello world")打印出`hzfe dev mode: hello world`：
function log () {
  let args = Array.prototype.slice.call(arguments)
  args.unshift('hzfe dev mode: ')
  console.log.apply(console, args)
}
log("hello world") // hzfe dev mode: hello world
```

## 学习资料
[JavaScript深入之call和apply的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)

[理解js中call,apply,caller,callee的区别](http://xuyuan923.github.io/2015/01/24/理解js中call,apply,bind,caller,callee的区别/)

[【优雅代码】深入浅出 妙用Javascript中apply、call、bind](http://www.cnblogs.com/coco1s/p/4833199.html)
