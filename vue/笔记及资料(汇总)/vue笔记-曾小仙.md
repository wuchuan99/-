node vscode git vue-cli webpack gulp

## Vue基础

> MVC：对项目的整体把控,M代表的是数据库中的数据,V代表的是前端的视图层，C用于处理M和V之间进行交互的业务逻辑(业务逻辑需要程序员自己控制，自己编写)

> MVVM:主要用于一些前端的框架,对MVC三层架构中的视图层再次进行层次划分，M是当前一个视图中需要用到的数据,V就是当前视图,VM负责M和V之间的数据调度,内部已经帮你完成了数据的绑定和交互

> MVC和MVVM之间的区别:MVC数据流通是单向的,MVVM是双向数据绑定

    M - C - V
    V - C - M

    M - VM(不经过VM进行逻辑处理也能直接更新视图) - V
    V - M
> 双向数据绑定的意思就是模型中的数据可以之间更新到视图上,视图中的数据发生改变也可以直接更新到模型中
能够做到双向数据绑定(通信)的原因:就是因为有VM的存在，VM内部的实现一般是框架已经处理完成,不需要程序员进行控制

### 系统指令

> `v-for循环渲染`

    <div v-for = '(item,index) in list' :key = 'index'></div>

### 过滤器

> `过滤器`:对需要展示的数据进行加工处理后再展示到界面，并不会改变原数据

- 全局过滤器

```JavaScript
Vue.filter('过滤器名称',function(value,arg){
    // value就是需要处理的原始数据
    // 对数据处理的业务逻辑
    return 处理完毕的数据
})

使用方法：原始数据 | 过滤器名称(123)
```

### 自定义指令

> `指令`: 其实就是Vue给我们提供的操作DOM元素的一些方法

- 全局指令

```JavaScript
Vue.directive('指令名称',{
    // 指令第一次绑定到元素身上时执行(在内存中绑定到了DOM对象上边)
    bind:function(el,binding,VNode){
        // el: 指令所在的DOM元素对象
        // binging.value 指令等号右边表达式的值
        // binging.expression 指令等号右边表达式
        // VNode.context 当前指令所在托管区域对于的Vue实例对象
    },
    // 当前指令所在元素插入到父节点时调用(不保证DOM已经渲染完毕)
    inserted:function(){},
    // 指令绑定的值发生变化时会去执行
    update:function(){},
    // 指令所在的组件值更新完毕时调用
    componentUpdated:function(){},
    // 自定义指令从当前元素解绑时调用
    unbind:function(){}
})
```

- 私有指令

```JavaScript
new Vue({
    el:'#app',
    // 私有指令
    directives:{
        '指令名称':{
            bind:function(el,binding){

            }
        }
    },
    // 私有过滤器
    filters:{
        '过滤器的名称':function(value){
            return 处理完毕的数据
        }
    }
})
```

## 生命周期钩子函数

> 钩子函数: 一个应用程序或者框架内部提前定义好的一批函数,这些函数会在特定的时间段自动执行
>
> 生命周期: 一个程序会存在初始化 - 运行 - 销毁等阶段

```
new Vue({
    el:'#app',
    data:{},
    methods:{},
    beforeCreated(){},
    // data中的数据和methods中的方法已经初始化完毕会去自动执行created方法
    created(){
        // 用于发生数据请求,也可以初始化一些数据
    },
    beforeMount(){},
    // 真实DOM已经渲染完毕会执行mounted函数
    mounted(){
        // 操作真实DOM
    }
    beforeUpdate(){},
    // data中的发生了变化而且被重新渲染到了界面上时才会执行
    updated(){
        // 数据更新后重新操作DOM
    },
    beforeDestroy(){},
    destroyed(){}
})
```


## vue-resource发生ajax请求

> vue.studyit.io

```javascript
// get请求
this.$http.get('url').then(function(res){
    // res.body 里面就是请求回来的数据
})

// post 请求 application/x-www-form-urlencode -> name=zs&age=18
// application/json => {'name':'zs','age':18}
this.$http.post('url',{name:'zs',age:18},{emulateJSON:true}).then(function(res){
    // res.body
})

// jsonp请求
this.$http.jsonp('url').then(function(res){
    // res.body
})
```

- **JSONP**:JSON数据格式的一种使用方式,用于解决跨域问题
- **跨域**: 由于浏览器同源策略的限制,不同源的URL不能互相通信,这种现象叫做跨域
- **同源策略**: 两个域名只有 **协议** **主机地址** **端口号**完全一致时才能相互通信访问
- JSONP实现原理：利用script标签的src属性发送请求不受同源策略的限制这个机制去实现跨域资源共享(script标签能够跨域访问是历史遗留问题,并不安全) 

    1. 动态创建一个script标签,并且将需要请求的URL地址设置给script标签的src属性,同时在URL地址后面拼接上一个回调函数名称
```
        function getJson(data){
            console.log(data)
        }
        <script src = 'http://lovegf.cn/jsonp?callback=getJson'> // getJson就是在前端本地定义好的一个方法的方法名
 ```   
    2. 后台接收到该请求后,先根据参数判断是否是JSONP请求,如果是,则将客户端请求的数据整理好,和前端发送过来的方法名一起拼接成一个函数调用的字符串,客户端需要的数据就是这个函数调用时传的参数
```
        // 假设要发送给前端的数据是 'ok'
        'getJson('ok')'
        // 响应给前端
        res.end('getJson('ok')')
 ```   
    3. 由于后台响应回来的数据是一个字符串,而且是函数调用,所以前端拿到响应回来的数据相当于调用了该方法,那么前端定义好的方法会自动执行,而且方法内的形参可以接收到实参值,也就是后台拼接的数据
```
        function getJson(data){
            console.log(data) // ok
        }
```
    4. 数据拿到之后,一般这个动态创建的script标签会被删除掉

## vue过渡动画

### 原生css类实现

