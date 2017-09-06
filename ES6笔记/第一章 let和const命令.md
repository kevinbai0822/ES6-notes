# ES6学习笔记

## let和const命令

### let命令
let命令用法类似于var，但是所声明的变量只在let命令所在代码块内有效。
```
{
    let a = 10;
    var b = 11;
}
a //ReferenceError: a is not defined
b //11
```
```
var a = [];
for (var i = 0; i < 10; i++) {
    a[i] = function(){
        console.log(i);
    }
}

a[6](); //10

var a = [];
for (let i = 0; i < 10; i++) {
    a[i] = function(){
        console.log(i);
    }
}

a[6](); //6
```
for循环是一个典型的例子，var定义的变量是全局变量，所以每一次循环新的值都会覆盖旧值，从而输出的结果为最后一轮的10。但是let定义的变量只在当前代码块内有效，所以每次循环的i都是一个新的变量，所以结果输出6。

#### 没有变量提升
此外，let不存在变量提升，所以变量要在声明之后才能使用，否则将会报错。这有利于开发者养成先声明变量再使用的习惯，同时这也意味着`typeof`不再是一个百分百安全的操作。
```
console.log(leo);
let leo = 7; //ReferenceError: leo is not defined
```

#### 暂时性死区
只要块级作用域内存在let命令，它声明的变量就绑定在这块区域内，不再受到外部影响。
```
var leo = 'lion';
if(1){
    leo = 7;
    let leo; //ReferenceError: leo is not defined
    console.log(leo); //undefined
}
```
如果在块级作用域内存在let声明，那么在声明之前，该变量都是不可用的，这被称为"暂时性死区"。暂时性死区的本质就是，只要一进入当前作用域，那么变量就是已经存在但不可实用的，只有当声明变量的代码出现，这个变量才可以使用。

#### 不允许重复声明
let不允许在相同的作用域内重复声明同一个变量。因此，不能再函数内部重复声明参数。
```
function func(arg){
    let arg; //error
}
```

### 块级作用域
ES5只有全局作用域和函数作用域，没有块级作用域，这会导致很多不合理的场景。
- 内层变量覆盖外层变量。
- 用来计数的循环变量泄露为全局变量。
ES6新增了块级作用域
```
function f1(){
    let n = 5;
    if(true){
        let n = 10;

    }
    console.log(n); //5
}
```
因为let存在块级作用域，所以外层代码块不受内层代码块的影响，输出结果为5，如果使用var定义，则输出结果为10。

块级作用域允许任意嵌套，外层作用域无法访问内层作用域的变量，并且内层作用域可以定义和外层作用域同名的变量。块级作用域使得立即执行匿名函数不再需要，因为不用担心变量泄露的问题。ES6规定函数的作用域在其所在的块级作用域内。
```
function f(){
    console.log("我在外面");
}

(function (){
    if(false){
        function f(){
            console.log("我在里面");
        }
    }

    f(); //我在外面
}());
```

### const命令
const命令一般用来声明常量。一旦声明，其值就不能改变。
```
const leo = 7;
leo; //7
const leo = 8; //SyntaxError: Identifier 'leo' has already been declared
```
const命令声明的常量必须立即使用，不能留到以后赋值。
```
const leo; //Missing initializer in const declaration
```
const作用域与let相同，同样没有变量提升，同样存在暂时性死区。

对于复合类型的对象，变量名不指向数据，而是只想数据所在的地址(参照javascript变量的存储)。const命令只是保持变量名指向的地址不变，不能保证改地址的变量不变，所以将对象声明为常量是要格外注意。
```
const obj = {};
obj.prop = 123;

obj = {}; //Assignment to constant variable
```
const声明的常量只在当前代码块内有效，如果想要设置块模块常量，采用export import的方式引入。

### 全局对象的属性
全局对象是最顶层的对象。在ES5中，全局对象的属性与全局变量是等价的。这可能使开发者在不知不觉中就创建了全局变量。ES6中规定，一方面var与function声明的全局变量依旧是全局对象的属性，另一方面，let const class声明的全局变量则不属于全局对象的属性。
```
var a = 0;
window.a //0

let b = 1;
window.b //undefined
```