# 为什么 Vue2 `this.xxx` 获取到 data 里的数据

## 1. 前言

大家好，我是[若川](https://lxchuan12.gitee.io)。欢迎关注我的[公众号若川视野](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/13/16efe57ddc7c9eb3~tplv-t2oaga2asx-image.image "https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/13/16efe57ddc7c9eb3~tplv-t2oaga2asx-image.image")，最近组织了[**源码共读活动**《1个月，200+人，一起读了4周源码》](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650756550&idx=1&sn=9acc5e30325963e455f53ec2f64c1fdd&chksm=8866564abf11df5c41307dba3eb84e8e14de900e1b3500aaebe802aff05b0ba2c24e4690516b&token=917686367&lang=zh_CN#rd)，感兴趣的可以加我微信 [ruochuan12](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650756550&idx=1&sn=9acc5e30325963e455f53ec2f64c1fdd&chksm=8866564abf11df5c41307dba3eb84e8e14de900e1b3500aaebe802aff05b0ba2c24e4690516b&token=917686367&lang=zh_CN#rd) 加微信群参与，长期交流学习。

之前写的[《学习源码整体架构系列》](https://juejin.cn/column/6960551178908205093) 包含`jQuery`、`underscore`、`lodash`、`vuex`、`sentry`、`axios`、`redux`、`koa`、`vue-devtools`、`vuex4`十余篇源码文章。

写相对很难的源码，耗费了自己的时间和精力，也没收获多少阅读点赞，其实是一件挺受打击的事情。从阅读量和读者受益方面来看，不能促进作者持续输出文章。

所以转变思路，写一些相对通俗易懂的文章。**其实源码也不是想象的那么难，至少有很多看得懂。比如工具函数**。本文通过学习`Vue3`源码中的工具函数模块的源码，学习源码为自己所用。歌德曾说：读一本好书，就是在和高尚的人谈话。
同理可得：读源码，也算是和作者的一种学习交流的方式。

阅读本文，你将学到：

```js
1. 如何学习 JavaScript 基础知识，会推荐很多学习资料
2. 如何学习调试 vue2 源码
3. 如何学习源码中优秀代码和思想，投入到自己的项目中
4. Vue 3 源码 shared 模块中的几十个实用工具函数
5. 我的一些经验分享
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

### new Vue

```js
function Vue (options) {
    if (!(this instanceof Vue)
    ) {
        warn('Vue is a constructor and should be called with the `new` keyword');
    }
    this._init(options);
}
// 
initMixin(Vue);
stateMixin(Vue);
eventsMixin(Vue);
lifecycleMixin(Vue);
renderMixin(Vue);

```

### _init

```js
function initMixin (Vue) {
    Vue.prototype._init = function (options) {
      var vm = this;
      // a uid
      vm._uid = uid$3++;

      // expose real self
      vm._self = vm;
      initLifecycle(vm);
      initEvents(vm);
      initRender(vm);
      callHook(vm, 'beforeCreate');
      initInjections(vm); // resolve injections before data/props
      //  初始化好状态
      initState(vm);
      initProvide(vm); // resolve provide after data/props
      callHook(vm, 'created');
    };
}
```

### initState

```js
function initState (vm) {
    vm._watchers = [];
    var opts = vm.$options;
    if (opts.props) { initProps(vm, opts.props); }
    if (opts.methods) { initMethods(vm, opts.methods); }
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
Object.defineProperty
```

等等基础知识。