```JavaScript
// 1. 将需要实现动画的元素使用transition标签包裹起来
<transition>
    <div>我是需要过渡动画的元素</div>
</transition>

// 2. 使用系统提供的类名书写动画样式
<style>
// v-enter: 动画进入时候的初始状态
// v-leave-to: 动画离开时候的结束状态
v-enter,v-leave-to {

}

// v-enter-active: 动画进入期间持续状态
// v-enter-active: 动画离开期间持续状态 
v-enter-active,v-leave-active {

}

// 动画离开时候的初始状态
// 动画进入时候的结束状态
v-leave,v-enter-to {

}

// 3. 也可以使用transition上面的name属性指定v-前缀
</style>
```
### animatie.css类库实现

```JavaScript
// 1. 将需要实现动画的元素使用transition标签包裹起来
<transition>
    <div>我是需要过渡动画的元素</div>
</transition>

// 2. 在transition上设置动画类名
<transition 
    enter-active-class="bounceIn" 
    leave-active-class="bounceOut" 
    :duration="{ enter: 200, leave: 400 }">
    <div>我是需要过渡动画的元素</div>
</transition>
```
### 动画钩子函数实现

```javascript
// 1. 将需要实现动画的元素使用transition标签包裹起来
<transition>
    <div>我是需要过渡动画的元素</div>
</transition>

// 2. 在transition上注册实现动画的钩子函数,入场动画三个,出场动画三个
<transition 
@before-enter = 'beforeEnter'
@enter = 'enter'
@after-enter = 'afterEnter'
@before-leave = 'beforeLeave'
@leave = 'leave'
@after-leave = 'afterLeave'
>
    <div>我是需要过渡动画的元素</div>
</transition>

// 3. 在vue实例的methods中定义钩子函数
new Vue({
    el:'#app',
    data:{},
    methods:{
        // 入场动画初始状态设置
        beforeEnter(){
            // 一般使用JS设置执行动画元素的初始位置和其他样式属性
        },
        // 入场动画持续过程
        enter(el,done){
            // 1. 固定写法 迫使浏览器重绘 刷新动画
            el.offsetWidth
            // 2. 一般设置执行动画元素最终的位置和其他样式属性

            // 3. 固定写法 调用afterEnter函数
            done()
        },
        // 入场动画结束状态
        afterEnter(){
            // 设置动画元素回到初始状态
        },
        // 出场动画初始状态
        beforeLeave(){

        },
        // 出场动画持续状态
        leave(){

        },
        // 出场动画结束状态
        afterLeave(){

        }
    }
})

```

### transition-group列表渲染

> 如果需要渲染的元素是一个列表,则不能使用transition包裹元素,需要使用transition-group

- `appear`属性可以使默认出现的元素出现过渡动画
- `tag`属性可以指定`transition-group`标签被渲染成一个什么元素,默认情况下`transition-group`会被渲染成`span`

```
<transition-group appear tag = 'div'>
    <div v-for = '(item,index) in list' :key = 'index'></div>
</transition-group>
```

## 组件

> http://blog.gdfengshuo.com/example/work/#/login

> https://github.com/UniverseKing/awesome-github-vue

> **注意**:
> 1. 定义组件时如果使用的是驼峰命名,那么使用组件时需要将驼峰的大写字母转成小写,并且用-连接两个单词 
```javascript
Vue.component('myCom',{
    template:'<div>我是一个驼峰命名的组件</div>'
})
// 使用
<my-com></my-com>
```
> 2. 组件的`template`属性中的模版内部如果有多个元素,必须被包含在唯一的一个根元素中
```javascript
<template>
    <div>
        <p>我是p元素</p>
        <span>我是span元素</span>
    </div>
</template>
```
> 3. 私有组件(子组件)使用`components`属性进行定义,定义好的私有组件只能在父组件的模板中使用
```javascript
<template id='father'>
    <div>
        <son></son>
    </div>
</template>
// 父组件
Vue.component('father',{
    template:'#father',
    components:{
        // 子组件
        son:{
            template:'<div>我是son组件</div>'
        }
    }
})
```

### 全局组件的三种定义方式

-  `vue.extend()`定义

```javascript
// 1. 使用vue.extend()定义组件模板
var dv = Vue.extend({
    template:'<div>我是一个组件</div>'
})

// 2. 使用Vue.component()注册组件
Vue.component('com',dv)

// 3. 在Vue托管区域像使用普通HTML标签一样使用组件
<com></com>

```
-  `vue.component()`定义

```javascript
// 1. 使用Vue.component定义组件
Vue.component('com',{
    template:'<div><p>我是一个组件中的元素</p><span>我也是组件中的元素</span></div>'
})

// 2. 使用组件
<com></com>
```

- 使用`template`标签定义模板

```javascript
// 1. 使用Vue.component定义组件，并且使用选择器选择模板
Vue.component('com',{
    template:'#temp'
}) 

// 2. 使用template标签定义模板,并且给template标签添加id
<template id='tmpl'>
    <div>
        <p>我是p元素</p>
        <span>我是span元素</span>
    </div>
</template>

// 3. 使用组件
<com></com>
```
### 私有组件的定义方式

> 私有组件使用`components`属性进行定义,该属性可以定义在Vue实例内部,也可以定义在组件内部,私有组件一般也是其他组件的子组件

```javascript
// Vue实例的私有组件
new Vue({
    el:'#app',
    data:{},
    methods:{},
    components:{
        'login': {
            template:'<div>我是一个私有组件,只能使用在当前#appVue实例托管区域内部</div>'
        }
    }
})


// 组件的私有组件(子组件)
<template id='account'>
    <div>
        我是账号组件,我有两个子组件login和register
        <login></login>
        <register></register>
    </div>
</template>
Vue.component('account',{
    tempalte:'#account',
    components:{
        'login':{
            template:'<div><com></com>我是登录组件,我的父组件是account</div>',
            components:{
                'com':{
                    template:'<div>123</div>'
                }
            }
        },
        'register':{
            template:'<div>我是注册组件,我的父组件是account</div>'
        }
    }
})
```
### is属性和component实现组件切换

