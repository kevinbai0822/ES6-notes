# stylus笔记
目前江湖上有多种css预处理器，最为熟知的应该就是less和Sass。因为项目需要，接触到了新的Stylus预处理器，具体的比较这里就不细说了。今天开始学习一下stylus。  
本笔记参考网上的中文文章以及官方项目[Stylus](https://github.com/stylus/stylus/)，所有实例出自英文文档，详细的英文文档请参考[这里](http://stylus-lang.com/)。

## 准备步骤
直接使用npm安装
```
$ npm install stylus -g
```
安装成功后，新建一个以.styl为后缀的文件。并执行以下命令。
```
stylus -w style.styl -o style.css
```
上面的命令监听名为style.styl的文件，并且同步编译成style.css。

## 选择器（Selectors）

### 缩进（Indentation）
在Stylus中，空格是有意义的，所以我们用缩进和凸排（减少缩进）来代替`{`和`}`
```
body
    color #fff
```
上面的代码编译成
```
body {
  color: #fff;
}
```
如果喜欢，你可以使用冒号来分隔属性和值。（这样做有利于分清哪个是属性哪个是值）。

### 规则集（Rule Sets）
就像CSS一样，Stylus允许你通多逗号来分隔多个选择器定义属性。
```
textarea, input
    border 1px solid #eee
```
同样的，可以另起一行写
```
textarea
input
	border 1px solid #eee
```
它们都会被编译成
```
textarea,
input {
	border: 1px solid #eee;
}
```
唯一的例外是，选择器看起来像属性。
```
foo bar baz
> input
    border 1px solid
```
上面的例子中，`foo bar baz`可能是一个选择器也可能是一个属性，所以为了这个原因，我们可以在尾部加上一个逗号。
```
foo bar baz,
form input,
> a
    border 1px solid
```

### 父级引用（Parent Reference）
字符`&`引用父选择器
```
textarea
input
    color #A7A7A7
    &:hover
        color #000
```
编译成
```
textarea,
input {
	color: #a7a7a7;
}
textarea:hover,
input:hover {
	color: #000;
}
```
上面的例子中，`textarea`和`input`的伪类hover都改变了颜色。

下面这个例子，IE浏览器利用了父级引用以及混合书写来实现2px的边框。
```
box-shadow()
    -webkit-box-shadow arguments
    -moz-box-shadow arguments
    box-shadow arguments
    html.ie8 &,
    html.ie7 &,
    html.ie6 &
      border 2px solid arguments[length(arguments) - 1]

body
    #login
        box-shadow 1px 1px 3px #eee
```
编译成
```
body #login {
	-webkit-box-shadow: 1px 1px 3px #eee;
	-moz-box-shadow: 1px 1px 3px #eee;
	box-shadow: 1px 1px 3px #eee;
}

html.ie8 body #login,
html.ie7 body #login,
html.ie6 body #login {
	border: 2px solid #eee;
}
```
如果你在选择器里需要使用`&`符号而不让它作为父级引用，你可以进行转义。
```
.foo[title*='\&']
// => .foo[title*='&']
```

### 部分引用（Partial Reference）
当`^[N]`出现在选择器中任意位置并且`N`是一个数字是，表示部分引用。  
部分引用与父级引用类似，但是当父级引用包含整个选择器的时候，部分引用仅包含嵌套选择器的第一个合并N级，因此你可以单独访问这些嵌套级。  
 ^[0] 将给你来自第一级的选择器, ^[1]将给你来自第二级的已渲染地选择器，依此类推。
```
.foo
  &__bar
    width: 10px

    ^[0]:hover &
        width: 20px
```
会被渲染成
```
.foo__bar {
  width: 10px;
}
.foo:hover .foo__bar {
  width: 20px;
}
```
负值从末尾开始计数，所以^[-1]在`&`之前的链中的最后一个选择器。
```
.foo
  &__bar
    &_baz
        width: 10px

        ^[-1]:hover &
            width: 20px
```
会被渲染成
```
.foo__bar_baz {
  width: 10px;
}
.foo__bar:hover .foo__bar_baz {
  width: 20px;
}
```
当你不知道你所处的嵌套层级时，负值的使用显得尤其有用。  
请注意，部分引用包含了嵌套层级之前选择器的整个渲染链，而不是选择器的一个“部分”。

### 范围部分引用（Ranges in partial references）
选择器中任意位置出现`^[N..M]`,`N`和`M`可以为数字，这代表了范围部分引用。  

如果你要获取选择器的原始部分，或者编程部分的范围，你可以使用范围部分引用。
```
.foo
  & .bar
    width: 10px

    ^[0]:hover ^[1..-1]
      width: 20px
```
上面会被渲染成
```
.foo .bar {
  width: 10px;
}
.foo:hover .bar {
  width: 20px;
}
```

### 初始引用（Initial Reference）
选择器开头的`~/`字符用于指向第一次嵌套时的选择器，可以将其视作`^[0]`的快捷方式，唯一的缺点是你只能在选择器的开头使用。
```
.block
  &__element
    ~/:hover &
      color: red
```
上面代码被编译成
```
.block:hover .block__element {
  color: #f00;
}
```

### 相对引用（Relative Reference）
`../`标记为一个相对引用，它指向`&`编译选择器的前一个。你可以使用嵌套的相对引用`../../`来获取更深的层级，但是注意，它只能在选择器的开头使用。
```
.foo
  .bar
    width: 10px

    &,
    ../ .baz
      height: 10px
```
上面代码被编译成
```
.foo .bar {
  width: 10px;
}
.foo .bar,
.foo .baz {
  height: 10px;
}
```

### 根引用（Root Reference）
选择器开始处的`/`字符表示根引用，它引用根上下文，这就意味着选择器不会向它添加父级选择器（除非你使用`&`），当你需要为某些嵌套选择器或者不在当前范围内的其他选择器编写一些样式时，根引用是很有帮助的。
```
textarea
input
  color #A7A7A7
  &:hover,
  /.is-hovered
    color #000
```
上面的代码被编译成
```
textarea,
input {
  color: #a7a7a7;
}
textarea:hover,
input:hover,
.is-hovered {
  color: #000;
}
```
### 消除歧义（Disambiguation）
诸如margin-n的表达式可以被解释为减法操作，也可以被解释为具有一元减号的属性。
如果要消除歧义，需要使用括号将表达式括起来。
```
pad(n)
  margin (- n)

body
  pad(5px)
```
编译成
```
body {
  margin: -5px;
}
```
但是，只有在函数中才会这样（因为函数既可以作为混合函数，也可以使用返回值进行调用）。
例如，下面这个就是好的（产生跟上面一样的结果）
```
body
  margin -5px
```

有Stylus无法处理的属性值？ `unquote()`可以帮你
```
filter unquote('progid:DXImageTransform.Microsoft.BasicImage(rotation=1)')
```
编译成
```
filter progid:DXImageTransform.Microsoft.BasicImage(rotation=1)
```

## 变量（Variables）

我们可以为变量分配表达式（即指定表达式为变量），并且在整个样式表中使用它们。
```
font-size = 14px

body
   font font-size Arial, sans-serif
```
编译成
```
body {
   font: 14px Arial, sans-serif;
}
```
变量甚至可以组成一个表达式列表。
```
font-size = 14px
font = font-size "Lucida Grande", Arial

body
  font font, sans-serif
```
编译成
```
body {
  font: 14px "Lucida Grande", Arial, sans-serif;
}
```

标识符（变量名，函数等）也可以包括$字符。 例如
```
$font-size = 14px
body {
  font: $font-size sans-serif;
}
```

### 属性查询（Property Lookup）
Stylus所独有的一个很酷的功能是，可以定义属性而不用将值分配给变量。一个很好的例子是元素的垂直水平居中（通常使用百分比和marigin负值来完成）
···
 #logo
   position: absolute
   top: 50%
   left: 50%
   width: w = 150px
   height: h = 80px
   margin-left: -(w / 2)
   margin-top: -(h / 2)
···
我们可以不使用w和h，而是简单的在属性名之前添加`@`字符来访问属性名所对应的值。
```
 #logo
   position: absolute
   top: 50%
   left: 50%
   width: 150px
   height: 80px
   margin-left: -(@width / 2)
   margin-top: -(@height / 2)
```
另外使用案例是基于其他属性有条件地定义属性。在下面这个例子中，我们默认指定z-index值为1，但是，只有在z-index之前未指定的时候才这样
```
  position()
    position: arguments
    z-index: 1 unless @z-index

  #logo
    z-index: 20
    position: absolute

  #logo2
    position: absolute
```
属性会“向上冒泡”查找堆栈直到被发现，或者返回null（如果属性搞不定）。下面这个例子，@color被解析成blue
```
body
    color: red
    ul
      li
        color: blue
        a
          background-color: @color
```

## 插值（Interpolation）
Stylus支持通过包含表达式的`{}`来插入值，使其变成标识符的一部分。例如，`-webkit-{'border' + '-radius'}`等同于`-webkit-border-radius`。  
一个很好的例子就是私有属性前缀的扩展。
```
vendor(prop, args)
    -webkit-{prop} args
    -moz-{prop} args
    {prop} args

  border-radius()
    vendor('border-radius', arguments)
  
  box-shadow()
    vendor('box-shadow', arguments)

  button
    border-radius 1px 2px / 3px 4px
```
编译成
```
button {
    -webkit-border-radius: 1px 2px / 3px 4px;
    -moz-border-radius: 1px 2px / 3px 4px;
    border-radius: 1px 2px / 3px 4px;
 }
```

### 选择器插值
插值也可以在选择器上使用，例如我们可以为表格的前5行指定高度。
```
table
  for row in 1 2 3 4 5
    tr:nth-child({row})
      height: 10px * row
```
编译成
```
table tr:nth-child(1) {
  height: 10px;
}
table tr:nth-child(2) {
  height: 20px;
}
table tr:nth-child(3) {
  height: 30px;
}
table tr:nth-child(4) {
  height: 40px;
}
table tr:nth-child(5) {
  height: 50px;
}
```
你可以通过创建一个字符串，将对个选择器合为一个变量，然后在同一个地方插入。
```
mySelectors = '#foo,#bar,.baz'

{mySelectors}
  background: #000
```
编译成
```
#foo,
#bar,
.baz {
  background: #000;
}
```

## 运算符（Operators）
### 运算符优先级
下面是运算符优先级表，由高到低。
```
 .
 []
 ! ~ + -
 is defined
 ** * / %
 + -
 ... ..
 <= >= < >
 in
 == is != is not isnt
 is a
 && and || or
 ?:
 = := ?= += -= *= /= %=
 not
 if unless
```

### 一元运算符
以下一元运算符是可用的，`!`, `not`, `-`, `+`, and `~`.
```
!0
// => true

!!0
// => false

!1
// => false

!!5px
// => true

-5px
// => -5px

--5px
// => 5px

not true
// => false

not not true
// => true
```

逻辑运算符not的优先级较低，因此，下面这个例子可以取而代之
```
a = 0
b = 1

!a and !b
// => false
// parsed as: (!a) and (!b)
```
使用
```
not a or b
// => false
// parsed as: not (a or b)
```

### 二元运算符
#### 下标[]
下标运算符允许我们从一个表达式获取值（从0开始），负值表示从末尾开始。
```
list = 1 2 3
 list[0]
 // => 1

 list[-1]
 // => 3
```
带括号的表达式可以当作元祖.
以下是使用元组进行错误处理的示例（并显示此构造的多功能性）
```
 add(a, b)
   if a is a 'unit' and b is a 'unit'
     a + b
   else
     (error 'a and b must be units!')

 body
   padding add(1,'5')
   // => padding: error "a and b must be units";
   
   padding add(1,'5')[0]
   // => padding: error;
   
   padding add(1,'5')[0] == error
   // => padding: true;

   padding add(1,'5')[1]
   // => padding: "a and b must be units";
```
这儿有个更复杂的例子。现在，我们调用内置的error()函数，当标识符（第一个值）等于error的时候返回错误信息。
```
if (val = add(1,'5'))[0] == error
  error(val[1])
```

### 范围......
同时提供包含界线操作符(..)和范围操作符(...)，见如下表达式。
```
 1..5
 // => 1 2 3 4 5

 1...5
 // => 1 2 3 4

 5..1
 // => 5 4 3 2 1
```
#### 加减
二元加乘运算其单位会转化，或使用默认字面量值。例如，5s - 2px结果是3s.
```
15px - 5px
// => 10px

5 - 2
// => 3

5in - 50mm
// => 3.031in

5s - 1000ms
// => 4s

20mm + 4in
// => 121.6mm

"foo " + "bar"
// => "foo bar"

"num " + 15
// => "num 15"
```

#### 乘除
```
2000ms + (1s * 2)
// => 4000ms

5s / 2
// => 2.5s

4 % 2
// => 0
```
当在属性值内使用/时候，你必须用括号包住。否则/会根据其字面意思处理（支持CSS的line-height）
```
font: 14px/1.5;
```
但是以下是14px÷1.5
```
font: (14px/1.5);
```