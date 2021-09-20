---
highlight: darcula
theme: smartblue
---

# 为什么 Vue2 `this.xxx` 直接获取到 data 里的数据

## 1. 前言

>大家好，我是[若川](https://lxchuan12.gitee.io)。欢迎关注我的[公众号若川视野](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/13/16efe57ddc7c9eb3~tplv-t2oaga2asx-image.image "https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/13/16efe57ddc7c9eb3~tplv-t2oaga2asx-image.image")，最近组织了[**源码共读活动**《1个月，200+人，一起读了4周源码》](https://juejin.cn/pin/7005372623400435725)，感兴趣的可以加我微信 [ruochuan12](https://juejin.cn/pin/7005372623400435725) 加微信群参与，长期交流学习。

之前写的[《学习源码整体架构系列》](https://juejin.cn/column/6960551178908205093) 包含`jQuery`、`underscore`、`lodash`、`vuex`、`sentry`、`axios`、`redux`、`koa`、`vue-devtools`、`vuex4`十余篇源码文章。

>写相对很难的源码，耗费了自己的时间和精力，也没收获多少阅读点赞，其实是一件挺受打击的事情。从阅读量和读者受益方面来看，不能促进作者持续输出文章。
>所以转变思路，写一些相对通俗易懂的文章。**其实源码也不是想象的那么难，至少有很多看得懂**。歌德曾说：读一本好书，就是在和高尚的人谈话。同理可得：读源码，也算是和作者的一种学习交流的方式。

本文源于一次[源码共读](https://juejin.cn/pin/7005372623400435725)群里群友的提问，请问@若川，“**为什么 data 中的数据可以用 this 直接获取到啊**”，当时我翻阅源码做出了解答。想着如果下次有人再次问到，我还需要回答一次。当时打算有空写篇文章告诉读者自己探究原理，于是就有了这篇文章。

阅读本文，你将学到：

```js
1. 如何学习调试 vue2 源码
2. data 中的数据为什么可以用 this 直接获取到
3. methods 中的方法为什么可以用 this 直接获取到
4. 如何学习源码中优秀代码和思想，投入到自己的项目中
```

大胆猜测，小心求证。

## this.xxx 获取到

众所周知，

```js
const vm = new Vue({
    data: {
        name: '我是若川',
    },
    methods: {
        sayName(){
            console.log(this.name);
        }
    },
});
console.log(vm.name); // 我是若川
console.log(vm.sayName()); // 我是若川
```

那么为什么 this.xxx 能获取到`data`里的数据。

```js
function Person(){

}

const p = new Person({
    data: {
        name: '若川'
    },
    methods: {
        sayName(){
            console.log(this.name);
        }
    }
});

console.log(p.name);
// undefined
console.log(p.sayName());
// Uncaught TypeError: p.sayName is not a function
```

## 准备环境调试源码一探究竟

可以在本地新建一个文件夹`examples`，新建文件`index.html`文件。
在`<body></body>`中加上如下`js`。

```js
<script src="https://unpkg.com/vue@2.6.14/dist/vue.js"></script>
<script>
    const vm = new Vue({
        data: {
            name: '我是若川',
        },
        methods: {
            sayName(){
                console.log(this.name);
            }
        },
    });
    console.log(vm.name);
    console.log(vm.sayName());
</script>
```

再全局安装`npm i -g http-server`启动服务。

```js
npm i -g http-server
cd examples
http-server .
// 如果碰到端口被占用，也可以指定端口
http-server -p 8081 .
```

这样就能在`http://localhost:8080/`打开刚写的`index.html`页面了。

### new Vue

```js
function Vue (options) {
    if (!(this instanceof Vue)
    ) {
        warn('Vue is a constructor and should be called with the `new` keyword');
    }
    this._init(options);
}
// 初始化
initMixin(Vue);
stateMixin(Vue);
eventsMixin(Vue);
lifecycleMixin(Vue);
renderMixin(Vue);
```

值得一提的是：`if (!(this instanceof Vue)){}` 判断是不是用了 `new` 关键词调用构造函数。
一般而言，我们平时应该不会考虑写这个。

当然看源码库也可以自己函数内部调用 `new` 。但 `vue` 一般一个项目只需要 `new Vue()` 一次，所以没必要。

而 `jQuery` 源码的就是内部 `new` ，对于使用者来说就是无`new`构造。

```js
jQuery = function( selector, context ) {
  // 返回new之后的对象
  return new jQuery.fn.init( selector, context );
};
```

因为使用 `jQuery` 经常要调用。
其实 `jQuery` 也是可以 `new` 的。和不用 `new` 是一个效果。

如果不明白 `new` 操作符的用处，可以看我之前的文章。[面试官问：能否模拟实现JS的new操作符](https://juejin.cn/post/6844903704663949325)

### _init 初始化函数

```js
// 代码有删减
function initMixin (Vue) {
    Vue.prototype._init = function (options) {
      var vm = this;
      // a uid
      vm._uid = uid$3++;

      // a flag to avoid this being observed
      vm._isVue = true;
      // merge options
      if (options && options._isComponent) {
        // optimize internal component instantiation
        // since dynamic options merging is pretty slow, and none of the
        // internal component options needs special treatment.
        initInternalComponent(vm, options);
      } else {
        vm.$options = mergeOptions(
          resolveConstructorOptions(vm.constructor),
          options || {},
          vm
        );
      }

      // expose real self
      vm._self = vm;
      initLifecycle(vm);
      initEvents(vm);
      initRender(vm);
      callHook(vm, 'beforeCreate');
      initInjections(vm); // resolve injections before data/props
      //  初始化状态
      initState(vm);
      initProvide(vm); // resolve provide after data/props
      callHook(vm, 'created');
    };
}
```

### initState 初始化状态

```js
function initState (vm) {
    vm._watchers = [];
    var opts = vm.$options;
    if (opts.props) { initProps(vm, opts.props); }
    // 有传入 methods，初始化方法
    if (opts.methods) { initMethods(vm, opts.methods); }
    // 有传入 data，初始化 data
    if (opts.data) {
      initData(vm);
    } else {
      observe(vm._data = {}, true /* asRootData */);
    }
    if (opts.computed) { initComputed(vm, opts.computed); }
    if (opts.watch && opts.watch !== nativeWatch) {
      initWatch(vm, opts.watch);
    }
}
```

`initState` 函数中根据一些传入的参数进行初始化。

这里我们先看初始化 `methods`，之后再看初始化 `data`。

### initMethods 初始化方法

```js
function initMethods (vm, methods) {
    var props = vm.$options.props;
    for (var key in methods) {
      {
        if (typeof methods[key] !== 'function') {
          warn(
            "Method \"" + key + "\" has type \"" + (typeof methods[key]) + "\" in the component definition. " +
            "Did you reference the function correctly?",
            vm
          );
        }
        if (props && hasOwn(props, key)) {
          warn(
            ("Method \"" + key + "\" has already been defined as a prop."),
            vm
          );
        }
        if ((key in vm) && isReserved(key)) {
          warn(
            "Method \"" + key + "\" conflicts with an existing Vue instance method. " +
            "Avoid defining component methods that start with _ or $."
          );
        }
      }
      vm[key] = typeof methods[key] !== 'function' ? noop : bind(methods[key], vm);
    }
}
```

#### bind

### initData

```js
function initData (vm) {
    var data = vm.$options.data;
    data = vm._data = typeof data === 'function'
      ? getData(data, vm)
      : data || {};
    if (!isPlainObject(data)) {
      data = {};
      warn(
        'data functions should return an object:\n' +
        'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
        vm
      );
    }
    // proxy data on instance
    var keys = Object.keys(data);
    var props = vm.$options.props;
    var methods = vm.$options.methods;
    var i = keys.length;
    while (i--) {
      var key = keys[i];
      {
        if (methods && hasOwn(methods, key)) {
          warn(
            ("Method \"" + key + "\" has already been defined as a data property."),
            vm
          );
        }
      }
      if (props && hasOwn(props, key)) {
        warn(
          "The data property \"" + key + "\" is already declared as a prop. " +
          "Use prop default value instead.",
          vm
        );
      } else if (!isReserved(key)) {
        proxy(vm, "_data", key);
      }
    }
    // observe data
    observe(data, true /* asRootData */);
}
```

### getData

```js
function getData (data, vm) {
    // #7573 disable dep collection when invoking data getters
    pushTarget();
    try {
      return data.call(vm, vm)
    } catch (e) {
      handleError(e, vm, "data()");
      return {}
    } finally {
      popTarget();
    }
}
```

### proxy

```js
/**
   * Perform no operation.
   * Stubbing args to make Flow happy without leaving useless transpiled code
   * with ...rest (https://flow.org/blog/2017/05/07/Strict-Function-Call-Arity/).
   */
function noop (a, b, c) {}
var sharedPropertyDefinition = {
    enumerable: true,
    configurable: true,
    get: noop,
    set: noop
};

function proxy (target, sourceKey, key) {
    sharedPropertyDefinition.get = function proxyGetter () {
      return this[sourceKey][key]
    };
    sharedPropertyDefinition.set = function proxySetter (val) {
      this[sourceKey][key] = val;
    };
    Object.defineProperty(target, key, sharedPropertyDefinition);
}
```

### Object.defineProperty

```js
```

## 总结

导图

```js
构造函数
call
bind
Object.defineProperty
```

等等基础知识。