```
// comname 是哪个组件名,则component就会被渲染成哪个组件
// component 就是一个占位标签 
<component :is='comname'></component>

new Vue({
    el:'#app',
    comname:'login'
})
```

### 父子组件传值

#### 父向子传值

```javascript
// 1. 先在父组件中定义好要传递的数据
new Vue({
    el:'#app'
    data:{
        msg:'我是要传递给子组件的数据'
    },
    components:{
        son:{
            template:'<div>我是子组件{{message}}</div>',
            props:['message']
        }
    }
})

// 2. 在父组件中使用子组件的时候,用绑定属性的方式将父组件中的数据传递给子组件
<div id='app'>
    <son v-bind:message = 'msg'></son>
</div>

// 3. 在子组件中定义一个props属性,该属性是一个数组,数组中定义用于接收传过来的变量。这个变量
// 和第二步绑定的这个属性同名
son:{
    template:'<div>我是子组件</div>',
    props:['message']
}

```

#### 子向父传值

> 发布订阅者 设计模式

```javascript
// 1. 定义好父子组件,定义好子组件需要传递给父组件的数据
new Vue({
    el:'#app'
    components:{
        son:{
            template:'<div>我是子组件{{message}}</div>',
            data:function(){
                return {
                    msg:'传给父组件的数据'
                }
            }
        }
    }
})

// 2. 在子组件中使用this.$emit触发一个自定义的方法名,然后传递数据
// 第一个参数就是自定义的方法名
// 第二个参数就是需要传递给父组件的数据
this.$emit('func',this.msg)

// 3. 在父组件中使用子组件时,绑定一个事件,事件名称和子组件触发的方法名同名
<div id='app'>
    <son @func = 'getmsg'></son>
</div>

// 4. 在父组件的methods中定义一个事件处理函数
methods:{
    getmsg:function(data){
        // data就是子组件传过来的数据
    }
}
```

### ref获取DOM和组件对象
```javascript
// 1. 在DOM元素或者组件上面绑定ref属性
<div ref='mydiv'>我是一个div</div>
<com ref = 'mycom'></com>
// 2. 使用this.$refs.属性值获取对象引用
this.$refs.mydiv // DOM对象
this.$refs.mycom // 组件对象
```
## vur-router路由

> **后端路由**: 服务端对前端请求的URL地址的监听,一个路由地址对应一个资源

> **前端路由**: 浏览器对前端页面实现的跳转方式,一般只在单页应用(SPA)程序中出现

> router  route routes  的区别是?
- router: 是Vue实例内部的一个属性,是专门用来挂载vue-router对象的
- route: 表示routers内部的每一个路由对象的引用
- routes: 是Vue-Router实例内部的一个属性,是用来保存所有的路由规则


### 基本使用步骤
```javascript
1. 创建路由对象并配置路由规则
// 1.1 创建组件模板对象
var login = {
    template:'<div>登录</div>'
}
var register = {
    template:'<div>注册</div>'
}
// 1.2 创建路由对象并配置路由规则
var router1 = new VueRouter({
    // routes用来存放路由规则,每个路由规则都是一个对象
    routes:[
        // path: 页面#后面的地址
        // component 需要显示的组件对象
        {path:'/',redirect:'/login'},
        {path:'/login',component:login},
        {path:'/register',component:register}
    ]
})

// 1.3 将创建处理的路由对象和Vue进行绑定
new Vue({
    el:'#app',
    // 将vuerouter对象注册到Vue内部,以保证在托管区域中科院使用VueRouter内部提供的组件以及其他属性和方法
    router:router1
})

2. 使用<router-view>进行占位
<div id='app'>  
    <router-view></router-view>
<div>

3. 使用<router-link>设置跳转路径
<router-link to='/login'>登录</router-link>
```
### redirect重定向

> 在路由规则中配置`redirect`属性可以使路由进行内部切换,当访问`path`中的路径时,会默认跳转到`redirect`中指定的路径去。

```JavaScript
{path:'/',redirect:'/login'} // 当访问根路径时,会默认跳转到login路径中

```
### linkActiveClass路由高亮
> `linkActiveClass` 用来设置被激活路由的样式类,默认类名是`router-link-active`

```JavaScript
new VueRouter({
    linkActiveClass:'mylink' // 被选中的路由对应的标签上面会有一个类名叫mylink
})
```

### 路由传参
#### query传参

```JavaScript
// 1. 在跳转路径后面使用查询字符串拼接参数
<router-link to = '/login?name=zs$age=18'>登录</router-link>

// 2. 在路由对应的组件内部使用`this.$route.query`获取参数
this.$route.query.name // zs
this.$route.query.id // 18
```
#### params传参

```JavaScript
// 1. 更改路由规则用于匹配参数
new VueRouter({
    routes:[
        // :id用于匹配/login/13后面的参数13
        {path:'/login/:id',component:login}
    ]
})
// 2. 在跳转路径后面使用/拼接参数
<router-link to = '/login/13'>登录</router-link>

// 3. 在路由对应组件内部使用`this.$route.params`获取参数
var login = {
    template:'<div>登录组件{{this.$route.params.id}}</div>',
    created(){
        console.log(this.$route.params.id) // 获取到的就是13
    }
}
```
### 嵌套路由
```JavaScript
// 1. 在父路由内部使用children属性配置子路由规则,子路由规则对象中的path属性不要 '/' 
// 2. 在父路由规则对应的组件模板中添加router-view用于显示子路由规则对应的组件
// 3. 在<router-link>中设置跳转路径(将父路由和子路由的path拼接)
```
### 路由命名视图

> 给`<router-view>`添加name属性,用于指定显示哪一个组件

```JavaScript
// 1. 将路由规则对象中的component改为components,表示一个路由规则下可以显示多个组件,每一个组件用一个名称对应起来,默认显示的组件用default对应
new VueRouter({
    routes:[
        // components对象内部按照 名称 - 组件对象的方式定义
        {path:'/',components:{
            'default': header,
            'left': leftBox,
            'main': mainBox
        }}
    ]
})

// 2. 给<router-view>添加name属性指定显示哪一个组件
<router-view></router-view>  // 不指定默认显示default对应的组件
<router-view name = 'left'></router-view> // 显示leftBox组件
<router-view name = 'main'></router-view> // 显示mainBox组件
```

