# 初学者也能看懂的 Vue2 源码中那些实用的基础工具函数

## 前言

大家好，我是[若川](https://lxchuan12.gitee.io)。欢迎关注我的[公众号若川视野](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/13/16efe57ddc7c9eb3~tplv-t2oaga2asx-image.image "https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/13/16efe57ddc7c9eb3~tplv-t2oaga2asx-image.image")，最近组织了[**源码共读活动**《1个月，200+人，一起读了4周源码》](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650756550&idx=1&sn=9acc5e30325963e455f53ec2f64c1fdd&chksm=8866564abf11df5c41307dba3eb84e8e14de900e1b3500aaebe802aff05b0ba2c24e4690516b&token=917686367&lang=zh_CN#rd)，感兴趣的可以加我微信 [ruochuan12](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650756550&idx=1&sn=9acc5e30325963e455f53ec2f64c1fdd&chksm=8866564abf11df5c41307dba3eb84e8e14de900e1b3500aaebe802aff05b0ba2c24e4690516b&token=917686367&lang=zh_CN#rd) 加微信群参与，长期交流学习。

之前写的[《学习源码整体架构系列》](https://juejin.cn/column/6960551178908205093) 包含`jQuery`、`underscore`、`lodash`、`vuex`、`sentry`、`axios`、`redux`、`koa`、`vue-devtools`、`vuex4`十余篇源码文章。

>写相对很难的源码，耗费了自己的时间和精力，也没收获多少阅读点赞，其实是一件挺受打击的事情。从阅读量和读者受益方面来看，不能促进作者持续输出文章。
>所以转变思路，写一些相对通俗易懂的文章。**其实源码也不是想象的那么难，至少有很多看得懂**。歌德曾说：读一本好书，就是在和高尚的人谈话。
>同理可得：读源码，也算是和作者的一种学习交流的方式。

本文通过学习`Vue2`源码中的工具函数模块的源码，学习源码为自己所用。

阅读本文，你将学到：

```js
1. 如何学习 JavaScript 基础知识，会推荐很多学习资料
2. 如何学习调试 vue2 源码
3. 如何学习源码中优秀代码和思想，投入到自己的项目中
4. Vue 3 源码 shared 模块中的几十个实用工具函数
5. 我的一些经验分享
```

## 2. 环境准备

### 2.1 读开源项目 贡献指南

打开 [vue](https://github.com/vuejs/vue-next)，
开源项目一般都能在 `README.md` 或者 [.github/contributing.md](https://github.com/vuejs/vue-next/blob/master/.github/contributing.md) 找到贡献指南。

而贡献指南写了很多关于参与项目开发的信息。比如怎么跑起来，项目目录结构是怎样的。怎么投入开发，需要哪些知识储备等。

我们可以在 [项目目录结构](https://github.com/vuejs/vue-next/blob/master/.github/contributing.md#project-structure) 描述中，找到`shared`模块。

`shared`: Internal utilities shared across multiple packages (especially environment-agnostic utils used by both runtime and compiler packages).

`README.md` 和 `contributing.md` 一般都是英文的。可能会难倒一部分人。其实看不懂，完全可以可以借助划词翻译，整页翻译和百度翻译等翻译工具。再把英文加入后续学习计划。

本文就是讲`shared`模块，对应的文件路径是：[`vue-next/packages/shared/src/index.ts`](https://github.com/vuejs/vue-next/blob/master/packages/shared/src/index.ts)

也可以用`github1s`访问，速度更快。[github1s packages/shared/src/index.ts](https://github1s.com/vuejs/vue-next/blob/master/packages/shared/src/index.ts)

### 2.2 按照项目指南 打包构建代码

为了降低文章难度，我按照贡献指南中方法打包把`ts`转成了`js`。如果你需要打包，也可以参考下文打包构建。

你需要确保 [Node.js](http://nodejs.org/) 版本是 `10+`, 而且 `yarn` 的版本是 `1.x` [Yarn 1.x](https://yarnpkg.com/en/docs/install)。

你安装的 `Node.js` 版本很可能是低于 `10`。最简单的办法就是去官网重新安装。也可以使用 `nvm`等管理`Node.js`版本。

```bash
node -v
# v14.16.0
# 全局安装 yarn

# 推荐克隆我的项目
git clone https://github.com/lxchuan12/vue-analysis.git
cd vue-analysis/vue

# 或者克隆官方项目
git clone https://github.com/vuejs/vue.git
cd vue

npm install --global yarn
yarn # install the dependencies of the project
npm build
```

可以得到 `vue-next/packages/shared/dist/shared.esm-bundler.js`，文件也就是纯`js`文件。接下来就是解释其中的一些方法。

>当然，前面可能比较啰嗦。我可以直接讲 `3. 工具函数`。但通过我上文的介绍，即使是初学者，都能看懂一些开源项目源码，也许就会有一定的成就感。
>另外，面试问到被类似的问题或者笔试题时，你说看`Vue3`源码学到的，面试官绝对对你刮目相看。

### 2.3 如何生成 sourcemap 调试 vue-next 源码

熟悉我的读者知道，我是经常强调生成`sourcemap`调试看源码，所以顺便提一下如何配置生成`sourcemap`，如何调试。这部分可以简单略过，动手操作时再仔细看。

其实[贡献指南](https://github.com/vuejs/vue-next/blob/master/.github/contributing.md)里描述了。
>Build with Source Maps
>Use the `--sourcemap` or `-s` flag to build with source maps. Note this will make the build much slower.

所以在 `vue-next/package.json` 追加 `"dev:sourcemap": "node scripts/dev.js --sourcemap"`，`yarn dev:sourcemap`执行，即可生成`sourcemap`，或者直接 `build`。

```json
// package.json
{
    "version": "3.2.1",
    "scripts": {
        "dev:sourcemap": "node scripts/dev.js --sourcemap"
    }
}
```

会在控制台输出类似`vue-next/packages/vue/src/index.ts → packages/vue/dist/vue.global.js`的信息。

其中`packages/vue/dist/vue.global.js.map` 就是`sourcemap`文件了。

我们在 Vue3官网找个例子，在 `vue-next/examples/index.html`。其内容引入`packages/vue/dist/vue.global.js`。

```js
// vue-next/examples/index.html
<script src="../../packages/vue/dist/vue.global.js"></script>
<script>
    const Counter = {
        data() {
            return {
                counter: 0
            }
        }
    }

    Vue.createApp(Counter).mount('#counter')
</script>
```

然后我们新建一个终端窗口，`yarn serve`，在浏览器中打开`http://localhost:5000/examples/`，如下图所示，按`F11`等进入函数，就可以愉快的调试源码了。

![vue-next-debugger](./images/vue-next-debugger.png)

## 3. 工具函数

### 3.1 emptyObject

```js
/*!
 * Vue.js v2.6.14
 * (c) 2014-2021 Evan You
 * Released under the MIT License.
 */
/*  */
var emptyObject = Object.freeze({});
```

### 3.2 isUndef 是否是未定义

```js
// These helpers produce better VM code in JS engines due to their
// explicitness and function inlining.
function isUndef (v) {
  return v === undefined || v === null
}
```

### 3.3 isDef 是否是已经定义

```js
function isDef (v) {
  return v !== undefined && v !== null
}
```

### 3.4 isTrue 是否是 true

```js
function isTrue (v) {
  return v === true
}
```

### 3.5 isFalse 是否是 false

```js
function isFalse (v) {
  return v === false
}
```

### 3.6 isPrimitive 判断值是否是原始值

```js
/**
 * Check if value is primitive.
 */
function isPrimitive (value) {
  return (
    typeof value === 'string' ||
    typeof value === 'number' ||
    // $flow-disable-line
    typeof value === 'symbol' ||
    typeof value === 'boolean'
  )
}
```

## 3.7 isObject 判断是对象

```js
/**
 * Quick object check - this is primarily used to tell
 * Objects from primitive values when we know the value
 * is a JSON-compliant type.
 */
function isObject (obj) {
  return obj !== null && typeof obj === 'object'
}
```

## 3.8 toRawType

```js
/**
 * Get the raw type string of a value, e.g., [object Object].
 */
var _toString = Object.prototype.toString;

function toRawType (value) {
  return _toString.call(value).slice(8, -1)
}
```

### 3.9 isPlainObject 是否是纯对象

```js
/**
 * Strict object type check. Only returns true
 * for plain JavaScript objects.
 */
function isPlainObject (obj) {
  return _toString.call(obj) === '[object Object]'
}
```

### 3.10 isRegExp 是否是正则表达式

```js
function isRegExp (v) {
  return _toString.call(v) === '[object RegExp]'
}
```

### 3.11 isValidArrayIndex 是否是可用的数组索引值

```js
/**
 * Check if val is a valid array index.
 */
function isValidArrayIndex (val) {
  var n = parseFloat(String(val));
  return n >= 0 && Math.floor(n) === n && isFinite(val)
}
```

### 3.12 isPromise 判断是否是 promise

```js
function isPromise (val) {
  return (
    isDef(val) &&
    typeof val.then === 'function' &&
    typeof val.catch === 'function'
  )
}
```

这里用 `isDef` 判断其实相对 `isObject` 来判断 来说有点不严谨。

### 3.13 toString 转字符串

```js
/**
 * Convert a value to a string that is actually rendered.
 */
function toString (val) {
  return val == null
    ? ''
    : Array.isArray(val) || (isPlainObject(val) && val.toString === _toString)
      ? JSON.stringify(val, null, 2)
      : String(val)
}
```

### 3.14 toNumber 转数字

```js
/**
 * Convert an input value to a number for persistence.
 * If the conversion fails, return original string.
 */
function toNumber (val) {
  var n = parseFloat(val);
  return isNaN(n) ? val : n
}
```

### 3.15 makeMap 

```js
/**
 * Make a map and return a function for checking if a key
 * is in that map.
 */
function makeMap (
  str,
  expectsLowerCase
) {
  var map = Object.create(null);
  var list = str.split(',');
  for (var i = 0; i < list.length; i++) {
    map[list[i]] = true;
  }
  return expectsLowerCase
    ? function (val) { return map[val.toLowerCase()]; }
    : function (val) { return map[val]; }
}
```

### isBuiltInTag 是否是内置的 tag

```js
/**
 * Check if a tag is a built-in tag.
 */
var isBuiltInTag = makeMap('slot,component', true);
```

### isReservedAttribute 是否是保留的属性

```js
/**
 * Check if an attribute is a reserved attribute.
 */
var isReservedAttribute = makeMap('key,ref,slot,slot-scope,is');
```

### remove 移除数组中的中一项

```js
/**
 * Remove an item from an array.
 */
function remove (arr, item) {
  if (arr.length) {
    var index = arr.indexOf(item);
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

### hasOwn 检测是否是自己的属性

```js
/**
 * Check whether an object has the property.
 */
var hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn (obj, key) {
  return hasOwnProperty.call(obj, key)
}
```

### cached 缓存

```js
/**
 * Create a cached version of a pure function.
 */
function cached (fn) {
  var cache = Object.create(null);
  return (function cachedFn (str) {
    var hit = cache[str];
    return hit || (cache[str] = fn(str))
  })
}

/**
 * Camelize a hyphen-delimited string.
 */
var camelizeRE = /-(\w)/g;
var camelize = cached(function (str) {
  return str.replace(camelizeRE, function (_, c) { return c ? c.toUpperCase() : ''; })
});

/**
 * Capitalize a string.
 */
var capitalize = cached(function (str) {
  return str.charAt(0).toUpperCase() + str.slice(1)
});

/**
 * Hyphenate a camelCase string.
 */
var hyphenateRE = /\B([A-Z])/g;
var hyphenate = cached(function (str) {
  return str.replace(hyphenateRE, '-$1').toLowerCase()
});
```

### polyfillBind bind 的垫片

```js
/**
 * Simple bind polyfill for environments that do not support it,
 * e.g., PhantomJS 1.x. Technically, we don't need this anymore
 * since native bind is now performant enough in most browsers.
 * But removing it would mean breaking code that was able to run in
 * PhantomJS 1.x, so this must be kept for backward compatibility.
 */

/* istanbul ignore next */
function polyfillBind (fn, ctx) {
  function boundFn (a) {
    var l = arguments.length;
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }

  boundFn._length = fn.length;
  return boundFn
}

function nativeBind (fn, ctx) {
  return fn.bind(ctx)
}

var bind = Function.prototype.bind
  ? nativeBind
  : polyfillBind;
```

### toArray 把类数组转成真正的数组

```js
/**
 * Convert an Array-like object to a real Array.
 */
function toArray (list, start) {
  start = start || 0;
  var i = list.length - start;
  var ret = new Array(i);
  while (i--) {
    ret[i] = list[i + start];
  }
  return ret
}
```

### extend 继承 合并

```js
/**
 * Mix properties into target object.
 */
function extend (to, _from) {
  for (var key in _from) {
    to[key] = _from[key];
  }
  return to
}
```

### toObject 转对象

```js
/**
 * Merge an Array of Objects into a single Object.
 */
function toObject (arr) {
  var res = {};
  for (var i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i]);
    }
  }
  return res
}
```

### noop 空函数

```js
/* eslint-disable no-unused-vars */
/**
 * Perform no operation.
 * Stubbing args to make Flow happy without leaving useless transpiled code
 * with ...rest (https://flow.org/blog/2017/05/07/Strict-Function-Call-Arity/).
 */
function noop (a, b, c) {}
```

### no 一直返回 false

```js
/**
 * Always return false.
 */
var no = function (a, b, c) { return false; };
/* eslint-enable no-unused-vars */
```

### identity 返回参数本身

```js
/**
 * Return the same value.
 */
var identity = function (_) { return _; };
```

### genStaticKeys 获取静态属性

```js
/**
 * Generate a string containing static keys from compiler modules.
 */
function genStaticKeys (modules) {
  return modules.reduce(function (keys, m) {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}
```

### looseEqual 宽松相等

```js
/**
 * Check if two values are loosely equal - that is,
 * if they are plain objects, do they have the same shape?
 */
function looseEqual (a, b) {
  if (a === b) { return true }
  var isObjectA = isObject(a);
  var isObjectB = isObject(b);
  if (isObjectA && isObjectB) {
    try {
      var isArrayA = Array.isArray(a);
      var isArrayB = Array.isArray(b);
      if (isArrayA && isArrayB) {
        return a.length === b.length && a.every(function (e, i) {
          return looseEqual(e, b[i])
        })
      } else if (a instanceof Date && b instanceof Date) {
        return a.getTime() === b.getTime()
      } else if (!isArrayA && !isArrayB) {
        var keysA = Object.keys(a);
        var keysB = Object.keys(b);
        return keysA.length === keysB.length && keysA.every(function (key) {
          return looseEqual(a[key], b[key])
        })
      } else {
        /* istanbul ignore next */
        return false
      }
    } catch (e) {
      /* istanbul ignore next */
      return false
    }
  } else if (!isObjectA && !isObjectB) {
    return String(a) === String(b)
  } else {
    return false
  }
}
```

### looseIndexOf 宽松的 indexOf

```js
/**
 * Return the first index at which a loosely equal value can be
 * found in the array (if value is a plain object, the array must
 * contain an object of the same shape), or -1 if it is not present.
 */
function looseIndexOf (arr, val) {
  for (var i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val)) { return i }
  }
  return -1
}
```

### once 确保函数只执行一次

```js
/**
 * Ensure a function is called only once.
 */
function once (fn) {
  var called = false;
  return function () {
    if (!called) {
      called = true;
      fn.apply(this, arguments);
    }
  }
}
```

## 总结
