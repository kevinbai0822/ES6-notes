# ES6学习笔记

## Proxy和Reflect
### Proxy概述
Proxy用于修改某些操作的默认行为，等同于在操作层面进行修改。就相当于对目标对象进行了代理，外界对目标对象的访问都要经过代理，代理对外界的访问进行过滤和改写，即用自己的定义覆盖语言原来的定义。  
ES6原生提供Proxy构造函数，用于生成Proxy实例。
```
var proxy = new Proxy(target,handler)
```
Proxy对象的所有用法都是上面这种形式，不同的只是handler参数的写法。其中，target参数表示所要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为。
```
var proxy = new Proxy({},{
    get: function(target, property){
        return 35;
    }
});

proxy.time //35
proxy.name //35
proxy.title //35
```
上面的例子中，作为构造函数Proxy接受两个参数：第一个参数是所要代理的目标对象（例子中是一个空对象），即如果没有Proxy介入，操作原来要访问的就是这个对象；第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。  
注意，要使Proxy起作用，必须针对Proxy实例进行操作，而不是针对目标对象进行操作。  
如果handler没有设置任何拦截，那就等同于直接通向源对象。
```
var target = {};
var handler = {};
var proxy = new Proxy(target, handler);
proxy.a = 'b';
target.a //b
```
上面的例子中，handler是一个空对象，没有任何拦截效果，访问handler就等同于访问target。  
一个技巧是将Proxy对象设置到object.proxty属性，从而可以在object对象上调用。
```
var object = {proxy: new Proxy(target,handler)}
```
Proxy 实例也可以作为其他对象的原型对象。
```
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```
`Object.create()会使用指定的原型对象及其属性去创建一个新的对象。`
同一个拦截器函数可以设置多个拦截操作。
Proxy支持的拦截操作：
```
（1）get(target, propKey, receiver)

拦截对象属性的读取，比如proxy.foo和proxy['foo']。

最后一个参数receiver是一个对象，可选，参见下面Reflect.get的部分。

（2）set(target, propKey, value, receiver)

拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。

（3）has(target, propKey)

拦截propKey in proxy的操作，返回一个布尔值。

（4）deleteProperty(target, propKey)

拦截delete proxy[propKey]的操作，返回一个布尔值。

（5）ownKeys(target)

拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。

（6）getOwnPropertyDescriptor(target, propKey)

拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。

（7）defineProperty(target, propKey, propDesc)

拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。

（8）preventExtensions(target)

拦截Object.preventExtensions(proxy)，返回一个布尔值。

（9）getPrototypeOf(target)

拦截Object.getPrototypeOf(proxy)，返回一个对象。

（10）isExtensible(target)

拦截Object.isExtensible(proxy)，返回一个布尔值。

（11）setPrototypeOf(target, proto)

拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。

如果目标对象是函数，那么还有两种额外操作可以拦截。

（12）apply(target, object, args)

拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。

（13）construct(target, args)

拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。
```

### Proxy实例的方法
见书第11.2小节

### Reflect概述
Refelct对象的设计目的有以下几个。
1. 将Object对象的一些明显属于语言内部的方法（比如Object.defineProperty），放到Reflect对象上。现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在Reflect对象上。也就是说，从Reflect对象上可以拿到语言内部的方法。  

2. 修改某些Object方法的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false。

3. 让Object操作都变成函数行为。某些Object操作是命令式，比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。

4. Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。这就让Proxy对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。也就是说，不管Proxy怎么修改默认行为，你总可以在Reflect上获取默认行为。
```
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target,name, value, receiver);
    if (success) {
      log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```
上面例子中，Proxy方法拦截target对象的属性赋值行为。它采用Reflect.set方法将值赋值给对象的属性，确保完成原有的行为，然后再部署额外的功能。

### Reflect静态方法
Reflect对象一共有13个静态方法。

- Reflect.apply(target,thisArg,args)
- Reflect.construct(target,args)
- Reflect.get(target,name,receiver)
- Reflect.set(target,name,value,receiver)
- Reflect.defineProperty(target,name,desc)
- Reflect.deleteProperty(target,name)
- Reflect.has(target,name)
- Reflect.ownKeys(target)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)
上面这些方法的作用，大部分与Object对象的同名方法的作用都是相同的，而且它与Proxy对象的方法是一一对应的。

解释见书第11.4小节