## watch的使用

> `watch`: 用于监听data中定义的数据的变化,只要被监听的数据发生了变化,就会执行对应的处理函数

### watch监视数据变化
```
new Vue({
    el:'#app',
    data:{
        msg:'hello'
    },
    watch:{
        'msg': function(newVal,oldVal){
            // newVal 表示变化后的值 oldVal 表示变化前的值
            // 只要data中定义的msg发生了改变,就会执行这个函数
        }
    }
})
```
### watch监视路由变化

> 通过监听`$route.path`可以监视到路由地址的变化

```javascript
new Vue({
    el:'#app',
    watch:{
        '$route.path': function(newVal,oldVal){
           // newVal 新跳转的路由地址
           // oldVal 跳转之前的路由地址
        }
    }
})
```
## computed计算属性

> `computed`:计算属性,是指在computed对象中定义属性,属性的值是该属性后面函数的返回值,而且函数内部如果用到了data中定义的其他数据,只要这些数据发生了变化,都会重新触发函数的执行,计算属性的值也会发生变化。

```javascript
new Vue({
    el:'#app',
    data:{
        msg:'hello'
    },
    computed:{
        'info': function(){
            // 只要msg发生了变化,会重新触发该函数的执行
            // info的值就是函数的返回值
           return this.msg + 'world'
        }
    },
    methods:{},
    watch:{}
})
```

- npm: node package manager node包管理工具

- nrm: node registry manager node 镜像地址管理工具

- nvm: node version manager node 版本管理工具

## webpack的学习

> 什么是webpack? 一个基于Node的前端构建工具,可以实现对项目的打包(构建),主要解决文件(模块)之间的依赖,高级代码的转译,文件(模块)的合并和压缩等问题。

### 基本使用
 
- `webpack3.~`版本安装使用

```JavaScript
// 1. 全局安装webpack
npm i webpack@3.10.0 -g

// 2. 使用
webpack 入口文件 输出文件
webpack ./src/main.js ./dist/build.js
```

- `webpack4.~`版本安装使用

```JavaScript
// 1. 全局安装webpack
npm i webpack  -g
npm i webpack-cli -g

// 2. 使用
webpack 入口文件 -o 输出文件 --mode development/production
webpack ./src/main.js -o ./dist/build.js --mode development
```

### webpack配置文件使用

```javascript
// 1. 在项目根目录新建webpack.config.js,并设置 打包的入口文件 和 输出文件
module.exports = {
    // 入口文件(绝对路径)
    entry: path.join(__dirname,'./src/main.js'),
    // 输出文件
    output: {
        path: path.join(__dirname,'./dist'), // 输出文件目录
        filename: 'build.js' // 输出文件名称
    }
}

// 2. 执行打包命令
webpack
```

### webpack-dev-server的使用

> 

#### 命令行方式

> `--hot`: 热更新,可以局部更新文件
> `--open`: 自动打开浏览器
> `--port`: 配置端口号
> `--contentBase`: 指定服务器目录

```javascript
// 1. 局部安装webpack-dev-server和webpack
npm i webpack-dev-server webpack --save-dev

// 2. 在package.json的scripts节点中添加启动命令
"scripts":{
    "dev":"webpack-dev-server --open --hot --contentBase --port 8888"
}

// 3. 在项目根目录运行项目
npm run dev

```

#### 配置文件方式
```javascript
// 1. 在webpack.config.js配置文件中添加如下配置
devServer:{
    open: true,
    port: 8888,
    contentBase: 'src',
    hot: true
}

// 2. 如果设置了 hot:true 这个属性,则需要配置热更新插件
// 2.0 引入webpack模块
const webpack = require('webpack')
// 2.1 配置热更新插件
plugins: [
    new webpack.HotModuleReplacementPlugin()
]

// 3. package.json中配置启动命令
"scripts":{
    "dev":"webpack-dev-server"
}
```

### `html-webpack-plugin`的使用
> 可以根据指定的HTML模板文件生成一个HTML页面,并且在HTML页面中自动引入打包好的js文件

```javascript
// 1. 安装html-webpack-plugin
npm i html-webpack-plugin --save-dev

// 2. 在webpack.config.js中的plugins节点中配置插件
// 2.1 导入html-webpack-plugin
const htmlWebpackPlugin = require('html-webpack-plugin')
// 2.2 配置插件
plugins:[
    new htmlWebpackPlugin({
        template:path.join(__dirname,'./src/index.html'),// 需要生成的HTML的模板文件
        filename:'index.html' // 生成的HTML文件的名称
    })
]
```

### `css-loader`处理CSS文件
> 解析处理文件后缀名为.css的文件

```javascript
// 1. 安装style-loader css-loader
npm i style-loader css-loader --save-dev

// 2. 在webpack.config.js中的moudle节点中配置解析规则
module:{
    rules:[
        {
            test:/\.css$/,
            use:['style-loader','css-loader']
        }
    ]
}
```
### `less-loader`处理less文件
> 解析处理文件后缀名为.less的文件

```javascript
// 1. 安装less less-loader
npm i less less-loader --save-dev

// 2. 在webpack.config.js中的moudle节点中配置解析规则
module:{
    rules:[
        {
            test:/\.less$/,
            use:['style-loader','css-loader','less-loader']
        }
    ]
}
```
### `scss-loader`处理scss文件
> 解析处理文件后缀名为.scss的文件

```javascript
// 1. 安装node-sass sass-loader
npm i node-sass sass-loader --save-dev

// 2. 在webpack.config.js中的moudle节点中配置解析规则
module:{
    rules:[
        {
            test:/\.scss$/,
            use:['style-loader','css-loader','sass-loader']
        }
    ]
}
```
### `url-loader`处理图片等资源文件
> 解析处理项目中引入的图片、字体图标、音视频等资源文件

