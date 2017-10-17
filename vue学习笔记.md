### vue学习笔记

#### 预热工作
从前天开始一路折腾搭建一个vue项目，过程真是艰辛到非常。
因为我使用的是Atom编辑器，安装插件真是一个巨大的坑，正常网络状态下一直安装不成功，可能是因为墙的关系，后来挂了代理照样安装不成功，蜜汁尴尬。之后修改了`.atom\.apmrc`文件，结合代理才解决问题。不过Atom的`setting`安装插件依旧不成功，需要使用apm才行。(前提安装最新版本的`node.js`)

#### 准备工作
1. 安装node.js
```
node -v //查看node版本以验证是否安装成功
```
2. 全局安装vue
```
npm install -g vue
vue -V //查看vue版本，注意V要大写
```
这里使用npm安装速度较慢，还可能会失败，可以使用`cnpm`进行安装

3. 全局安装vue-cli
```
npm install -g vue-cli
```

4. 安装webpack
```
npm install -g webpack
```
使用vue-cli快速构建一个vue项目
```
cd <your project filepath>
vue init webpack <your project name>
```
这里有一个问题，就是构建出的项目中只有`src`文件夹，其余的文件夹都没有，一开始我以为是`webpack`的配置问题，但是没有`build`文件夹，这个问题没搞懂。后来想结合`vue-router`进行开发，于是安装了`vue-router`重新构建项目，构建完之后项目就变的正常了，真特么玄学解决问题。

5. 构建完项目之后
```
cd <your project>
npm install //安装项目所需要的依赖包
npm run dev //运行项目
```

项目中得到`router`文件夹用来配置路由，所有组件应该通过路由文件配置指向何处，最后将所有组件挂载在根节点上

在`router`文件夹中的`index.js`中进行配置，`import`需要用到的组件，然后通过`path`配置路径，注意`name`必须要一致。

#### 关于在项目中使用less
在项目中使用`less`,现在项目所在目录执行
```
npm install less less-loader --save-dev
```
安装完成之后，在`webpack`中进行配置
```
{
    test:/\.css$/,
    loader:'style-loader!css-loader'
},
{
    test:/\.less$/,
    loader:'style-loader!css-loader!less-loader'
}
```

#### 使用vuex进行组件间的状态管理
在项目所在目录执行
```
npm install vuex --save-dev //用于开发环境
npm install vuex --save //用于生产环境
npm install vuex -g //全局安装
```
安装完成之后在项目的`main.js`中全局引入vuex
```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```