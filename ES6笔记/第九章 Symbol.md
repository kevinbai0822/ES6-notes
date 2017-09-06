# ES6 学习笔记

## Symbol
### 概述
ES5的原始数据类型总共有6种：Undefined、Null、Boolean、String、Number、Object。  
现在有一种情况，ES5对象的属性名都是字符串，这很容易造成属性名的冲突。比如，你使用一个别人提供的对象，现在你想为这个对象添加新的方法，新方法的名字很有可能与已有方法产生冲突。所以，ES6引入了第7种原始数据类型`Symbol`,它表示独一无二的值。  
Symbol值通过Symbol函数生成。这就是说，对象的属性名可以有两种类型：一种是原来就有的字符串，另一种就是新增的Symbol类型。只要属性名属于Symbol类型，就不会与其他属性名冲突。
```
let s = Symbol();
typeof(s) //symbol
```
注意，Symbol函数前不能使用new命令，否则会报错。这是因为生成的Symbol是一个原始数据类型，不是对象，所以也不能添加属性。  
Symbol函数可以接受一个字符串作为参数，表示对Symbol实例的描述。注意，Symbol函数接受的参数只是对当前Symbol值的描述，所以即使具有相同参数的Symbol函数，其返回值也不相等。
```
let s1 = Symbol('leo');
let s2 = Symbol('leo');
s1 === s2 //false
```
Symbol值不能与其他类型的值进行运算，否则会报错。但是，Symbol值可以显式地转为字符串。
```
let s1 = Symbol('leo');
String(s1) //Symbol(leo)
s1.toString() //Symbol(leo)
```
另外Symbol值也能转为布尔值，但是不能转为数值。

### 作为属性名的Symbol
由于每一个Symbol值都是不相等的，这意味着Symbol可以作为标识符用于对象的属性名，保证不会出现同名的属性。
```
var mySymbol = Symbol();

// 第一种写法
var a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
var a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
var a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```
注意，Symbol值作为对象属性名时不能使用点运算符。
```
var mySymbol = Symbol();
var a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```
上面的例子中，因为点运算符后面总是字符串，所以不会读取Symbol值作为标识符所指代的值。所以，才对象内部使用Symbol值定义属性时，Symbol值必须放在方括号中。
```
let s = Symbol();

let obj = {
  [s]: function (arg) { ... }
};

obj[s](123);
```

### 消除魔术字符串
略

### 属性名的遍历
Symbol 作为属性名，该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。但是，它也不是私有属性，有一个Object.getOwnPropertySymbols方法，可以获取指定对象的所有 Symbol 属性名。
```
var obj = {};
var a = Symbol('a');
var b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

var objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```
另一个新的API，Reflect.ownKeys方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。
```
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};

Reflect.ownKeys(obj)
//  ["enum", "nonEnum", Symbol(my_key)]
```
由于以 Symbol 值作为名称的属性，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。

### Symbol.for()，Symbol.keyFor()
有时，我们希望重新使用同一个Symbol值，Symbol.for方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建并返回一个以该字符串为名称的Symbol值。
```
var s1 = Symbol.for('foo');
var s2 = Symbol.for('foo');

s1 === s2 // true
```
Symbol.keyFor方法返回一个已登记的 Symbol 类型值的key。
```
var s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

var s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```

### 内置的Symbol值
ES6提供了11个内置的Symbol值。
1. Symbol.hasInstance  
对象的Symbol.hasInstance属性，指向一个内部方法。当其他对象使用instanceof运算符，判断是否为该对象的实例时，会调用这个方法。

2. Symbol.isConcatSpeadable  
对象的Symbol.isConcatSpreadable属性等于一个布尔值，表示该对象使用Array.prototype.concat()时，是否可以展开。

3. Symbol.species  
对象的Symbol.species属性，指向当前对象的构造函数。创造实例时，默认会调用这个方法，即使用这个属性返回的函数当作构造函数，来创造新的实例对象。

4. Symbol.match  
对象的Symbol.match属性，指向一个函数。当执行str.match(myObject)时，如果该属性存在，会调用它，返回该方法的返回值。

5. Symbol.replace  
对象的Symbol.replace属性，指向一个方法，当该对象被String.prototype.replace方法调用时，会返回该方法的返回值。

6. Symbol.search  
对象的Symbol.search属性，指向一个方法，当该对象被String.prototype.search方法调用时，会返回该方法的返回值。

7. Symbol.split
对象的Symbol.split属性，指向一个方法，当该对象被String.prototype.split方法调用时，会返回该方法的返回值。

8. Symbol.iterator
对象的Symbol.iterator属性，指向该对象的默认遍历器方法。

9. Symbol.toPrimitive  
对象的Symbol.toPrimitive属性，指向一个方法。该对象被转为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值。

Symbol.toPrimitive被调用时，会接受一个字符串参数，表示当前运算的模式，一共有三种模式。
- Number：该场合需要转成数值
- String：该场合需要转成字符串
- Default：该场合可以转成数值，也可以转成字符串

10. Symbol.toStringTag  
对象的Symbol.toStringTag属性，指向一个方法。在该对象上面调用Object.prototype.toString方法时，如果这个属性存在，它的返回值会出现在toString方法返回的字符串之中，表示对象的类型。也就是说，这个属性可以用来定制[object Object]或[object Array]中object后面的那个字符串。

11. Symbol.unscopables  
对象的Symbol.unscopables属性，指向一个对象。该对象指定了使用with关键字时，哪些属性会被with环境排除。