```javascript
// 1. 安装file-loader ulr-loader
npm i file-loader ulr-loader --save-dev

// 2. 在webpack.config.js中的moudle节点中配置解析规则
module:{
    rules:[
        {
            test:/\.(png|jpg|gif)/,
            use:'url-loader?limit=20000&name=[hash:8]-[name].[ext]'
            // 当需要打包的文件小于limit的值,会将图片转换成base64字符串在build.js中进行引用
            // 当需要打包的文件大于limit的值,会将图片打包到输出目录中
            //一般比较小的图片才需要被转成base64,所以该值不宜太大

            // name=[hash:8]-[name].[ext]表示将来打包出来的文件名称以8位hash值和原来的名称进行拼接,文件扩展名不变
        }
    ]
}
```

### `babel`的使用

> 解析webpack默认不能处理的JS

```JavaScript
// 1. 安装babel-core babel-loader babel-plugin-transform-runtime
npm i babel-core babel-loader babel-plugin-transform-runtime --save-dev

// 2. 安装babel-preset-env babel-preset-stage-0
npm i babel-preset-env babel-preset-stage-0 --save-dev

// 3. 在webpack.config.js中的moudle节点中配置解析规则
module:{
    rules:[
        {
            test:/\.js$/,
            use:'babel-loader',
            exclude:/node_modules/ //排除掉node_modules文件夹,让babel-loader不去解析node_modules文件夹中的js文件
        }
    ]
}

// 4. 在项目根目录新建.babelrc文件,书写如下配置
{
    "presets":["env","stage-0"],
    "plugins":["transform-runtime"]
}

```

## webpack中使用Vue

```JavaScript
// 1. 安装vue vue-loader vue-template-compiler
npm i vue --save
npm i vue-loader vue-template-compiler --save-dev

// 2. webpack.config.js中配置解析规则
module: {
    rules: [
        {
            test:/\.vue$/,use:'vue-loader'
        }
    ]
}

// 3. 在index.html中加入一个需要Vue托管的区域
<div id="app"></div>

// 4. 新建App.vue组件
<template>
    <div>vue组件</div>
</template>
<script>
export default {

}
</script>
<style>
</style>

// 5. main.js中导入vue的包,渲染App.vue组件
import Vue from 'vue'
import App from './App.vue'

new Vue({
    el:'#app',
    render:c=>c(App) //将App组件渲染到id为app的区域中
})

```

## webpack 中集成vue-router

```JavaScript
// 1. 安装vue-router
npm i vue-router --save

// 2. 在main.js中导入vue-router并实例化路由添加路由规则
import VueRouter from 'vue-router'
Vue.use(VueRouter) // 保证在vue的组件中可以使用vue-router提供的组件 如<router-view> <router-link>

const router = new VueRouter({
    routes: [
        {
            path: '/login', component: login
        }
    ]
})

// 3. 将实例化的router绑定到Vue
new Vue({
    el: '#app',
    render: c => c(App),
    router // 将router实例绑定到Vue上 以保证vue实例中可以访问$route $router等属性
})

// 4. 在App.vue中添加路由占位router-view标签
<router-link to = "/login">登录<router-link>
<router-view></router-view>
```
--- 

## --save和--save-dev
```
// 项目开发期间需要安装的依赖
// --save-dev 会将安装的包名以及版本号记录到package.json文件中的devDependencies节点中
npm install webpack --save-dev 等价于 npm install webpack -D

// 项目上线后需要安装的依赖
// --save 会将安装的包名以及版本号记录到package.json文件中的dependencies节点中
npm i webpack --save 等价于 npm i webpack -S

npm i 会将所有在package.json文件中的devDependencies节点和dependencies节点内所有的依赖包全部安装一遍
npm i --save-dev 只会下载devDependencies中的所有依赖包
npm i --save 只会下载dependencies中的所有依赖包

// 为什么安装包的时候要区分 --save 和 --save-dev
// 1. 使项目的阅读更加友好，便于理解项目结构
// 2. 安装对应阶段的包时更加方便

npm i cnpm -g

cnpm 
```

## VSCode用户代码片段
```json
"Print to vue": {
		"prefix": "vuec",
		"body": [
			"<template>",
			"    <div>",
			"      御剑乘风来,除魔天地间!$1",
			"    </div>",
			"</template>",
			"           ",
			"<script>",
			"export default{",
			"    $2         ",
			"}",
			"</script>",
			"<style scoped>",
			"$3            ",
			"</style>"
		],
		"description": "create a vue template"
	}
```

## yarn的使用

```
// 1. 使用npm全局安装yarn
npm i yarn -g

// 2. 初始化命令
yarn init -y

// 3. 安装包
yarn add package-name // 会将安装的包记录到dependencies节点里面
yarn add package-name --dev // 会将安装的包记录到devDependencies节点里面

yarn global add package-name // 安装全局包

// 4. 删除包
yarn remove package-name

// 5. 执行命令
yarn run dev

yarn global bin 查看全局包安装的路径
```

## vue-cli脚手架的使用

```
// 1. 安装vue-cli脚手架
npm i vue-cli -g

// 2. 初始化项目模板
vue init webpack 项目名称
eslint(语法规范化插件) 不要安装 
e2e(测试框架) 不要安装
unit test(单元测试框架) 不要安装

// 3. 进入项目安装所有依赖
npm i

// 4. 运行
npm run dev

config/index.js中 17/18行改端口号和自动打开浏览器
```

## 将项目提交到码云

```
// 1. 在项目文件夹初始化git
git init

// 2. 将代码提交到暂存区
git add .

// 3. 提交代码
git commit -m '描述信息'

// 4. 关联远程分支
git remote add origin https://gitee.com/UniverseKing/tes.git

// 5. 推送到远程分支
git push -u origin master

// 6. 查看文件信息
git status

// 7. 查看log
git log --oenline

// 8. 切换版本记录
git reset --hard 版本号
```

