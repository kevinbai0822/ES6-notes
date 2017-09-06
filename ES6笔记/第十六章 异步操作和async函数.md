# ES6学习笔记

## 异步操作和async函数
异步编程对于javascript语言极为重要。众所周知，javascript是单线程的，如果没有异步编程，那么程序会卡死。
ES6之前，异步编程大概有4种方法。
- 回调函数
- 事件监听
- 发布订阅者模式
- Promise对象
ES6的异步编程进入了一个全新的阶段，ES7的async更是大放异彩。

### 基本概念
#### 异步
所谓的异步，就是一个任务分成两个阶段，先执行第一段，然后转做其他任务，等做好准备之后，再回来执行第二段。

#### 回调函数
javascript对异步编程的实现就是回调函数。所谓回调函数，就是将任务的第二阶段写在一个函数中，等到重新执行该任务的时候直接调用这个函数。

#### Promise
回调函数本身是没有问题的，问题在于，如果任务的阶段过多的话，会出现多个回调函数嵌套的问题，这样代码会出现横向发展，不利于维护，即所谓的“回调地狱”。

Promise就是为了解决这个问题而提出的，它不是一个新的语法功能，只是一种新的写法，将回调函数的横向加载改成纵向加载。

Promise最大的问题在于代码冗余，原来的任务被Promise包装一下之后，一眼看上去都是一堆then，这使得语义变得不清晰。

### Generator函数
详见笔记第十四章。

### Thunk函数
#### 参数的求值策略
函数的参数到底应该什么时候求值？
- 传值调用，即进入函数体前就计算表达式的值，然后将计算的结果传入函数。
- 传名调用，即将表达式传入函数体，只在用到的时候进行求值。

#### Thunk函数的含义
编译器的“传名调用”往往是先将参数放到一个临时函数中，再将这个临时函数传入函数体。这个临时函数就叫做Thunk函数。

后续请看书第17.3小节

### co模块
请看书第17.4小节

### async函数
async函数就是Generator函数的语法糖。

下面是一个Generator函数
```
var fs = require('fs');

var readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```
写成async函数，就是下面这样。
```
var asyncReadFile = async function () {
  var f1 = await readFile('/etc/fstab');
  var f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```
一比较就会发现，async函数就是将 Generator 函数的星号（*）替换成async，将yield替换成await，仅此而已。

async函数对 Generator 函数的改进，体现在以下四点。
1. 内置执行器。  
Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。

2. 更好的语义。  
async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。

3. 更广的适用性。  
co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

4. 返回值是 Promise。  
async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。


#### async 函数的实现
async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里。
```
async function fn(args) {
  // ...
}

// 等同于

function fn(args) {
  return spawn(function* () {
    // ...
  });
}
```
所有的async函数都可以写成上面的第二种形式，其中的spawn函数就是自动执行器。

#### async函数的用法
async函数返回一个 Promise 对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。
```
async function getStockPriceByName(name) {
  var symbol = await getStockSymbol(name);
  var stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```
上面代码是一个获取股票报价的函数，函数前面的async关键字，表明该函数内部有异步操作。调用该函数时，会立即返回一个Promise对象。

#### 注意点
第一点，前面已经说过，await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中。
```
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

第二点，多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。
```
let foo = await getFoo();
let bar = await getBar();
```
上面代码中，getFoo和getBar是两个独立的异步操作（即互不依赖），被写成继发关系。这样比较耗时，因为只有getFoo完成以后，才会执行getBar，完全可以让它们同时触发。
```
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```
上面两种写法，getFoo和getBar都是同时触发，这样就会缩短程序的执行时间。