## vue-loader深度作用选择器

> 如果希望在父组件中去改变子组件中的样式,有时候通过普通子类选择器无法实现,主要是由于添加了`scoped`属性导致,`vue-loader`中提供了[深度作用选择器](https://vue-loader.vuejs.org/zh-cn/features/scoped-css.html)可以实现

> 这种问题主要出现在使用第三方组件时,需要改第三方组件的样式时

```CSS
/*假设.content是父组件中的类 img是子组件中的标签*/
/*css中使用>>>*/
.content >>> img {

}
/*scss或者less中使用/deep/*/
.content /deep/ img {

}
```

## Promise

> `Promise` 是异步编程的一种解决方案.简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，`Promise` 是一个对象，可以从该对象获取异步操作的消息。

### 基本用法

	// 1. 创建promise实例,在实例中执行异步操作(比如发送网络请求)
	// 2. 异步操作成功时,调用reslove函数传递数据
	// 3. 异步操作失败时,调用reject函数传递错误信息
	const promise = new Promise(function(resolve, reject) {
		// 异步操作
		// ... 
		if (/* 异步操作成功 */){
			resolve(value);
		} else {
			reject(error);
		}
	});
	
	// 4. 使用promise实例then方法接收reslove或reject返回的数据
	promise.then(function(value) {
		// 此处数据即为reslove回来的数据
  		// success
	}, function(error) {
		// 此处数据即为reject回来的数据
		// failure
	});
	
### 网络请求案例

	// 1. 定义一个使用promise封装的请求函数,函数内部返回一个promise实例
	function fetch(){
		// 函数内部返回一个promise实例
		return new Promise(function(reslove,reject){
			// 发送异步请求
			axios.get('http://www.lovegf.cn:8090/api/getlunbo').then(function(res){
				// 请求正常
				if(res.status == 1){
					reslove(res.data)
				}else{
					reject(res.error)
				}
			})
		})
	}
	
	// 2. 调用函数发送请求,通过Promise.prototype.then方法获取resolve或reject出来的数据
	fetch().then(function(res){
		// res为reslove函数传出的数据
	},function(err){
		// err为reject函数传出的错误
	})
	
### 解决回调地狱

> 假设有三个请求A、B、C,B请求需要依赖A请求的数据,C请求需要依赖B请求的数据.
> 
> 传统回调函数式写法如下:

	function dependices_fetch(){
		// A请求
		axios.get('A').then(function(res){
			if(res.status == 1){
				// B请求
				axios.get('B').then(function(res){
					if(res.status == 1){
						// C请求
						axios.get('C').then(function(res){
							// 请求完毕,执行后续逻辑
						})
					}
				})
			}
		})
	}

> 这种代码虽然能够满足业务,但是代码组织结构非常不便于阅读
> 
> 通过Promise可以封装代码,使用链式方式解决这种多个异步依赖的回调
> 
> 如下:

	function fetch(url){
		return new Promise(function(reslove,reject){
			axios.get(url).then(function(res){
				if(res.status == 1){
					reslove(res.data)
				}else{
					reject(res.error)
				}
			})
		})
	}
	
	//then方法内部返回的promise实例reslove或reject出来的对象会在下一个then方法内部得到
	fetch('A').then(function(res){
		// A 请求正常
		return fetch('B')	// 这里返回一个新的promise实例,在后面的then中可以得到该实例reslove或reject出来的对象
	}).then(function(res){
		// B 请求正常
		return fetch('C')
	}).then(function(res){
		// C 请求正常
		// 请求完毕
	})
	
### 多个异步请求结果组合问题

> 假设有A、B、C三个异步请求,需要三个请求的数据都回来之后,将数据整合后再渲染页面,这种需求可以使用`Promise.all()`
> 
> Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

	function fetch(url){
			return new Promise(function(reslove,reject){
				axios.get(url).then(function(res){
					if(res.status == 1){
						reslove(res.data)
					}else{
						reject(res.error)
					}
				})
			})
	}
	
	const p1 = fetch('A')
	const p2 = fetch('B')
	const p3 = fetch('C')
	const p = Promise.all([p1, p2, p3]);
	p.then(function(res){
		// res是一个数组,存放着p1,p2,p3的返回值
	})

## async 和 await 的使用

> async 和 await 是ES7中新提供的两个操作异步函数的关键字,本质是对Promise进一步的封装

> await 只能用在 async 定义的函数内部,await 可以等待异步操作,并同步的得到异步操作的返回值

```JavaScript
// 1. 基于Promise的异步操作函数
const fs = require('fs')
function getFileByPath(fpath) {
    return new Promise(function (resolve, reject) {
        fs.readFile(fpath, 'utf-8', (err, dataStr) => {

            if (err) return reject(err)
            resolve(dataStr)

        })
    })
}

// 2. 基于 await 的异步依赖问题(异步操作3依赖异步操作2,异步操作2依赖异步操作1)
// 2.1 先使用async定义异步操作
const start = async function(){
// 2.2 在async内部使用await 同步操作    
    const n1 = await getFileByPath('./1.txt')
    const n2 = await getFileByPath('./2.txt')
    const n3 = await getFileByPath('./3.txt')

    // n3会等待n2 n2 会等待n1
}
```

## axios 使用

```JavaScript
1. 安装axios
npm i axios qs --save

2. 在main.js中导入axios,并将axios 挂载到Vue的原型中
improt axios from 'axios'
Vue.prototype.$axios = axios 

3. get请求

this.$axios.get('url').then(function(res){
    // 注意:
    // 1. 响应的数据在res.data中
    // 2. 在回调函数内部的this 不是Vue实例(通常会将这个回调函数改成箭头函数)
})

4. post 请求
// 注意:当后台要求传递的参数数据格式是 application/x-www-form-urlencode
// 需要对参数进行序列化
// 使用qs模块序列化qs.stringify({name:'zs',age:18})
// 如果后台要求的参数格式 是json格式 appliction/json 不需要序列化
this.$axios.post('url',qs.stringify({name:'zs',age:18})).then(res=>{

})

5. 全局配置
// 5.1 设置全局的请求根域名
axios.defaults.baseURL = 'http://vue.studyit.io/';
// 5.2 通过设置请求拦截器 配置post参数序列化
axios.interceptors.request.use(function (config) {
    // 统一处理post请求的参数 为application/x-www-form-urlencode
    if (config.method == 'post') {
        config.data = qs.stringify(config.data)
    }
    return config;
});
```

## Vue插件开发

```JavaScript
1. 单独新建一个JS文件,在这个文件中导入需要制作成Vue插件的模块
import axios from 'axios'

2. 在该模块身上定义一个install函数,可以接受两个参数,第一个用于接收Vue构造函数
第二个用于配置插件的一些选项
axios.install = function(Vue,config){
    // 1. 注册全局组件
    // 2. 将属性挂载到原型中
}
3. 导出制作好的Vue插件模块
export default axios

4. 使用插件
// 能够使用Vue.use的前提就是传入的对象一定要有install方法
// 调用Vue.use()的时候 会自动去调用 传入对象的 install方法
import axios from '插件路径'
Vue.use(axios) // 会自动调用axios的install方法

```

## 去除webpack打包后的严格模式

> 在使用babel-loader的时候,会将所有转换的代码加上严格模式

```JavaScript
// 方法一: .babelrc文件中忽略不需要使用严格模式转换的文件路径
 "ignore": [
    "./src/js/mui/mui.min.js"
  ]

// 方法二: babel-loader配置中排除掉不需要严格模式转换的文件
{ 
    test: /\.js$/,
    use: 'babel-loader',
    exclude: /mui\.min\.js/
}

// 方法三: babel-plugin-transform-remove-strict-mode 移除整个项目打包编译时的严格模式
// https://www.npmjs.com/package/babel-plugin-transform-remove-strict-mode
1. 安装babel-plugin-transform-remove-strict-mode
npm install babel-plugin-transform-remove-strict-mode --save-dev

2. babelrc中添加
{
  "plugins": ["transform-remove-strict-mode"]
}
```
## 路由模式

> 通过给`vue-router`配置`mode`属性,可以指定URL路径的显示方式

> `mode`属性默认值是`hash`,此时URL中有#,如:http://localhost:8888/#/home,实现方式即`window.location.hash`

> 如果`mode`设置成`history`,此时URL路径没有#,如:http://localhost:8888/home,实现方式为`window.history`,这种方式同一个URL不能再其他页面打开,需要服务端配置

```JavaScript
new VueRouter({
    mode:'hash',
    routes:[

    ]
})
```

## Vuex的使用

> Vuex是一个状态管理库,或者说是专为Vue应用程序开发设计的状态管理模式,它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

> 注:所谓状态,可以理解成项目中各个组件需要用到的数据。

> Demo:https://gitee.com/UniverseKing/vuex-study

### 初始化公共状态

``` JavaScript
1. 安装vuex
npm i vuex --save

2. 入口文件中实例化Store
import Vuex from 'vuex'
Vue.use(Vuex)
var store = new Vuex.Store({
  // 1. 用于定义状态(公共数据),类似于Vue实例中的data方法
  state:{
      msg:'初始化的数据'
  },
  // 2. 用于修改状态,类似于Vue实例中methods
  mutations:{
      change(state,arg){
          // 更改状态
          state.msg = arg
      }
  },
  // 3. 用于获取数据(获取数据之前可以进行一些操作),类似于Vue实例中的过滤器和计算属性
  // getters 主要会用在跨组件传值
  // getters 中定义的方法内部依赖的数据发生变化会自动重新调用函数计算返回值
  getters:{
      fixmsg(state){
          return `${state.msg}----处理后的数据`
      }
  },
  // 4. actions和mutations都是定义对数据进行操作的方法,mutations中都是同步方法,mutations中定义异步方法
  // Action 提交的是 mutation，而不是直接变更状态。所以需要修改状态还是需要使用mutations中定义的方法

  // 从网络请求回来的数据需要保存到store
  // 发送网络请求的方法可以定义到actions中
  // actions主要用于处理异步方法
  actions:{
      asyncchange(context,arg){
          // 异步方法
          setTimeout(() => {
              context.commit('change',arg)
          }, 3000)
      }
  }
})

3. 注入到Vue实例中
new Vue({
    el:'#app',
    store
})
```

### 使用状态

``` JavaScript
1. 使用state中的数据
JavaScript: this.$store.state.msg
HTML: $store.state.msg

2. 使用getters中的数据
JavaScript: this.$store.getters.fixmsg
HTML: $store.getters.msg
```

### 变更状态(修改数据)

> 状态的变更必须使用mutations中提供的方法进行修改

``` JavaScript
1. 提交mutations中的变更方法
this.$store.commit('change','我是被修改的数据')

2. 异步提交actions中的变更方法
this.$store.dispatch('asyncchange','我是被异步修改的数据')
```

### 使用辅助函数

> 辅助函数可以直接将`state`,`getters`中的数据映射到Vue组件中的计算属性上,可以将`mutations`,`actions`中的方法映射到组件中的`methods`中

``` JavaScript
import { mapState } from 'vuex'
import { mapGetters } from 'vuex' 
import { mapMutations } from 'vuex'
import { mapActions } from 'vuex'

new Vue({
    computed:mapGetters([
        'count'
    ])
})
// ==>等价于
new Vue({
    computed:{
        count(){
            return this.$store.state['count']
        }
})
```

## vue-cli配置代理解决开发阶段跨域问题

```JavaScript
// webpack.config.js配置文件导出对象中加入
 devServer:{
    // 代理跨域
    proxy: {
      '/api': {
        target: 'http://vue.studyit.io/api', // 需要被代理的api根域名
        changeOrigin: true, // 是否跨域
        pathRewrite: {
          '^/api': '' // 重写(target中只有根域名时不需要配置此选项)
        }
      }
    }
  }
---------------------------
// vue-cli中
config/index.js中
proxyTable: {
    '/api': {
        target: 'http://vue.studyit.io/api',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
    }
},
```

## webpack打包优化

> 目前项目中所有资源都会被打包到最终生成的一个build.js中,而build.js又被引用到index.html中,当打开index.html时,首先需要从互联网下载build.js,只有build.js全部下载完毕,浏览器的解析引擎才会开始执行该js代码进行代码解析,当解析引擎将代码解析完毕后再交给渲染引擎开始进行渲染,所有工作几乎都是同步执行,当build.js体积非常大时,往往应用的首页加载速度比较慢,而且会看到白屏效果.可以从以下三方面进行优化首页加载速度过慢问题:

1. 下载文件速度慢

2. 解析JS比较慢

3. 渲染比较慢

### 服务端渲染(SSR)

> SEO优化

> 在服务端先将JS进行解析,解析完成之后生成HTML片段,再将HTML返回给前端,前端直接执行渲染界面

### 抽取CSS

> 将项目中所有CSS抽取到单独的文件中进行加载,减小了build.js的体积,只要build.js下载完成即可开始解析代码

> https://github.com/webpack-contrib/extract-text-webpack-plugin

```JavaScript
1. 安装插件

npm install extract-text-webpack-plugin --save-dev

2. webpack.config.js中引入插件

var ExtractTextPlugin = require('extract-text-webpack-plugin');

3. 修改css-loader配置
{
    test: /\.css$/,
    use: ExtractTextPlugin.extract({
        fallback: "style-loader",
        use: "css-loader"
    })
},
{
    test: /\.less$/,
    use: ExtractTextPlugin.extract({
        fallback: 'style-loader',
        use: ['css-loader', 'sass-loader']
    })
}, // 处理 less 文件的 loader
{
    test: /\.scss$/,
    use: ExtractTextPlugin.extract({
        fallback: 'style-loader',
        use: ['css-loader', 'sass-loader']
    })
}

4. plugin配置中使用插件
new ExtractTextPlugin('app.css')
```

### 分离第三方包

> 将项目中引用的第三方JS代码抽取到一个单独的文件,也可以减少build.js的体积,还是只需要build.js下载完毕浏览器解析引擎即可开始进行解析,然后浏览器同时再去下载其他JS和CSS

> 一般将发布阶段所需要依赖的包全部进行分离,即`package.json`中`dependencies`中的所有包进行分离

```JavaScript
 1. 引入webpack和路径处理模块
var webpack = require('webpack');
var path = require('path');

2. 修改入口文件
entry: {
    app: path.resolve(__dirname, 'src/main.js'),
    // 需要分离的第三方包名写在数组中
    vendors: ['vue', 'vue-resource', 'vue-router', 'vuex', 'mint-ui', 'moment', 'vue-preview']
},

3. plugin中配置插件
// 分离第三方包插件
new webpack.optimize.CommonsChunkPlugin({
    name: 'vendors',
    filename: 'vendors.js' // 分离出来的js文件的名称
})
```

### 组件按需加载

> vue-router提供的路由懒加载可以按需加载组件。

> 特点: 当没有访问到某个页面路由时,不去加载对应的组件代码,节约数据请求量,加快首页DOM渲染速度

```
将import home from './components/Home.vue' 这种导入方式换成
const home = () => import('../components/Home.vue');
```

## 路由导航钩子函数

> 在跳转路由时,进行拦截,一般用于权限验证或登录等操作

> https://router.vuejs.org/zh-cn/advanced/navigation-guards.html

### 路由独享守卫

```JavaScript
const router = new VueRouter({
  routes: [
    {
      path: '/home',
      component: HomeContainer,
      // 进入路由之前进行拦截
      beforeEnter: (to, from, next) => {
        // from 从哪个路由来
        // to 到哪个路由去
        // 释放守卫 进入下一个环节
      }
    }
  ]
})
```

### 全局守卫

```JavaScript
router.beforeEach((to, from, next) => {
  
})
```

## HBuilder打包

> HBuilder打包 是将前端开发的HTML/CSS/JS文件进行打包,打包成可以直接安装在手机上面的App,不借助浏览器就可以直接运行

```JavaScript
1. 使用webpack将项目进行打包,打包好的文件会在dist目录
webpack -p

2. 打开Hbuilder,新建移动App项目,然后删掉css/js/imgs文件夹和index.html文件

3. 将使用webpack打包的dist目录里面的文件全部拷贝到新建的移动项目目录中

4. 选中HTML文件,点击HBuilder导航上面的`发行`-`打原生安装包`,直接下一步,选安卓公共证书-`打包`

5. 在项目的`unpackage\release`中可以得到打包完毕的apk文件
```

## cordova打包

> 直接在本地进行打包

> 需要电脑配置Java环境和Android环境 `gradle`

```JavaScript
// 1. 安装全局cordova
npm i cordova -g

// 2. 创建cordova项目
cordova create cordova_project

// 3. 添加需要打包的平台
cordova platform add android

// 4. 运行或者打包
cordova run android
cordova build android

```

## vue-cli打包完成后

1. 错误信息:`postcss-svgo: Error in parsing SVG: Unquoted attribute value`

    - 需要修改mui.css中的 **图片引用** 单引号全部改成双引号

2. 提示必须使用服务器的方式打开,即使用localhost方式打开

        // 1. 安装http-server
        npm i http-server -g
        // 2. 使用hs -o命令打开index.html文件
        hs -o

3. 如果打包完成后希望直接用file协议打开,需要修改config/index.js 中的build下的`assetsPublicPath: '/'`换成`assetsPublicPath: './'`

## Parcel使用

```JavaScript

npm i parcel-bundler -g

parcel index.html
```

----

> 项目地址:https://gitee.com/UniverseKing/vue_cms

> [Vue组件](https://github.com/UniverseKing/awesome-github-vue#%E5%BA%94%E7%94%A8%E5%AE%9E%E4%BE%8B)
