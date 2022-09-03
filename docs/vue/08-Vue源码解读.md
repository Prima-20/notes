# 一、`Vue`源码解析--响应式原理

# 1、课程目标

-    `Vue.js`的静态成员和实例成员初始化过程
- ​	首次渲染的过程
-    数据响应式原理

# 2、准备工作

  `Vue`源码的获取

   项目地址：`https://github.com/vuejs/vue`

   为什么分析`Vue2.6`? 新的版本发布后，现有项目不会升级到`3.0`,`2.x`还有很长的一段过渡期。

`3.0`项目地址`https://github.com/vuejs/vue-next`

源码目录结构(在`src`目录下面定义的就是源码内容)：

```
compiler: 编译相关（主要作用:就是把模板转换成render函数，在render函数中创建虚拟DOM）
core:Vue核心库
platforms:平台相关代码,web:基于web的开发，weex是基于移动端的开发
server:SSR，服务端渲染
sfc:将.vue文件编译为js对象
shared:公共的代码
```

​     在`core`目录是`Vue`的核心库，在`core`目录下面，也定义了很多的文件夹，下面我们先简单来看一下。

 `components`目录下面定义的是`keep-alive.js`组件。

`global-api`:定义的是`Vue`中的静态方法。`vue.filter`,`vue.extend`,`vue.mixin`,`vue.use`等。

`Instance`:创建`vue`的实例，定义了`Vue`的构造函数，初始化，以及生命周期的钩子函数等。

`observer`:定义响应式机制的位置，

`util`:定义公共成员。

`vodom`:定义虚拟`DOM`

# 3、打包

这里我们来介绍一下，关于`Vue`源码中使用的打包方式。

打包工具`Rollup`

  `Vue.js` 所使用的打包工具为`Rollup`,`Rollup`比`Webpack`更加轻量，`Webpack`是把所有的文件（例如：图片文件，样式等）当作模块进行打包，`Rollup`只处理`js`文件，所以`Rollup`更适合在`Vue.js`这样的库中进行使用。

`Rollup`打包不会生成冗余的代码，如果是`Webpack`打包，那么会生成一些浏览器支持模块化的代码。

以上就是`Webpack`与`Rollup`之间的区别。根据以上的讲解，其实我们可以总结出，`Rollup`更适合在库的开发中使用，`Webpack`更适合在项目开发中使用，所以它们各自有自己的应用场景。

下面看一下打包的步骤:

第一步：安装依赖

```
npm i
```

第二步设置:`sourcemap`

`sourcemap`是代码地图，在`sourcemap`中记录了打包后的代码与源码之间的对应关系。如果出错了，也会告诉我们源码中的第几行出错了。怎样设置`sourcemap`呢？在`package.json` 文件中的`dev`脚本中添加`--sourcemap`.

```
"dev": "rollup -w -c scripts/config.js --sourcemap --environment TARGET:web-full-dev",
```

第三步:执行`dev`,运行`npm run dev`执行打包，用的是`rollup`,`-w` 参数是监听源码文件的变化，源码文件变化后自动的重新进行打包。`-c`是设置配置文件，`scripts/config.js`就是配置文件,`environment`环境变量，通过后面设置的值，来打包生成不同版本的`Vue`.`web-full-dev`:`web`:指的是打包`web`平台下的，`full`:表示完整版，包含了编译器与运行时，`dev`:表示的是开发版本，不会对代码进行压缩。

`web-runtime-cjs-dev`: `runtime`:表示运行时，`cjs`：表示`CommonJS`模块。

在执行``npm run dev``进行打包之前，可以先来看一下`dist`目录，该目录下面已经有很多的`js`文件，这些文件针对的是不同版本的`Vue`.那么为了更好的看到，执行`npm run dev`命令后的打包效果，在这里可以将这些文件先删除掉。

# 4、Vue不同版本说明

`https://cn.vuejs.org/v2/guide/installation.html#对不同构建版本的解释`

完整版：同时包含`编译器`和`运行时`版本。

​              什么是编译器？用来将模板字符串编译成为`javascript`渲染函数(`render`函数，`render`函数用来生成虚拟`DOM`)的代码，体积大，效率低。

​			什么是运行时？用来创建`Vue`实例，渲染并处理虚拟`DOM`等的代码，体积小，效率高，基本上就是除去编译器的代码。

还有一点需要说明的是：`Vue`包含了不同的模块化方式。

`UMD`:指的是通用的模块版本，支持多种模块方式，`UMD` 版本可以通过 `<script>` 标签直接用在浏览器中

**[CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1)**：`CommonJS `版本用来配合老的打包工具比如` [Browserify`](http://browserify.org/) 或` [webpack 1`](https://webpack.github.io/)

**[`ES Module`](http://exploringjs.com/es6/ch_modules.html)**：从 2.6 开始 `Vue `会提供两个 `ES Modules (ESM，也是ES6的模块化方式，这时标准的模块化方式，后期会使用该方式替换其它的模块化方式)` 构建文件：

- 为打包工具提供的` ESM`：为诸如 `[webpack 2](https://webpack.js.org/)` 或 `[Rollup]`(https://rollupjs.org/) 提供的现代打包工具。`ESM` 格式被设计为可以被静态分析(在编译的时候进行代码的处理也就是解析模块之间的依赖，而不是运行时)，所以打包工具可以利用这一点来进行“tree-shaking”并将用不到的代码排除出最终的包。为这些打包工具提供的默认文件 (`pkg.module`) 是只有运行时的 `ES Module` 构建 (`vue.runtime.esm.js`)。
- 为浏览器提供的 `ESM `(2.6+)：用于在现代浏览器中通过 `<script type="module">` 直接导入。

如果使用`vue-cli`创建的项目，默认的就是运行时版本，并且使用的是`ES6`的模块化方式。

同时使用`vue-cli`创建的项目中，有很多的`.vue`文件，而这些文件浏览器是不支持的，所以在打包的时候，会将这些单文件转换成`js`对象，在转换`js`对象的过程中，会将`.vue`文件中的`template`转换成`render`函数。

所以单文件组件在运行的时候也是不需要编译器的。

# 5、寻找入口文件

查看`vue`的源码，就需要找到对应的入口文件。

```
"dev": "rollup -w -c scripts/config.js --sourcemap --environment TARGET:web-full-dev",
```

可以从`srcipts/config.js`这个配置文件中进行查找。

该配置文件中的内容是比较多的。所以可以看一下文件的底部，底部导出了相应的内容、

如下所示：

```js
//判断环境变量是否有`TARGET`
//如果有的话，使用`genConfig()`生成`rollup`配置文件。
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

下面，我们看一下`genConfig`方法的代码实现

在`getConfig`方法中有一行代码如下所示：

```js
 const opts = builds[name]
```

我们看到在`builds`这个对象中，该对象中的属性就是：环境变量的值。

由于在`package.json`文件中，关于`dev`的配置中的环境变量的值为`web-full-dev`.

所以下面，我们在`builds`对象中查找该属性对应的内容。

具体内容如下：

```js
 // Runtime+compiler development build (Browser)
  'web-full-dev': {
      //表示入口文件，我们查找的就是该文件。
    entry: resolve('web/entry-runtime-with-compiler.js'),
        //出口，打包后的目标文件
    dest: resolve('dist/vue.js'),
        //模块化的方式，这里是umd
    format: 'umd',
        //打包方式，env的取值可以是开发模式或者是生产模式
    env: 'development',
        //别名，这里先不用关系
    alias: { he: './entity-decoder' },
        //表示的就是文件的头，打包好的文件的头部信息。
    banner
  },
```

在`web-full-dev`中定义的就是在打包的时候，需要的一些配置的基本信息。

通过以上代码的注意，我们知道，`web-full-dev`打包的是完整版，包含了运行时与编译器。

下面我们来看一下，入口文件，入口文件的地址为`web/entry-runtime-with-compiler.js`, 但是问题是在`scripts`目录中，我们没有发现`web`目录，我们进入`reslove`方法看一下，

```js
const aliases = require('./alias')//导入alias模块
const resolve = p => {
    //根据传递过来的参数，安装`/`进行分隔，然后获取第一项内容。
    //很明显这里获取的是 web
  const base = p.split('/')[0]
  //根据获取到的`web`,从aliases中获取一个值，下面看一下aliases中的内容。
  if (aliases[base]) {
      //aliases[base]的值:src/platforms/web
      // p的值为：web/entry-runtime-with-compiler.js
      //p.slice(base.length + 1)：获取到的就是entry-runtime-with-compiler.js
      //整个返回的内容是：src/platforms/web/entry-runtime-with-compiler.js 的绝对路径并返回
    return path.resolve(aliases[base], p.slice(base.length + 1))
  } else {
    return path.resolve(__dirname, '../', p)
  }
}

```

`aliases`中的内容定义在`scripts/alias.js`文件，具体的代码如下：

```js
const path = require('path')

const resolve = p => path.resolve(__dirname, '../', p)

module.exports = {
  vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
  compiler: resolve('src/compiler'),
  core: resolve('src/core'),
  shared: resolve('src/shared'),
  web: resolve('src/platforms/web'),
  weex: resolve('src/platforms/weex'),
  server: resolve('src/server'),
  sfc: resolve('src/sfc')
}

```

通过上面的代码，我们可以看到这里是通过`path.resolve`获取到了当前的绝对路径，并且是在`scripts`目录的上一级`src`下面去查找`platforms/web`目录中的内容。



下面我们继续来看一下`genConfig`方法。

```js
function genConfig (name) {
    //获取到了关于配置的基础信息
  const opts = builds[name]
  //config对象就是所有的配置信息
  const config = {
    input: opts.entry,//入口
    external: opts.external,
    plugins: [
      flow(),
      alias(Object.assign({}, aliases, opts.alias))
    ].concat(opts.plugins || []),
    output: {
      file: opts.dest,//出口
      format: opts.format,
      banner: opts.banner,
      name: opts.moduleName || 'Vue'
    },
    onwarn: (msg, warn) => {
      if (!/Circular/.test(msg)) {
        warn(msg)
      }
    }
  }

  // built-in vars
  const vars = {
    __WEEX__: !!opts.weex,
    __WEEX_VERSION__: weexVersion,
    __VERSION__: version
  }
  // feature flags
  Object.keys(featureFlags).forEach(key => {
    vars[`process.env.${key}`] = featureFlags[key]
  })
  // build-specific env
  if (opts.env) {
    vars['process.env.NODE_ENV'] = JSON.stringify(opts.env)
  }
  config.plugins.push(replace(vars))

  if (opts.transpile !== false) {
    config.plugins.push(buble())
  }

  Object.defineProperty(config, '_name', {
    enumerable: false,
    value: name
  })
//将配置信息返回
  return config
}
```

# 6、从入口开始

通过上一小节的内容，我们已经找到了对应的入口文件`src/platform/web/entry-runtime-with-compiler.js`

下面我们要对入口文件进行分析，在分析的过程中，我们要解决一个问题，如下代码所示：

```js
const vm=new Vue({
  el:'#app',
template:'<h3>hello template</h3>'
render(h){
  return h('h4','hello render')
}
})
```

在上面的代码中，我们在创建`Vue`的实例的时候，同时指定了`template`与`render`，那么会渲染执行哪个内容？

一会我们通过查看源码来解决这个问题。

下面我们打开入口文件。

先来看一下`$mount`

```js
//保留Vue实例的$mount方法
const mount = Vue.prototype.$mount;
//$mout:挂载，作用就是把生成的DOM挂载到页面中。
Vue.prototype.$mount = function (
  el?: string | Element,
  //非ssr情况下为false,ssr的时候为true
  hydrating?: boolean
): Component {
  //获取el选项，创建vue实例的时候传递过来的选项。
  //el就是DOM对象。
  el = el && query(el);

  /* istanbul ignore if */
  //如果el为body或者是html,并且是开发环境，那么会在浏览器的控制台
  //中输出不能将Vue的实例挂载到<html>或者是<body>标签上
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== "production" &&
      warn(
        `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
      );
    //直接返回vue的实例
    return this;
  }
  //获取options选项
  const options = this.$options;
  // resolve template/el and convert to render function
  //判断options中是否有render(在创建vue实例的时候，也就new Vue的时候是否传递了render函数)
  if (!options.render) {
    //没有传递render函数。获取template模板，然后将其转换成render函数
    //关于将`template`转换成render的代码比较多，目录先知道其主要作用就可以了
    let template = options.template;
    if (template) {
      if (typeof template === "string") {
          //如果是id选择器
        if (template.charAt(0) === "#") {
            //获取对应的DOM对象的innerHTML,作为模板
          template = idToTemplate(template);
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== "production" && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            );
          }
        }
      } else if (template.nodeType) {
          //如果模板是元素，返回元素的innerHTML
        template = template.innerHTML;
      } else {
          //如果不是字符串，也不是元素，在开发环境中会给出警告信息，模板不合法
        if (process.env.NODE_ENV !== "production") {
          warn("invalid template option:" + template, this);
        }
          //返回Vue实例。
        return this;
      }
    } else if (el) {
        //如果选项中没有设置template模板，那么获取el的outerHTML 作为模板。
      template = getOuterHTML(el);
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== "production" && config.performance && mark) {
        mark("compile");
      }
//把template模板编译成render函数
      const { render, staticRenderFns } = compileToFunctions(
        template,
        {
          outputSourceRange: process.env.NODE_ENV !== "production",
          shouldDecodeNewlines,
          shouldDecodeNewlinesForHref,
          delimiters: options.delimiters,
          comments: options.comments,
        },
        this
      );
      options.render = render;
      options.staticRenderFns = staticRenderFns;

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== "production" && config.performance && mark) {
        mark("compile end");
        measure(`vue ${this._name} compile`, "compile", "compile end");
      }
    }
  }
  //如果创建 Vue实例的时候，传递了render函数，这时会直接调用mount方法。
  // mount方法的作用就是渲染DOM,这块内容在下一小节会讲解到。
  return mount.call(this, el, hydrating);
};

```

下面代码是`query`方法实现的代码。

```js
/**
 * Query an element selector if it's not an element already.
 */
export function query(el: string | Element): Element {
  // 如果el等于字符串，表明是选择器。
  //否则是DOM对象，直接返回
  if (typeof el === "string") {
    //获取对应的DOM元素
    const selected = document.querySelector(el);
    if (!selected) {
      //如果没有找到，判断是否为开发模式，如果是开发模式
      //在控制台打印“找不到元素”
      process.env.NODE_ENV !== "production" &&
        warn("Cannot find element: " + el);
      //这时会创建一个`div`元素返回。
      return document.createElement("div");
    }
    //返回找到的dom元素
    return selected;
  } else {
    return el;
  }
}
```

看完上面的代码后，我们就可以回答最开始的时候，提出的问题，如果传递了`render`函数，是不会处理`template`这个模板的，直接调用`mount`方法渲染`dom`

现在面临的一个问题就是`$mount`这个方法是在哪儿被调用的呢？

在`core/instance/init.js`文件中，查找到如下代码：

```
 if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
```

通过以上代码，可以看到调用了`$mount`方法。

也就是在`Vue._init`方法中调用的。

那么`_init`方法是在哪被调用的呢？
在`core/instance/index.js`文件中、

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

```

通过以上的代码，可以看到在`Vue`这个方法中调用了`_init`方法，而`Vue`方法，是在创建`Vue`实例的时候被调用的。所以上面的`Vue`方法就是一个构造函数。

现在我们将以上的内容做一个总结，重点是以下三点内容。

- `el`不能是`body` 或者是`html`标签
- 如果没有`render`，把`template`转换成`render`函数。
- 如果有`render`方法，直接调用`mount`挂载`DOM`

# 7、Vue的初始化过程

在这一小节中，我们需要考虑如下的一个问题。

`Vue`实例成员和`Vue`的静态成员是从哪里来的？

在`src/platforms/web`目录下面定义的文件都是与平台有关的文件。

下面我们还是看一下开始文件：`entry-runtime-with-compiler.js`文件。

```js
/* @flow */

import config from "core/config";
import { warn, cached } from "core/util/index";
import { mark, measure } from "core/util/perf";
//导入Vue的构造函数
import Vue from "./runtime/index";
import { query } from "./util/index";
import { compileToFunctions } from "./compiler/index";
import {
  shouldDecodeNewlines,
  shouldDecodeNewlinesForHref,
} from "./util/compat";

const idToTemplate = cached((id) => {
  const el = query(id);
  return el && el.innerHTML;
});
//保留Vue实例的$mount方法，方便下面重写$mount的功能
const mount = Vue.prototype.$mount;
//$mout:挂载，作用就是把生成的DOM挂载到页面中。
Vue.prototype.$mount = function (
  el?: string | Element,
  //非ssr情况下为false,ssr的时候为true
  hydrating?: boolean
): Component {
  //获取el选项，创建vue实例的时候传递过来的选项。
  //el就是DOM对象。
  el = el && query(el);

  /* istanbul ignore if */
  //如果el为body或者是html,并且是开发环境，那么会在浏览器的控制台
  //中输出不能将Vue的实例挂载到<html>或者是<body>标签上
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== "production" &&
      warn(
        `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
      );
    //直接返回vue的实例
    return this;
  }
  //获取options选项
  const options = this.$options;
  // resolve template/el and convert to render function
  //判断options中是否有render(在创建vue实例的时候，也就new Vue的时候是否传递了render函数)
  if (!options.render) {
    //没有传递render函数。获取template模板，然后将其转换成render函数
    //关于将`template`转换成render的代码比较多，目录先知道其主要作用就可以了
    let template = options.template;
    // 如果模板存在
    if (template) {
      //判断对应的类型如果是字符串
      if (typeof template === "string") {
        //如果模板是id选择器
        if (template.charAt(0) === "#") {
          //获取对应的DOM对象的innerHTML
          template = idToTemplate(template);
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== "production" && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            );
          }
        }
      } else if (template.nodeType) {
        //如果模板是元素，返回元素的innerHTML
        template = template.innerHTML;
      } else {
        if (process.env.NODE_ENV !== "production") {
          warn("invalid template option:" + template, this);
        }
        return this;
      }
    } else if (el) {
      //如果模板不存在
      template = getOuterHTML(el);
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== "production" && config.performance && mark) {
        mark("compile");
      }
//把template模板编译成render函数
      const { render, staticRenderFns } = compileToFunctions(
        template,
        {
          outputSourceRange: process.env.NODE_ENV !== "production",
          shouldDecodeNewlines,
          shouldDecodeNewlinesForHref,
          delimiters: options.delimiters,
          comments: options.comments,
        },
        this
      );
      options.render = render;
      options.staticRenderFns = staticRenderFns;

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== "production" && config.performance && mark) {
        mark("compile end");
        measure(`vue ${this._name} compile`, "compile", "compile end");
      }
    }
  }
  //如果创建 Vue实例的时候，传递了render函数，这时会直接调用mount方法。
  // mount方法的作用就是渲染DOM，这里的mount就是下面我们要看的./runtime/index文件中的$mount,只不过
   // 在当前的文件中重写了。
  return mount.call(this, el, hydrating);
};

/**
 * Get outerHTML of elements, taking care
 * of SVG elements in IE as well.
 */
function getOuterHTML(el: Element): string {
  if (el.outerHTML) {
    //如果有outerHTML属性，返回内容的HTML形式
    return el.outerHTML;
  } else {
    //创建div
    const container = document.createElement("div");
    //把el的内容克隆，然后追加到div中
    container.appendChild(el.cloneNode(true));
    //返回div的innerHTML
    return container.innerHTML;
  }
}
//注册Vue.compile方法，根据HTML字符串返回render函数
Vue.compile = compileToFunctions;

export default Vue;
```



通过查看该文件中的代码，可以发现，在该文件中并没有创建`Vue`的实例，关于实例的创建在如下导入的文件中。

```js
//导入Vue的构造函数
import Vue from "./runtime/index";
```

通过前面的讲解，我们知道在入口文件中，最主要的方法是`mount`.

```js
//保留Vue实例的$mount方法，方便下面重写$mount的功能
const mount = Vue.prototype.$mount;
```

在该方法中，很重要的一个操作就是将`template`模板，转换成`render`函数。

下面，我们先来看一下`./runtime/index`文件中的内容。

```js
// install platform specific utils
//给Vue.config注册了方法，这些方法都是与平台相关的方法。这些方法是在Vue内部使用的。

Vue.config.mustUseProp = mustUseProp;
//是否为保留的标签，也就是说，传递过来的内容是否为HTML中特有的标签
Vue.config.isReservedTag = isReservedTag;
//是否是保留的属性，也就是说，传递过来的内容是否为HTML中特有的属性
Vue.config.isReservedAttr = isReservedAttr;
Vue.config.getTagNamespace = getTagNamespace;
Vue.config.isUnknownElement = isUnknownElement;

```

以上内容简短了解一下就可以。

下面我们继续查看如下内容：

```js
// install platform runtime directives & components
//通过extend方法注册了与平台相关的全局的指令与组件。
//extend的作用就是将第二个参数的成员全部拷贝到第一个参数中
//那么问题是注册了哪些指令与组件呢？
extend(Vue.options.directives, platformDirectives);
extend(Vue.options.components, platformComponents);
```

那么问题是注册了哪些指令与组件呢？

我们可以查看，`extend`的第二个参数，以上`extend`参数的内容，分别来自如下两个文件。

```js
import platformDirectives from "./directives/index";
import platformComponents from "./components/index";
```

我们首先看一下`./directives/index`文件中的内容，如下所示：

```js
import model from './model'
import show from './show'

export default {
  model,
  show
}

```

通过以上代码，我们可以看到这个文件中导出了`v-show`与`v-model`这两个指令。

下面再看一下`./components/index`文件中的内容。

```js
import Transition from './transition'
import TransitionGroup from './transition-group'

export default {
  Transition,
  TransitionGroup
}

```

以上文件导出的就是`v-Transition`与`v-TransitionGroup`这两个组件的内容。

通过下面两行代码，我们还可以发现一个相应的问题。

```js
extend(Vue.options.directives, platformDirectives);
extend(Vue.options.components, platformComponents);
```

就是全局的指令与组件分别存储到了`Vue.options.directives`与`Vue.options.components`中。

例如，我们在项目中使用`Vue.component`注册的组件，都存储到了`Vue.options.components`中，也就说存储了`Vue.options.components`中的组件都是全局可以访问的。

我们继续向下看，如下代码

```js
// install platform patch function
Vue.prototype.__patch__ = inBrowser ? patch : noop;
```

以上代码就是在`Vue`的原型中注册了`__patch__`函数，该函数对我们来说比较熟悉了，在学习虚拟`DOM`的时候，我们知道`patch`函数的作用就是将虚拟`DOM`转换成真实的`DOM`.在给`patch`函数赋值的时候，首先判断是否为浏览器的环境，如果是则返回`patch`,否则返回`noop`,`noop`是一个空函数。

这里有一个问题就是：`inBrowser`是怎样判断是否为浏览器环境的呢？

`inBrowser`定义在如下文件中`import { devtools, inBrowser } from "core/util/index";`

该文件的代码如下：

```js
/* @flow */

export * from 'shared/util'
export * from './lang'
export * from './env'
export * from './options'
export * from './debug'
export * from './props'
export * from './error'
export * from './next-tick'
export { defineReactive } from '../observer/index'

```

通过以上的代码，我们并没有发现`isBrowser`的定义，那么很明显都被封装到具体的文件中了。

`isBrowser`是与环境有关的内容，所以很明显来自`env`这个文件，在该文件中，可以看到如下代码：

```js
export const inBrowser = typeof window !== 'undefined'
```

如果`window`对象不等于`undefined`,表明当前的环境就是浏览器的环境

```js
//给Vue原型注册了$mount方法，也就是给Vue的实例注册了$mount方法，在entry-runtime-with-compiler.js文件中对该方法进行了重写。
//在该方法中调用了mountComponent方法，用来渲染DOM
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
    //这里重新获取el,并且判断是否为浏览器环境.
    //这里有一个问题，就是为什么要重新获取el,在entry-runtime-with-compiler.js文件中重写$mount方法的时候也获取了el,这里为什么要重新获取呢？
    //原因是：entry-runtime-with-compiler.js是带编译器版本的Vue,而当前文件是运行时版本执行的，那么就不会执行entry-runtime-with-compiler.js文件来获取el,所以这里必须要重新获取el。
  el = el && inBrowser ? query(el) : undefined;
  return mountComponent(this, el, hydrating);
};
```

下面看一下`mountComponent`方法的内部实现。

```
import { mountComponent } from "core/instance/lifecycle";
```

```js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el//把el赋值给Vue实例中的$el属性
    //判断$options中是否有render函数。
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
        //如果在运行时环境中，如果使用了template模板，会出现如下的警告信息。
        //警告信息：使用的是运行时版本，编译器无效，应该使用完整版，或者是编写render函数。
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
    //触发beforeMount钩子函数，表示的是挂载之前。
  callHook(vm, 'beforeMount')

  let updateComponent //完成组件的更新
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
      //重点看这一段代码：
      // vm._render():表示执行的是用户传入的render函数，或者是执行编译器生成的render函数。
      // render( )函数最终会返回虚拟DOM,把返回的虚拟DOM传递给_update函数。
      // _update函数，会将虚拟DOM转换成真实的DOM。
      //也就说该方法执行完毕后，页面会呈现出具体的内容。
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
 //在 Watcher中调用了updateComponent方法。
    // 
    new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
      //触发了mounted钩子函数，表明页面已经挂载完毕了。
    callHook(vm, 'mounted')
  }
  return vm
}

```



`mountComponent`后面的代码就是一些调试的代码，这里也不要在看。

下面，把我们看到的代码总结一下。以上代码定义的是与平台有关的代码，主要是注册了`patch`方法，以及`$mount`方法。还有就是注册了全局的指令与组件。

但在当前的这个文件中，我们还是没有看到`Vue`的构造函数。

在该文件的顶部，可以看到如下导入的语句。

```
import Vue from "core/index";
```

该文件中的代码如下：

```js
import Vue from "./instance/index";
import { initGlobalAPI } from "./global-api/index";
import { isServerRendering } from "core/util/env";
import { FunctionalRenderContext } from "core/vdom/create-functional-component";
//给Vue构造函数注册一些静态的方法。
initGlobalAPI(Vue);
//以下通过defineProperty定义的内容都是与服务端渲染有关的内容。
Object.defineProperty(Vue.prototype, "$isServer", {
  get: isServerRendering,
});

Object.defineProperty(Vue.prototype, "$ssrContext", {
  get() {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext;
  },
});

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, "FunctionalRenderContext", {
  value: FunctionalRenderContext,
});
//指定Vue版本。
Vue.version = "__VERSION__";

export default Vue;
```

下面我们来看一下`initGlobalAPI`方法中的代码。

该方法的代码定义在`core/global-api/index.js`文件中。

具体的代码如下：

```js
/* @flow */

import config from "../config";
import { initUse } from "./use";
import { initMixin } from "./mixin";
import { initExtend } from "./extend";
import { initAssetRegisters } from "./assets";
import { set, del } from "../observer/index";
import { ASSET_TYPES } from "shared/constants";
import builtInComponents from "../components/index";
import { observe } from "core/observer/index";

import {
  warn,
  extend,
  nextTick,
  mergeOptions,
  defineReactive,
} from "../util/index";

export function initGlobalAPI(Vue: GlobalAPI) {
  // config
  const configDef = {};
  configDef.get = () => config;
  if (process.env.NODE_ENV !== "production") {
    configDef.set = () => {
      warn(
        "Do not replace the Vue.config object, set individual fields instead."
      );
    };
  }
  // 初始化了`Vue.config`对象，该对象是Vue的静态成员
  Object.defineProperty(Vue, "config", configDef);

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  //这些工具方法不视作全局API的一部分，除非你已经意识到某些风险，否则不要去依赖他们，也就是说，使用这些API会出现一些问题。
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive,
  };
  //静态方法set/delete/nextTick
  //挂载到了Vue的构造函数中。后期会继续看内部源码的实现
  Vue.set = set;
  Vue.delete = del;
  Vue.nextTick = nextTick;

  // 2.6 explicit observable API
  //让一个对象变成可响应式的，内部调用了observe方法。
  Vue.observable = <T>(obj: T): T => {
    observe(obj);
    return obj;
  };
    
  //初始化了Vue.options对象，并给其扩展了
  //components/directives/filters内容，后面还会看这块内容
  Vue.options = Object.create(null);
  ASSET_TYPES.forEach((type) => {
    Vue.options[type + "s"] = Object.create(null);
  });

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue;
  //设置keep-alive组件
  extend(Vue.options.components, builtInComponents);
//注册Vue.use(),用来注册插件
  initUse(Vue);
    //注册Vue.mixin( )实现混入
  initMixin(Vue);
    //注册Vue.extend( )基于传入的options返回一个组件的构造函数
  initExtend(Vue);
    // 注册Vue.directive(),Vue.component( ),Vue.filter( )
  initAssetRegisters(Vue);
}

```

目前我们只需要知道在`initGlobalAPI`方法，初始化了`Vue`的静态方法。

现在，我们回到`import Vue from "core/index";` 文件，发现在该文件中也没有创建`Vue`的构造函数，

但是在其顶部，导入了：`import Vue from "./instance/index";`

打开该文件查看的代码如下所示：

```js
import { initMixin } from "./init";
import { stateMixin } from "./state";
import { renderMixin } from "./render";
import { eventsMixin } from "./events";
import { lifecycleMixin } from "./lifecycle";
import { warn } from "../util/index";
//Vue构造函数
function Vue(options) {
  //判断是否为生产环境，如果不等于生产环境并且如果this不是Vue的实例
  //那么说明用户将其作为普通函数调用，而不是通过new来创建其实例，所以会出现如下错误提示
  if (process.env.NODE_ENV !== "production" && !(this instanceof Vue)) {
    warn("Vue is a constructor and should be called with the `new` keyword");
  }
  //调用_init( )方法
  this._init(options);
}
//注册vm的_init( )方法，初始化vm
initMixin(Vue);
//注册vm（Vue实例）的$data/$props/$set/$delete/$watch
stateMixin(Vue);
//初始化事件相关的方法
//$on/$once/$off/$emit
eventsMixin(Vue);
//初始化生命周期相关的混入方法
// $forceUpdate/$destroy
lifecycleMixin(Vue);
//混入render
// $nextTick
renderMixin(Vue);

export default Vue;

```

下面看一下`stateMixin`方法，在该方法中，我们可以看到如下的代码

```js
 //在Vue的实例总初始化了一些属性和者方法。
  Object.defineProperty(Vue.prototype, "$data", dataDef);
  Object.defineProperty(Vue.prototype, "$props", propsDef);

  Vue.prototype.$set = set;
  Vue.prototype.$delete = del;

```

通过上面的代码，我们可以看到在`stateMixin`方法中为`Vue`的原型上注册了对应的属性和方法，也就说在这个位置实现了为`Vue`的实例初始化属性和方法。

到此位置，我们最开始提出的问题：

`Vue`实例成员和`Vue`的静态成员是从哪里来的？

现在已经全部找到。

关于`import Vue from "./instance/index";`文件中的其他方法，后续课程内容中还会查看。

目前我们只需要知道，在在该文件中，创建了`Vue`的构造函数，并且设置了`Vue`实例的成员。

这里还有一个小的问题就是，在创建`Vue`的实例的时候，这里使用了构造函数，而没有使用类(`class`)的形式，

原因是：因为使用类来实现构造函数的，下面的方法就不容易实现。在这些方法中为`Vue`的原型上挂在了很多的成员，而使用类的构造函数不容易实现。

现在我们将这块内容做一总结：

通过前面的讲解，我们看了四个文件

`src/platforms/web/entry-runtime-with-compiler.js`

- `web`平台相关的入口
- 重写了平台相关的`$mount( )`方法，把`template`模板转换成`render`函数
-  注册了`Vue.compile( )`方法，可以根据传递的`HTML`字符串返回`render`函数

`src/platforms/web/runtime/index.js`

- `web`平台相关

- 注册和平台相关的全局指令：`v-model`,`v-show`

- 注册和平台相关的全局组件:`v-transition`,`v-transition-group`

- 全局的指令与组件分别存储到了`Vue.options.directives`与`Vue.options.components`中。

- 全局方法

  ​	`__patch__`：把虚拟`DOM`转换成真实`DOM`

  ​	`$mount`：挂载方法，把`DOM`渲染到页面中，在`src/platforms/web/entry-runtime-with-compiler.js`文件中重写了

  `$mount`,使其具有了编译的能力。

- `src/core/index.js`

    与平台无关

    设置了`Vue`的静态方法，`initGlobalAPI(Vue)`

- `src/core/instance/index.js`

  与平台无关

  定义了`Vue`的构造方法，调用了`this._init(options)`方法（该方法是整个程序的入口）

  给`Vue`中混入了常用的实例成员。

# 8、静态成员初始化

​	这一小节我们来看一下`Vue`中静态成员的初始化，通过前面的讲解，我们知道静态成员都是在文件``src/core/index.js``中完成初始化，下面再看一下该文件。

在该文件有一个`initGlobalAPI`方法，在该方法中注册了一些静态成员。

```
//给Vue构造函数注册一些静态的方法，属性。
initGlobalAPI(Vue);
```

`initGlobalAPI`方法的具体实现如下：

```js
export function initGlobalAPI(Vue: GlobalAPI) {
  // config
  const configDef = {};
  //为configDef对象添加一个get方法，返回一个config对象
  configDef.get = () => config;
  if (process.env.NODE_ENV !== "production") {
    //如果不是生产环境，则为开发环境，这时会为configDef添加一个set方法，如果为config进行赋值操作，会出现不能给`Vue.config`重新赋值的错误。
    configDef.set = () => {
      warn(
        "Do not replace the Vue.config object, set individual fields instead."
      );
    };
  }
  // 初始化了`Vue.config`对象，该对象是Vue的静态成员
  //这里不是定义响应式数据，而是为Vue定义了一个config属性
  //并且为其设置了configDef约束。
  Object.defineProperty(Vue, "config", configDef);

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  //这些工具方法不视作全局API的一部分，除非你已经意识到某些风险，否则不要去依赖他们，也就是说，使用这些API会出现一些问题。
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive,
  };
  //静态方法set/delete/nextTick
  //挂载到了Vue的构造函数中。后期会继续看内部源码的实现
  Vue.set = set;
  Vue.delete = del;
  Vue.nextTick = nextTick;

  // 2.6 explicit observable API
  //让一个对象变成可响应式的，内部调用了observe方法。
  Vue.observable = <T>(obj: T): T => {
    observe(obj);
    return obj;
  };
  //初始化了Vue.options对象，并给其扩展了
  //components/directives/filters内容
  Vue.options = Object.create(null);
  ASSET_TYPES.forEach((type) => {
    Vue.options[type + "s"] = Object.create(null);
  });

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue;

  extend(Vue.options.components, builtInComponents);

  initUse(Vue);
  initMixin(Vue);
  initExtend(Vue);
  initAssetRegisters(Vue);
}

```

在上面的代码中，首先初始化了`Vue.config`对象,并且添加了相应的约束。

```js
 // 初始化了`Vue.config`对象，该对象是Vue的静态成员
  //这里不是定义响应式数据，而是为Vue定义了一个config属性
  //并且为其设置了configDef约束。
  Object.defineProperty(Vue, "config", configDef);
```

初始化`Vue.config`对象后，在什么位置为其挂载了成员。

在`platforms/web/runtime/index.js`文件中

```js
// install platform specific utils
//给Vue.config注册了方法，这些方法都是与平台相关的方法。这些方法是在Vue内部使用的。

Vue.config.mustUseProp = mustUseProp;
//是否为保留的标签，也就是说，传递过来的内容是否为HTML中特有的标签
Vue.config.isReservedTag = isReservedTag;
//是否是保留的属性，也就是说，传递过来的内容是否为HTML中特有的属性
Vue.config.isReservedAttr = isReservedAttr;
Vue.config.getTagNamespace = getTagNamespace;
Vue.config.isUnknownElement = isUnknownElement;
```

下面，我们来看如下的代码，前面的代码我们后面在讲解(`set`,`delete`,`observable`等内容)。

```js
  //初始化了Vue.options对象，并给其扩展了
  //components/directives/filters内容

//创建了Vue.options对象，并且没有指定原型。这样性能更高。
 Vue.options = Object.create(null);
  ASSET_TYPES.forEach((type) => {
      //从`ASSET_TYPES`数组中取出每一项，并为其添加了`s`,来作为Vue.options对象的属性。
      //也就是说给Vue.options中挂载了三个成员，分别是：components/directives/filters，并且都初始化成了空对象。这三个成员的作用是用来存储全局的组件，指令和过滤器，我们通过Vue.component,Vue.directive,Vue.filter创建的组件，指令，过滤器最终都会存储到Vue.options中的这三个成员中。
    Vue.options[type + "s"] = Object.create(null);
  });
```

对`ASSET_TYPES`数组进行遍历，那么该数组中存储的是什么内容呢？该数组具体定义的位置：

```js
import { ASSET_TYPES } from "shared/constants";
```

在`src/shared`目录下面找到`constants.js`文件，该文件中的代码如下：

```js
export const SSR_ATTR = 'data-server-rendered'
//该数组中定义了我们比较常见的Vue.component,Vue.directive,Vue.filter的方法名称。
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
//生命周期的钩子函数名称，后面在讲解这块内容
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured',
  'serverPrefetch'
]

```

现在，我们知道了`ASSET_TYPES`数组中的内容，然后在来看一下对应的循环内容。

看完循环内容后，我们再来看一下如下代码：

```js
//将Vue的构造函数存储到了_base属性中，后期会用到。  
Vue.options._base = Vue;
```

```js
//设置keep-alive组件
//extend方法的作用是将第二个参数中的属性，拷贝到第一个参数中。下面可以看一下extend方法的具体实现。
extend(Vue.options.components, builtInComponents);
```

`extend`方法定义在

```js
import {
  warn,
  extend,
  nextTick,
  mergeOptions,
  defineReactive,
} from "../util/index";
```

查看`until/index`中的内容

```js
export * from 'shared/util'
export * from './lang'
export * from './env'
export * from './options'
export * from './debug'
export * from './props'
export * from './error'
export * from './next-tick'
export { defineReactive } from '../observer/index'
```

在`src/shared/util`文件中，打开该文件搜索`extend`

```js
export function extend (to: Object, _from: ?Object): Object {
  for (const key in _from) {
    to[key] = _from[key]
  }
  return to
}
```

实现了一个浅拷贝。把一个对象的成员拷贝给另外一个对象

下面的代码是将`builtInComponents`拷贝给`Vue.options.components`,这里完成的就是全局组件的注册

```js
//设置keep-alive组件
extend(Vue.options.components, builtInComponents);
```

下面再来看一下`builtInComponents`中的内容。

```js
import builtInComponents from "../components/index";
```

```js
import KeepAlive from './keep-alive'
export default {
  KeepAlive
}

```

通过上面的代码可以看到，导出了`KeepAlive`这个组件

下面看一下` initUse(Vue);` 该方法的作用:`注册Vue.use(),用来注册插件`

`initUser`方法定义在`global-api/use.js`文件。

```js
import { toArray } from '../util/index'
//参数为Vue的构造函数
export function initUse (Vue: GlobalAPI) {
    //为Vue添加use函数
    //plugin:是一个函数或者是对象，表示的就是插件
  Vue.use = function (plugin: Function | Object) {
      //installedPlugins：表示已经安装的插件
      //注意:this表示的是Vue的构造函数
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    //判断传递过来的plugin这个插件是否在installedPlugins中存在，如果存在表示已经注册安装了，直接返回
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
      //下面就是实现插件的注册，如果plugin中有install这个属性，表示传递过来的plugin表示的是对象
      //这时候直接调用plugin中的install完成插件的注册。也就是说如果在注册插件的时候，传递的是一个对象，这个对象中一定要有install这个方法，关于这块内容我们在前面的课程中也已经讲解过来。
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
        //如果传递的是函数，直接调用函数
      plugin.apply(null, args)
    }
      //将插件添加的数组中
    installedPlugins.push(plugin)
    return this
  }
}
```

下面我们再来查看一下`initMixin`方法，该方法的作用就是用来注册`Vue.mixin( )`用来实现混入。

具体代码的位置：`global-api/mixin.js`文件

```js
import { mergeOptions } from '../util/index'

export function initMixin (Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
      //将mixin这个对象中的所有成员拷贝到options中，this指的就是Vue
    this.options = mergeOptions(this.options, mixin)
    return this
  }
}

```

下面，我们再来看一下`initExtend`方法，该方法的作用是注册`Vue.extend( )`，基于传入的`options`返回一个组件的构造函数。

代码位置：`global-api/entend.js`

```js
  Vue.extend = function (extendOptions: Object): Function {
    extendOptions = extendOptions || {}
      //Vue的构造函数
    const Super = this
    const SuperId = Super.cid
    //从缓存中加载组件的构造函数
    const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }

    const name = extendOptions.name || Super.options.name
    if (process.env.NODE_ENV !== 'production' && name) {
        //如果是开发环境验证组件的名称
      validateComponentName(name)
    }
//VueComponent表示组件的构造函数
    const Sub = function VueComponent (options) {
        //调用_init()初始化
      this._init(options)
    }
    //改变了Sub这个构造函数的原型，让其继承了Vue, Super.prototype表示的是Vue的原型。
    //所以说所有的Vue组件都是继承自Vue.
    Sub.prototype = Object.create(Super.prototype)
    Sub.prototype.constructor = Sub
    Sub.cid = cid++
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    )
    Sub['super'] = Super

    // For props and computed properties, we define the proxy getters on
    // the Vue instances at extension time, on the extended prototype. This
    // avoids Object.defineProperty calls for each instance created.
    if (Sub.options.props) {
      initProps(Sub)
    }
    if (Sub.options.computed) {
      initComputed(Sub)
    }

    // allow further extension/mixin/plugin usage
      //把Super(Vue)的成员拷贝到Sub这个构造函数中，这样就表明我们创建的组件具有了这些成员。
    Sub.extend = Super.extend
    Sub.mixin = Super.mixin
    Sub.use = Super.use

    // create asset registers, so extended classes
    // can have their private assets too.
    ASSET_TYPES.forEach(function (type) {
      Sub[type] = Super[type]
    })
    // enable recursive self-lookup
    if (name) {
      Sub.options.components[name] = Sub
    }

    // keep a reference to the super options at extension time.
    // later at instantiation we can check if Super's options have
    // been updated.
    Sub.superOptions = Super.options
    Sub.extendOptions = extendOptions
    Sub.sealedOptions = extend({}, Sub.options)

    // cache constructor
      //把组件的构造函数缓存到options._Ctor
    cachedCtors[SuperId] = Sub
      //返回组件的构造函数VueComponent
    return Sub
  }
```

以上内容简单了解一下就可以。

下面看一下：`initAssetRegisters`方法，在该方法中注册了`Vue.directive()`,`Vue.component( )`,`Vue.filter( )`

在这里需要注意的一点就是，以上三个方法并不是一个一个的注册的，而是一起注册的。为什么会一起注册呢？因为这三个方法的参数是一样的。这块可以参考文档。

下面看一下具体的源码实现。

 

```js
/* @flow */

import { ASSET_TYPES } from 'shared/constants'
import { isPlainObject, validateComponentName } from '../util/index'

export function initAssetRegisters (Vue: GlobalAPI) {
  /**
   * Create asset registration methods.
   */
    //变量ASSET_TYPES数组，为`Vue`定义相应的方法
    //ASSET_TYPES数组包括了`directive`,`component`,`filter`
  ASSET_TYPES.forEach(type => {
      //分别给Vue中的`directive`,`component`,`filter`注册方法
    Vue[type] = function (
      id: string,//是名字(组件，指令，过滤器的名字)
      definition: Function | Object //定义，可以是对象或者是函数，这两个参数可以通过查看手册：https://vuejs.bootcss.com/api/#Vue-directive
    ): Function | Object | void {
      if (!definition) {
          // 如果没有传递第二个参数，通过this.options找到之前存储的directive,component,filter，并返回
          //通过前面的学习，我们知道Vue.directive,Vue.component,Vue.filter都注册到了this.options['directives'],this.options['components'],this.options['filters']中
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && type === 'component') {
          validateComponentName(id)
        }
      //判断从ASSET_TYPES数组中取出的是否为`component`(也就是是否为组件)
      //同时判断definition参数是否为对象
        if (type === 'component' && isPlainObject(definition)) {
            //为definition设置名字，如果在Vue.component中的第二个参数设置了name属性，那么就使用该属性的值.
            //如果没有设置，则使用id的值作为definition的名字
          definition.name = definition.name || id
            //this.options._base表示的是Vue的构造函数。
            //Vue.extend()：我们看过，作用就是将一个普通的对象转换成了VueComponent的构造函数、
            //看到这里，我们回到官方手册:https://vuejs.bootcss.com/api/#Vue-component
            // 注册组件，传入一个扩展过的构造器
			//Vue.component('my-component', Vue.extend({ /* ... */ }))
			// 注册组件，传入一个选项对象 (自动调用 Vue.extend)
			//Vue.component('my-component', { /* ... */ })
            //以上是官方手册中的内容，如果在使用Vue.component方法的时候，传递的第二个参数为Vue.extend,
            //那么会直接执行this.options[type + 's'][id] = definition这样代码，因为如果传递的是Vue.extend,那么以上if判断条件不成立。
            // 表示将definition对象的内容存储到this.options中，形式this.options[components]['my-component']
            //如果传递的是一个对象，那么会执行 this.options._base.extend(definition)这行代码。
            //那么现在我们就明白了文档中的这句话的含义:注册组件，传入一个选项对象 (自动调用 Vue.extend)
            
          definition = this.options._base.extend(definition)
        }
      //如果是指令，那么第二个参数可以是可以是对象，也可以是函数。
      //如果是对象，直接执行 this.options[type + 's'][id] = definition这行代码
      //如果是函数，会将definition设置给bind与update这两个方法，
      //在官方手册中，有如下内容
      // 注册 (指令函数)
		//Vue.directive('my-directive', function () {
  			// 这里将会被 `bind` 和 `update` 调用
		//})
      //现在我们能够理解为什么会写`这里将会被 `bind` 和 `update` 调用`这句话了
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
      //最终注册的Vue.component,Vue.filter,Vue.directive都会存储到this.options['components']
      //this.options['filters'],this.options['directives']中。是一个全局的注册。
      //Vue.component,Vue.filter,Vue.directive 是全局注册的组件，过滤器，指令
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```

# 9、Vue实例成员初始化

通过前面的学习我们知道，实例成员在``src/core/instance/index.js`文件中，下面我们来看一下该文件中的如下方法内容：

```js
//注册vm的_init( )方法，初始化vm
initMixin(Vue);
//注册vm（Vue实例）的$data/$props/$set/$delete/$watch 属性或方法
stateMixin(Vue);
//初始化事件相关的方法
//$on/$once/$off/$emit
eventsMixin(Vue);
//初始化生命周期相关的混入方法
// $forceUpdate/$destroy
lifecycleMixin(Vue);
//混入render
// $nextTick
renderMixin(Vue);

```

这些方法都是以`Mixin`进行结尾，表示混入的意思，并且传递的参数都是`Vue`的构造函数，也就是说这些方法都是为`Vue`混入一些成员。

`initMixin`方法的主要作用就是为`Vue`实例增加了`_init`方法，该方法在`Vue`的构造函数中被调用，该方法也是整个应用的入口。关于`_initMixin`方法中的代码，后面我们还会详细的讲解。

`stateMixin`方法，具体的代码如下：

```js
export function stateMixin(Vue: Class<Component>) {
  // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {};
  dataDef.get = function () {
    return this._data;
  };
  const propsDef = {};
  propsDef.get = function () {
    return this._props;
  };
  if (process.env.NODE_ENV !== "production") {
    dataDef.set = function () {
      warn(
        "Avoid replacing instance root $data. " +
          "Use nested data properties instead.",
        this
      );
    };
    propsDef.set = function () {
      warn(`$props is readonly.`, this);
    };
  }
//为Vue的原型上添加`$data`与`$props`属性。也就是完成了`$data`与`$props`的初始化
    //在这里为什么使用Object.defineProperty添加属性呢？
    //原因是：第三个参数，第三个参数就是一个约束。
    //dataDef和propsDef都是对象，并且为其添加了get,在开发环境中添加了set.
    //在访问`$data`或者是`$props`的时候会执行get,如果在开发环境中向`$data`或者是`$props`赋值会执行set，从而给出相应的错误提示信息
  Object.defineProperty(Vue.prototype, "$data", dataDef);
  Object.defineProperty(Vue.prototype, "$props", propsDef);

    //为Vue的原型上挂在了$set与$delete方法，与Vue.set和Vue.delete是一样的。
  Vue.prototype.$set = set;
  Vue.prototype.$delete = del;
//监视数据的变化，该方法后面还会在进行查看。
  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this;
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options);
    }
    options = options || {};
    options.user = true;
    const watcher = new Watcher(vm, expOrFn, cb, options);
    if (options.immediate) {
      try {
        cb.call(vm, watcher.value);
      } catch (error) {
        handleError(
          error,
          vm,
          `callback for immediate watcher "${watcher.expression}"`
        );
      }
    }
    return function unwatchFn() {
      watcher.teardown();
    };
  };
}

```

下面，我们看一下`eventsMixin`方法。

```
//初始化事件相关的方法
//$on/$once/$off/$emit
eventsMixin(Vue);
```

```
$on:注册事件
$once：注册事件，只能触发一次
$off：取消事件
$emit:是触发事件
```

下面我们只看一下`$on`中的代码，其它内容的代码，可以自己查看。

```js
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
    const vm: Component = this
    //如果event是一个数组，遍历该数组，给每一个事件，添加对应的处理函数。
    if (Array.isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$on(event[i], fn)
      }
    } else {
        //如果是字符串，则根据event(事件名称)从_events对象中查找对应内容，如果没有则指定一个空数组，
        //然后向数组中添加了对应的处理函数。这样每个事件都有了对应的处理函数。
      (vm._events[event] || (vm._events[event] = [])).push(fn)
      // optimize hook:event cost by using a boolean flag marked at registration
      // instead of a hash lookup
      if (hookRE.test(event)) {
        vm._hasHookEvent = true
      }
    }
    return vm
  }
```

下面看一下`lifecycleMixin`方法：

```js
//初始化生命周期相关的混入方法
//  _update/$forceUpdate/$destroy
lifecycleMixin(Vue);
```

```js
 // _update方法的作用就是把虚拟DOM转换成真实的DOM
//首次渲染的时候会调用，数据更新会调用
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const restoreActiveInstance = setActiveInstance(vm)
    vm._vnode = vnode
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
   //首次渲染
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // updates（数据更新）
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    restoreActiveInstance()
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  }
```

其它方法，后续再看。

下面看一下`renderMixin`函数

```js
//混入render
// $nextTick
renderMixin(Vue);
```

```js
 installRenderHelpers(Vue.prototype)//安装了渲染有关的帮助方法
```

`installRenderHelpers`内部实现：

```js
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode //创建一个虚拟文本节点
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier
}

```

以上这些函数都是在编译的时候会用到，也就是将`template`模板编译成`render`函数的时候。

在`installRenderHelpers` 创建了`$nextTick`,关于这块内容，我们后期再来查看

```js
Vue.prototype.$nextTick = function (fn: Function) {
    return nextTick(fn, this)
  }
```

下面有一个`_render`方法，在该方法中有一个非常重要的代码。

```js
  vnode = render.call(vm._renderProxy, vm.$createElement)
```

在上面的代码中首先看一下`render`的定义：

```js
const { render, _parentVnode } = vm.$options
```

从上面这行代码中，我们知道了`render`来自于`vm.$options`,那么就表明这个`render`是在用户创建`Vue`的实例的时候执行的的`render`.

`render.call`范围`render`方法的调用，第一个参数改变`this`的指向，第二个参数：` vm.$createElement`其实就是我们在创建`Vue`的实例的时候，指定的`h`函数。`h`函数的作用就是创建虚拟`DOM`

以上就是我们这一小节讲解的内容，当然其内部还有一些其它方法，这些我们会在后面在继续查看。

# 10、init方法

在``src/core/instance/index.js`文件中,创建了`Vue`的构造函数，并且在其内部调用了`_init( )方法`,而该方法的初始化是在`initMixin(Vue);`方法中完成的，下面我们再来看一下该方法的实现。`_init( )`方法完成了初始化的工作，所以我们重点看一下该方法中初始化了哪些内容？

```js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
      //Vue的实例
    const vm: Component = this
    // a uid
      //唯一标识
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
      //如果是Vue实例，不需要被observe处理。
    vm._isVue = true
    // merge options
      // 合并options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
        //将用户掺入的options与Vue构造函数中的options进行合并、
        //在初始化静态成员的时候已经为Vue构造函数初始化了v-show,v-model,keep-alive等指令和组件
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
      //设置渲染的代理对象
    if (process.env.NODE_ENV !== 'production') {
        //开发环境调用initProxy方法
      initProxy(vm)
    } else {
        //渲染的时候设置的代理对象就是Vue的实例
        //在渲染的时候会用到该属性。
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
      //下面的函数是完成Vue实例的一些初始化操作。
      //初始与生命周期相关的属性（$children,$parent,$root,$refs）
    initLifecycle(vm)
      //初始化当前组件的事件      
    initEvents(vm)
      //初始化了render中所使用的h函数
      //同时还初始化了$slots/$attrs 等等
      //在initRender方法中，注意如下两行代码
               //   vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
              //vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
      //当我们在创建一个Vue实例的时候，是可以直接传递一个render函数，render函数需要一个参数就是h函数，
      //$createElement就是传递过来的h函数。其作用就是将虚拟DOM转换成真实DOM
      //当我们把template编译成render函数的时候，在内部调用的是_c这个函数，在模板编译的过程中，会看到这个方法
      // 而$createElement函数，是在new Vue实例的时候传递的render函数所调用的。
    initRender(vm)
      //触发生命周期中的beforeCreate钩子函数
    callHook(vm, 'beforeCreate')
      //初始化inject，把inject的成员注入到Vue的实例上
    initInjections(vm) // resolve injections before data/props
      //初始化Vue实例中的methods/computed/watch等，关于该函数会在下一小节中进行讲解
    initState(vm)
      // 初始化provide
    initProvide(vm) // resolve provide after data/props
      //触发生命周期中的created钩子函数
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }
//调用$mount挂载整个页面，并且进行页面的渲染
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

`initProxy`方法的实现

```js
onst hasProxy =
    typeof Proxy !== 'undefined' && isNative(Proxy)
//如果有Proxy，通过new Proxy来创建一个代理。
  if (hasProxy) {
    const isBuiltInModifier = makeMap('stop,prevent,self,ctrl,shift,alt,meta,exact')
    config.keyCodes = new Proxy(config.keyCodes, {
      set (target, key, value) {
        if (isBuiltInModifier(key)) {
          warn(`Avoid overwriting built-in modifier in config.keyCodes: .${key}`)
          return false
        } else {
          target[key] = value
          return true
        }
      }
    })
  }
```

# 11、initState方法

`initState`方法的作用就是：初始化`Vue`实例中的`methods/computed/watch`等.

```js
export function initState(vm: Component) {
  vm._watchers = [];
    //获取options
  const opts = vm.$options;
    //初始化props,并且注入到Vue实例中
  if (opts.props) initProps(vm, opts.props);
    //初始化methods，把methods中的方法注册到Vue实例中，下面看一下initMethods方法的内部实现
  if (opts.methods) initMethods(vm, opts.methods);
    //如果options中有data属性会调用initData方法，下面查看一下initData方法内部实现
  if (opts.data) {
    initData(vm);//初始化data
  } else {
      //如果没有data属性，会为Vue的实例创建一个空对象，并且将其修改成响应式的。关于响应式的内容我们后期还会查看。
    observe((vm._data = {}), true /* asRootData */);
  }
    //初始化computed，会把computed注册到Vue的实例中，可以自己查看源码
  if (opts.computed) initComputed(vm, opts.computed);
    //初始化watch，会把watch注册到Vue的实例中，可以自己查看源码
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch);
  }
}
```

`initMethods`方法实现

```js
function initMethods(vm: Component, methods: Object) {
  const props = vm.$options.props;//获取props
  for (const key in methods) {//对所有的methods进行遍历
    if (process.env.NODE_ENV !== "production") {//在开发环境中
      if (typeof methods[key] !== "function") {//获取对应的method如果不是函数，会给出相应的警告
        warn(
          `Method "${key}" has type "${typeof methods[
            key
          ]}" in the component definition. ` +
            `Did you reference the function correctly?`,
          vm
        );
      }
        // 如果method的名字与props中的属性名字重名也会给出相应的警告
      if (props && hasOwn(props, key)) {
        warn(`Method "${key}" has already been defined as a prop.`, vm);
      }
        //判断方法的名称是否为Vue的实例，同时判断方法的名称是否以_或者是$开头。
        //如果以 _ 开头表示一个私有属性，所以不建议方法名称以 _ 开头。
        // 以$开头的都是成员都是Vue的成员，所以也不建议方法名称使用$开头
      if (key in vm && isReserved(key)) {
        warn(
          `Method "${key}" conflicts with an existing Vue instance method. ` +
            `Avoid defining component methods that start with _ or $.`
        );
      }
    }
      //把method注册到Vue的实例中
      //首先判断从methods中取出来的方法如果不是"function",返回noop,也就是一个空函数。
      //如果是"funciton"，返回该函数，同时通过bind修改函数内部this的指向，然后指向到Vue的实例。
    vm[key] =
      typeof methods[key] !== "function" ? noop : bind(methods[key], vm);
  }
}
```

`initData`方法实现

```js
function initData(vm: Component) {
  let data = vm.$options.data;//获取props中的data内容
    //初始化_data,判断data的类型是不是一个函数如果不是直接返回data，如果是调用getData方法,getData中就是通过call来调用data函数。
    //其实就是初始化组件中的data,组件中的data就是一个函数，如果是Vue实例中的data,那么就是一个对象。
  data = vm._data = typeof data === "function" ? getData(data, vm) : data || {};
  if (!isPlainObject(data)) {
    data = {};
    process.env.NODE_ENV !== "production" &&
      warn(
        "data functions should return an object:\n" +
          "https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function",
        vm
      );
  }
  // proxy data on instance
    //获取data中的所有属性
  const keys = Object.keys(data);
    // 获取props
  const props = vm.$options.props;
    // 获取methods
  const methods = vm.$options.methods;
  let i = keys.length;
    //判断data中的成员是否和`props/methods`重名。在开发环境中有重名会出现相应的警告信息。
  while (i--) {
    const key = keys[i];
    if (process.env.NODE_ENV !== "production") {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        );
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== "production" &&
        warn(
          `The data property "${key}" is already declared as a prop. ` +
            `Use prop default value instead.`,
          vm
        );
    } else if (!isReserved(key)) {
        //isReserved方法就是判断当前的属性是否以`_`和`$`开头，如果是以`_`和`$`开头就不会把当前的属性注入到Vue实例中。否则会将当前属性（key的值）注册到Vue的实例中。
      proxy(vm, `_data`, key);
    }
  }
  // observe data
    //对data做响应式的处理。
  observe(data, true /* asRootData */);
}

```

下面可以看一下`proxy`方法中的代码。

```js
export function proxy(target: Object, sourceKey: string, key: string) {
    // 如果访问get，那么返回的就是this._data中当前属性的值
  sharedPropertyDefinition.get = function proxyGetter() {
    return this[sourceKey][key];
  };
  sharedPropertyDefinition.set = function proxySetter(val) {
    this[sourceKey][key] = val;
  };
    //把当前属性注入到Vue实例中
  Object.defineProperty(target, key, sharedPropertyDefinition);
}
```

# 12、总结

下面我们把上面讲解的内容，做一个总结：

在首次渲染的时候，首先对`Vue`进行初始化，同时完成实例成员与静态成员的初始化。初始化完成后会执行构造函数，在构造函数中调用了`_init`方法，该方法是整个`Vue`的入口，在该方法中调用了`vm.$mount`方法，通过前面的学习，我们知道有两个`$mount`, 首先第一个`$mount`来自于`src/platforms/web/entry-runtime-with-compiler.js`文件，这个`$mount`方法的作用就是把模板编译成`render`函数，当然，在把模板编译成`render`函数之前，先判断一下是否传入了`render`这个选项，如果没有传入，这时就会去获取`template`选项，如果也没有`template`这个选项，那么会把`el`中的内容作为模板，然后把模板编译成`render`函数。这里是通过`compileToFunctions`这个函数，把模板编译成`render`函数。把编译好的`render`函数存储到`options.render`中。

下面会调用`src/platforms/web/runtime/index.js`文件中的`$mount`方法。在这个方法中会重新获取`el`,因为运行时版本，是不会执行`src/platforms/web/entry-runtime-with-compiler.js`文件中的代码，所以这里需要重新获取`el`,下面调用`mountComponent( )`方法，该方法定义的文件为`src/core/instance/lifecycle.js`,在`mountComponent( )`方法中，首先判断是否有`render`选项，如果没有但是传入了模板，并且当前是开发环境，那么会打印一个警告信息。警告信息为运行时版本不支持编译器。下面触发了`beforeMount`这个钩子函数，然后定义`updateComponent`,在`updateComponent`中，调用了`vm._render( )`函数和`vm._update`函数。

`vm._render()`函数的作用就是生成虚拟`DOM`（在 `vm._render()`这个方法中，调用了在创建`Vue`实例的时候传入的`render`函数,或者是将`template`编译成的`render`函数）,`vm._update( )`函数的作用就是将虚拟`DOM`转换成真实的`DOM`（在`vm._udpate`这个方法中调用了`vm.__patch__`方法将虚拟`DOM`转换成了真实的`DOM`然后挂在到页面中）.接下来创建了`Watcher`的实例，在`Watcher`实例中调用了`updateComponent`, 接下来会执行`mounted`这个钩子函数，完成挂在，最后返回`Vue`的实例。

# 13、响应式处理入口

通过查看 源码解决如下的问题

-   `vm.msg={count:0}` 重新给属性赋值，是否是响应式的？
- `vm.arr[0]=4` 给数组元素赋值，视图是否会更新
- `vm.arr.length=0` 修改数组的`length`,视图是否会更新
- `vm.arr.push(5)` 视图是否会更新

响应式处理的入口

 整个响应式处理的过程是比较复杂的，下面我们先查看`src/core/instance/init.js`文件,在该文件中有`initState(vm)`方法，该方法完成了`vm`状态的初始化，初始化了`_data`,`_props`,`methods`等。

然后我们再来看一下，`src/core/instance/state.js`

```js
//数据的初始化
if(opts.data){
    initData(vm)//把data中的成员注入到Vue实例，并且转换成响应式的对象
}else{
    //如果options选项中没有data，这里会将data初始化一个空对象。传入到observe这个方法中转换成响应式的对象
    observe(vm._data={},true)//observe就是响应式的入口。
}
```

下面看一下`src/core/instance/init.js`文件，在该文件中找到`initState(vm)`方法，该方法就是注册`vm`（`Vue`实例）的`$data/$props/$set/$delete/$watch` 属性或方法.当我们单击进入该方法后，可以看到该方法所在的文件为`src/core/instance/state.js`.

在该文件中找到如下代码

```js
 if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  } 
```

下面在进入`initData`方法，代码如下：

```js
function initData(vm: Component) {
  let data = vm.$options.data;//获取props中的data内容
    //初始化_data,判断data的类型是不是一个函数如果不是直接返回data对象，如果是调用getData方法,getData中就是通过call来调用data函数。
    //其实就是初始化组件中的data,组件中的data就是一个函数，如果是Vue实例中的data,那么就是一个对象。
  data = vm._data = typeof data === "function" ? getData(data, vm) : data || {};
  if (!isPlainObject(data)) {
    data = {};
    process.env.NODE_ENV !== "production" &&
      warn(
        "data functions should return an object:\n" +
          "https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function",
        vm
      );
  }
  // proxy data on instance
    //获取data中的所有属性
  const keys = Object.keys(data);
    // 获取props
  const props = vm.$options.props;
    // 获取methods
  const methods = vm.$options.methods;
  let i = keys.length;
    //判断data中的成员是否和`props/methods`重名。在开发环境中有重名会出现相应的警告信息。
  while (i--) {
    const key = keys[i];
    if (process.env.NODE_ENV !== "production") {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        );
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== "production" &&
        warn(
          `The data property "${key}" is already declared as a prop. ` +
            `Use prop default value instead.`,
          vm
        );
    } else if (!isReserved(key)) {
        //isReserved方法就是判断当前的属性是否以`_`和`$`开头，如果是以`_`和`$`开头就不会把当前的属性注入到Vue实例中。否则会将当前属性（key的值）注册到Vue的实例中。
      proxy(vm, `_data`, key);
    }
  }
  // observe data
    //对data做响应式的处理。
  observe(data, true /* asRootData */);
}

```

在上面的代码中，我们可以看到最后调用了`observe`方法。`data`就是传递过来的`options`选项中的`data`,第二个参数为`true`,表示的就是根数据,根数据会做相应的处理。

```js

/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */
//试图为value创建一个Observer对象,如果创建成功，将创建的Observer返回，或者返回一个已经存在的Observer对象。也就是说valu这个参数如果已经有Observer对象，直接返回。
export function observe (value: any, asRootData: ?boolean): Observer | void {
  // 判断 value 是否是对象,如果不是对象，或者是VNode，直接返回，不做响应式的处理
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void //声明一个Observer类型的变量
  // 判断value中是否有'__ob__'这个属性，如果有，那么就需要判断value中的ob这个属性是否为Observer的实例
  //如果value中的ob属性是Observer的实例，在这就赋值给ob这个变量。最后直接返回ob.
  //这一点就和最开始我们说的是一样的，也是如果已经存在Observer对象，直接返回,相当于做了一个缓存的效果。
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
      //如果value中没有ob这个属性，那么就需要创建一个Observer对象。
      //在创建Observer对象之前，需要做一些判断的处理。
      //这里我们重点看一下,如下的判断      
        //(Array.isArray(value) || isPlainObject(value)) &&
    //Object.isExtensible(value) &&
    //!value._isVue
      //(Array.isArray(value):判断传递过来的value是否为一个数组
      //isPlainObject(value)):判断value是否为一个对象。
      //!value._isVue:判断value是否为一个Vue的实例，在core/instance/index.js文件中，调用了initMixin方法，
      //在该方法中，设置了_isVue这个属性，如果传递过来的的value是Vue的实例就不要通过Observer设置响应式。
      //如果value可以进行响应式的处理，就需要创建一个Observer对象。
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    // 创建一个 Observer 对象
      //在Observer中把value中的所有属性转换成get与set的形式。
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

这里最重要的就是`Observer`对象的创建，下一小节我们来看一下`Observer`中是怎样处理的。

# 14、Observer

在上一小节中，我们已经找到了响应式的入口，`Observer`.（`src/core/observer/index.js`）

下面我们看一下`Observer`对应的代码。`Observer`是一个类

```js
/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
//Observer类附加到每个被观察的对象，一旦附加，observer就会转换目标对象的所有属性，将其转换成`getter/setter`.
//其目的就是收集依赖和派发更新。其实就是我们前面所讲的发送通知。
export class Observer {
  // 观察对象
  value: any;
  // 依赖对象
  dep: Dep;
  // 实例计数器
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    // 初始化实例的 vmCount 为0
    this.vmCount = 0
    // 将实例挂载到观察对象的 __ob__ 属性上。
    def(value, '__ob__', this)
    // 数组的响应式处理,后面再看具体的实现
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      // 为数组中的每一个对象创建一个 observer 实例
      this.observeArray(value)
    } else {
      // 遍历对象中的每一个属性，转换成 setter/getter
      this.walk(value)
    }
  }

```

下面 看一下对应的`walk`方法的实现。



```js
  walk (obj: Object) {
    // 获取观察对象的每一个属性
    const keys = Object.keys(obj)
    // 遍历每一个属性，设置为响应式数据
    for (let i = 0; i < keys.length; i++) {
        //该方法就是将对象中的属性转换成getter和setter.
        //当然在将属性转换成getter/setter前，也做了其它的一些处理，例如收集依赖，当数据发生变化后，发送通知等。
        //下一小节，查看该方法的实现。
      defineReactive(obj, keys[i])
    }
  }

```

最好还调用了`observeArray`方法，该方法的作用就是将数组转换成响应式的。

```js
 observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
```

`Observer`类核心的作用就是对数组和对象做响应式的处理。

# 15、defineReactive

这一小节我们来看一下`Observer`类中的`defineReactive`方法，在上一小节中我们说过该方法的作用就是：就是将对象中的属性转换成`getter`和`setter`.

下面看一下`defineReactive`方法中的源码实现。

```js
// 为一个对象定义一个响应式的属性
/**
 * Define a reactive property on an Object.
 */
//shallow的值为true,表示只监听对象中的第一层属性，如果是false那就是深度监听，也就是说当key这个属性的值是一个对象，那么还需要监听这个对象中的每个值的变化。
export function defineReactive (
  obj: Object,//目标对象
  key: string, //转换的属性
  val: any,
  customSetter?: ?Function,//用户自定义的函数，很少会用到。
  shallow?: boolean
) {
  // 创建依赖对象实例，其作用就是为key收集依赖，也就是收集所有观察当前key这个属性的所有的watcher.
  const dep = new Dep()
  // 获取 obj 的属性描述符对象,在属性描述符中可以定义getter/setter. 还可以定义configurable,也就是该属性是否为可配置的。
  const property = Object.getOwnPropertyDescriptor(obj, key)
  //判断是否存在属性描述符并且configurable的值为false。如果configurable的值为false,表明是不可配置的，那么就不能通过delete将这个属性进行删除。、
  //也不能通过 Object.defineProperty进行重新的定义。
  //而在接下的操作中我们需要通过 Object.defineProperty对属性重新定义描述符，所以这里判断了configurable属性如果为false,则直接返回。
  if (property && property.configurable === false) {
    return
  }
  // 获取属性中的get和set.因为obj这个对象有可能是用户传入的，如果是用户传入的那么就有可能给obj这个对象中的属性设置了get/set.
    //所以这里先将用户设置的get和set存储起来，后面需要对get/set进行重写,为其增加依赖收集与派发更新的功能。
  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  //如果传入了两个参数（obj和key）,这里需要获取对应的key这个属性的值。
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }
  // 判断shallow是否为false.如果当前的shallow是false,那么就不是浅层的监听。那么需要调用observe，也就是val是一个对象，那么需要将该对象中的所有的属性转换成getter/setter，observe方法返回的就是一个Observer对象
  let childOb = !shallow && observe(val)
  //下面就是通过Object.defineProperty将对象的属性转换成了get和set  
  Object.defineProperty(obj, key, {
    enumerable: true,//可枚举
    configurable: true,//可以配置
    get: function reactiveGetter () {
    // 首先调用了用户传入的getter,如果用户设置了getter，那么首先会通过用户设置的getter获取对象中的属性值。
        //如果没有设置getter,直接返回我们前面获取到的值。
      const value = getter ? getter.call(obj) : val
      //下面就是收集依赖（这块内容我们在一下小节中再来讲解，下面再来看一下set）
      // 如果存在当前依赖目标，即 watcher 对象，则建立依赖
      if (Dep.target) {
        dep.depend()
        // 如果子观察目标存在，建立子对象的依赖关系
        if (childOb) {
          childOb.dep.depend()
          // 如果属性是数组，则特殊处理收集数组对象依赖
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      // 返回属性值
      return value
    },
    set: function reactiveSetter (newVal) {
      //如果用户设置了getter,通过用户设置的getter获取对象中的属性值，否则直接返回前面获取到的值。
      const value = getter ? getter.call(obj) : val
      // 如果新值等于旧值或者新值旧值为NaN则不执行
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // 如果没有 setter 直接返回
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      // 如果setter存在则调用，为对象中的属性赋值
      if (setter) {
        setter.call(obj, newVal)
      } else {
          //当getter和setter都不存在，将新值赋值给旧值。
        val = newVal
      }
      // 如果新值是对象，那么把这个对象的属性再次转换成getter/setter
        //childOb就是一个Observe对象。
      childOb = !shallow && observe(newVal)
      // 派发更新(发布更改通知)
      dep.notify()
    }
  })
}
```

# 16、依赖收集

在`defineReactive`方法中定义了`getter/setter`.在`getter`中做了收集依赖。依赖收集就是把依赖该属性的`watcher`对象，添加到`dep`中的`subs`数组中。

当数据发生变化后，通知数组中的`watch`,在前面的课程中，我们模拟过这个过程，下面看一下在`Vue`中是怎样实现的。

```js
//下面就是收集依赖
      // 判断Dep中是否有target属性，该属性中存储的就是watcher对象
      if (Dep.target) {
          //depend方法就是进行依赖收集，就是把watch对象添加到Dep中的subs数组中。
        dep.depend()
        // 如果子对象存在，建立子对象的依赖关系
        if (childOb) {
            //每一个Observer对象，都有一个dep对象，然后调用depend方法建立子对象的依赖关系。
          childOb.dep.depend()
          // 如果属性是数组，则收集数组对象依赖
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
```

现在我们对以上代码有了一个基本的了解，下面我们思考一个问题是,在什么时候给`Dep`的`target`属性赋值的？要回答这个问题，我们首先要考虑的是，什么时候创建`Watcher`对象的。

在`src/core/instance/lifecycle.js`文件中创建了`Watcher`对象。

找到该文件中的`mountComponent`方法，在该方法中创建了`Watcher`对象。

```js
  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
```

下面进入`Watcher`的内部，看一下在其内部是怎样给`Dep`的`target`属性赋值的。在其内部有个`get`方法，在该方法的内部的内部调用了` pushTarget(this)`,在这里`this`就是`Watcher`对象，下面进入`pushTarget`方法的内部。

```js
// Dep.target 用来存放目前正在使用的watcher
// 全局唯一，并且一次也只能有一个watcher被使用
// The current target watcher being evaluated.
// This is globally unique because only one watcher
// can be evaluated at a time.
Dep.target = null
const targetStack = []
// 入栈并将当前 watcher 赋值给 Dep.target
// 父子组件嵌套的时候先把父组件对应的 watcher 入栈，
// 再去处理子组件的 watcher，子组件的处理完毕后，再把父组件对应的 watcher 出栈，继续操作
export function pushTarget (target: ?Watcher) {
    //将Watcher对象存储到栈中。
    //因为在V2.0以后每一个组件对应一个Watcher对象，如果组件之间有嵌套，先处理子组件，所以这时应该先将父组件的	`Watcher`存储起来，这里是存储到栈中了，
    //子组件处理完毕后，把父组件中的Watcher从栈中弹出，继续处理父组件。
  targetStack.push(target)
  Dep.target = target//给Dep中的target赋值。
}

export function popTarget () {
  // 出栈操作
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}

```

下面，我们查看一下` dep.depend()`这个方法内部的代码。我们知道`depend`方法的作用就是收集依赖。

```js
 // 将观察对象和 watcher 建立依赖
  depend () {
    if (Dep.target) {
      // 如果 target 存在，把 dep 对象添加到 watcher 的依赖中
        
      Dep.target.addDep(this)
    }
  }
```

`Dep.target`指的就是`Watcher` ,所以接下来我们查看`observer/watcher.js`这个文件中的代码。

```js
  addDep (dep: Dep) {
    const id = dep.id//唯一表示，每次创建一个Dep对象的时候，会让该编号加1，这里可以进入Dep中查看。例如，在页面中有两个{{msg}}，针对这个属性，只会收集一次依赖，即使使用两次msg，这样就避免了重复收集依赖。
    if (!this.newDepIds.has(id)) {
        //如果在newDepIds集合中没有id,将其添加到该集合中。
      this.newDepIds.add(id)
        //将dep对象添加到newDeps集合中。
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
          // 调用dep对象中的addSub方法，将Watcher对象添加到subs数组中。this为Watcher对象。
        dep.addSub(this)
      }
    }
  }
```

下面我们来看一下`Dep`中的`addSub`方法

```js
  // 将Watcher对象添加到subs数组中。
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

```

# 17、数组的响应式处理

数组的响应式处理的核心代码在`Observer`类的构造函数中(`/core/observer/index.js`)

```js
 constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    // 初始化实例的 vmCount 为0
    this.vmCount = 0
    // 将实例挂载到观察对象的 __ob__ 属性
    def(value, '__ob__', this)
    // 数组的响应式处理
    if (Array.isArray(value)) {
      //  export const hasProto = '__proto__' in {}
        //判断当前浏览器是否支持对象的原型这个属性，目的完成浏览器兼容的处理
      if (hasProto) {
          //支持对象的原型，则调用如下的函数，
          //value是数组，
          //arrayMethods:数组相关的方法。
          //该方法重新设置数组的原型属性，对应的值为arrayMthods.
        protoAugment(value, arrayMethods)
      } else {
          
        copyAugment(value, arrayMethods, arrayKeys)
      }
      // 为数组中的每一个对象创建一个 observer 实例
      this.observeArray(value)
    } else {
      // 遍历对象中的每一个属性，转换成 setter/getter
      this.walk(value)
    }
  }
```



`arrayMethods`的实现如下：

```js
//数组构造函数的原型
const arrayProto = Array.prototype
//使用Object.create创建一个对象，让对象的原型指向arrayProto，也就是数组的prototype
export const arrayMethods = Object.create(arrayProto)
//我们可以看到，如下内容都是数组中的方法，而且这些方法会对数组进行修改，例如push向数组增加内容，造成了原有数组的更新。
//而当数组中的内容发生了变化后，我们要调用Dep中的notity方法发送通知。通知watcher,数据发生了变化，要重新更新视图。
//但是数组的原生方法不知道Dep,也就不会调用Dep中的notity方法。所以说要做一些处理。下面看一下怎样进行处理的。
const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
//对methodsToPatch数组进行遍历。
//method表示的是从methodsToPatch数组中取出来的方法的名字
methodsToPatch.forEach(function (method) {
  // cache original method
    // arrayProto：数组的原型，这里就是获取数组的原始方法，例如push,pop等
  const original = arrayProto[method]
  // 调用 Object.defineProperty() ，将method中存储的方法的名字，重新定义到arrayMthods,也就是给arrayMthods对象，重新定义push,pop等这些方法。
  // 方法的值，就是defineProperty()方法的第三个参数mutator，该方法需要参数args:该参数中存储的就是我们在调用push或者是pop时传递的内容。
  def(arrayMethods, method, function mutator (...args) {
    // 执行数组的原始方法
    const result = original.apply(this, args)
    // 获取数组关联的Observer对象
    const ob = this.__ob__
    //存储数组中新增的内容，例如如果是push,unshift，则将args赋值给inserted，因为这时args存储的就是新增的内容。
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
            //如果是splice方法，那么把第三个值存储到inserted中。
        inserted = args.slice(2)
        break
    }
    // 如果有新增的元素，将会重新遍历数组中的元素，并且将其设置为响应式数据。也就是说，调用push,unshift,splice方法，向数组中添加的内容都是响应式的。
    if (inserted) ob.observeArray(inserted)
    // notify change
    // 找到Observer中的dep对象，调用其中的notify方法来发送通知
    ob.dep.notify()
      //返回方法执行的结果。
    return result
  })
})
```

以上就是对数组中的会修改数组原有内容的方法的处理。具体的过程就是先调用数组中的原始方法，例如`push`,`pop`等这些方法。

下面就是找到对数组进行新增元素的这些方法，例如:`push`,`unshift`,`splice，`,如果新增了元素，就调用`observer`中的`observerArray`这个方法，去遍历数组中的这些新增的元素，然后转换成响应式。当调用了数组中的新增元素的这些方法后，会发送通知。最后返回方法执行的结果。

下面我们再回到`Observer`类的构造函数中(`/core/observer/index.js`)。

如果浏览器不支持原型属性会调用`copyAugment`方法，该方法有三个参数，前两个参数与`protoAugment`方法参数的含义是一样的，第三个参数是`arrayKeys`.

`arrayKeys`具体的含义是：

```js
const arrayKeys = Object.getOwnPropertyNames(arrayMethods)
```

获取数组中的`push`,`pop`等方法的名字，`arrayKeys`是一个数组。

下面看一下`copyAugment`方法中的代码

```js
function copyAugment (target: Object, src: Object, keys: Array<string>) {
    //变量传递过来的数组中方法的名字
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i]
    //给数组对象重新定义这些数组的方法，例如pop,push. 当然这些方法都是经过处理的。该方法的作用与protoAugment方法的一样
    def(target, key, src[key])
  }
}
```

下面我们再来看一下`observeArray`方法。

遍历数组中的成员，为数组中的每个对象设置为响应式的对象。

```js
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```



# 18、数组练习88888

```html
<div id="app">
    {{arr}}
</div>
<script src="vue.js"></script>
<script>
  const vm=new Vue({
      el:'#app',
      data:{
          arr:[2,3,5]
      }
  })
  //看一下如下操作是否为响应式
  // vm.arr.push(8)
  // vm.arr[0]=100
  // vm.arr.length=0
</script>
```

通过前面的代码的阅读，我们知道`push`方法，在`Vue`中做了一定的处理，当通过`push`方法向数组中添加了一个新的数据后，对应的视图页面也会进行更新。

`vm.arr[0]=100`,会修改数组中的内容，但是当数组中的内容修改了以后，视图并没有发生任何的变化，所以这种操作并不是响应式的。也就是说通过数组的索引来

修改数组的时候，并没有调用`Dep`中的`notify`,也就没有通知`watcher`去重新渲染视图。通过源码，我们并没有发现对这种情况的处理，也就是没有监听数组中的每个属性（`index`,`length`都是数组的属性）将其转换成响应式，同理`vm.arr.length=0`也不是响应式的。

在源码中，我们看的是对数组进行遍历，对数组中的每个元素中是对象的元素转换成了响应式。

如果现在我们想让` vm.arr[0]=100`和` vm.arr.length=0`这种操作变成响应式的效果，应该怎样实现呢？

首先，我们先来看` vm.arr[0]=100`,就是修改数组中的第一个元素，这里我们可以使用`splice`方法来达到相同的目的，而且通过前面阅读源码我们知道，`splice`方法是响应式的，也就是通过该发你规范修改完了数组后，对应的视图也会进行。所以对应的代码为：

```js
//第一个参数0，表示删除arr数组中的第一个元素。
//第二个参数1.表示的是删除的个数
//第三个参数100，表示的是把删除的元素用100来替换。
vm.arr.splice(0,1,100)

```

下面我们再来看一下关于`vm.arr.length=0`,表示的是清空数组中的内容。

为了达到响应式的效果，这里也可以使用splice方法来完成，清空数组的操作。

```js
vm.arr.splice(0)//清空数组中的所有元素
```

通过查看源码我们对上面的问题有了更加深入的理解。

# 19、Watcher

关于`Watcher`我们首先会查看一下在首次渲染的时候的执行过程，然后再来看一下当数据发生变化后，`Watcher`的执行过程。

`Watcher`分为三种，`Computed Watcher`(计算属性，本质也是通过`Watcher`来实现的),用户`Watcher`(侦听器)，渲染`Watcher`

前面两种`Watcher`是在`initState`的时候初始化的。

下面我们来复习一下首次渲染的时候`Watcher`的执行过程。

`/src/core/instance/lifecycle.js`中的`mountComponent`组件中创建的。

如下代码所示：

```js
 //vm:Vue的实例
// updateComponent
//noop空函数，
 new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
          //触发beforeUpate方法。
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)//ture表示渲染的watcher.
```

下面进入`Watcher`类中，看一下做了哪些事情。以下的代码是`Watcher`类的构造函数。

```js
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
      //将Vue的实例记录到了vm这个属性中。
    this.vm = vm
    if (isRenderWatcher) {
        //如果是渲染`Watcher`,则将`Watcher`的实例保存到`Vue`实例中的`_watcher`属性中。
      vm._watcher = this
    }
      //将所有的`Watcher`实例都保存到`Vue`实例中的`_watchers`这个数组中。
      // _watchers数组中不仅存储了渲染`Watcher`,还存储了计算属性对应的watcher,还有就是侦听器。
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
      //创建渲染watcher的实例的时候，传递过来的cb函数就是一个noop空函数。
      //如果是用户创建的`Watcher`的时候，传递过来的就是一个回调函数。
    this.cb = cb
    this.id = ++uid //为了区分每个watcher,创建一个编号 uid for batching
    this.active = true// active表示这个watcher是否为活动的watcher,如果为true,则为活动的watcher.
    this.dirty = this.lazy // for lazy watchers
      //下面的集合记录的都是Dep
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
      //判断expOrFn是否为函数，我们在前面创建渲染watcher的时候，传递过来的updateComponent函数给了expOrFn这个参数。
    if (typeof expOrFn === 'function') {
        //将函数保存到了getter中。
      this.getter = expOrFn
    } else {
      // expOrFn 是字符串的时候，也就是创建侦听器的时候传递的内容，例如 watch: { 'person.name': function... }
        // 这时候侦听器侦听的内容是字符串，也就是person.name
      // parsePath('person.name') 返回一个函数获取 person.name 的值
        //parsePath的作用就是生成一个函数，来获取`person.name`的值。将返回的函数记录到了getter中。记录geeter中的目的，就是当获取属性值的时候会触发对应的getter,当触发getter的时候会触发依赖。
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
      //如果是计算属性，lazy的值为true,表示延迟执行。如果是渲染watcher,会立即调用get方法。
    this.value = this.lazy
      ? undefined
      : this.get()
  }

```

下面就是`get`方法的源码。

```js
  get () {
      //把当前的watcher对象入栈，并且把当前的watcher赋值给Dep的target属性。
      //当有父子组件嵌套的时候，先将父组件的watcher入栈，然后对子组件进行处理，处理完毕后，在从栈中获取父组件的watcher进行处理。
    pushTarget(this)
    let value
    const vm = this.vm
    try {
        //对getter函数进行调用，如果是渲染函数，这里调用的是updateComponent
        //当updateComponent函数执行完毕后会将虚拟DOM转换成真实的DOM,然后渲染到页面中。
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
          //表示进行深度监听，深度监听表示的就是如果监听的是一个对象的话，会监听这个对象下的子属性、
        traverse(value)
      }
        //处理完毕后，做一些清理的工作，例如将watcher从栈中弹出
      popTarget()
        //将Watcher从subs数组中移除
      this.cleanupDeps()
    }
    return value
  }
```

以上就是首次渲染的时候，`Watcher`的执行过程。

**下面，我们来看一下当数据发生更新的时候，`Watcher`是怎样工作的？**

下面我们来查看一下`observer/dep.js`

我们知道当数据发生了变化后，会调用`Dep`中的`notify`这个方法，下面我们来看一下该方法的代码，如下所示:

```js

  // 发布通知
  notify () {
    // stabilize the subscriber list first
      //subs数组中存储的就是`watcher`对象，调用slice方法实现了克隆，因为下面会对subs数组中的内容进行排序。
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
        //按照Watcher对象中的id值进行从小到大的排序，也就是按照`watcher`的创建顺序进行排序，从而保证了在执行`watcher`的时候顺序是正确的。
      subs.sort((a, b) => a.id - b.id)
    }
    // 调用每个watcher对象的update方法实现更新
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```

在上面的代码中，对`subs`数组进行遍历，然后获取对应的`Watcher`,然后调用`Watcher`对象的`update`方法，下面我们来看一下`update`这个方法中的内容

```js
update () {
    /* istanbul ignore else */
    //在渲染watcher的时候，把lazy属性与sync属性设置为了false
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
        //渲染watcher会执行queueWatcher,
        //该方法的作用就是将watcher的实例放到一个队列中。
      queueWatcher(this)
    }
  }

```

下面，我们来查看一下`queueWatcher`方法内部的代码：

```js
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id//获取watcher的id属性
  //has是一个对象，下面获取has中的值，如果为null,表示当前这个watcher对象还没有被处理。
  //下面加这个判断的目的，就是为了防止watcher被重复性的处理。
  if (has[id] == null) {
    has[id] = true//把has[id]设置为true,表明当前的watcher对象已经被处理了。
      //下面就是开始正式的处理watcher
      //flushing为true,表明queue这个队列正在被处理。队列中存储的是watcher对象，也就是watcher对象正在被处理。
      //如果下面的判断条件成立，表明没有处理队列，那么就将watcher放到队列中。
    if (!flushing) {
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
        //如果执行else表明队列正在被处理，那么这里需要找到队列中一个合适位置，然后把watcher插入到队列中。
        //那么这里是怎样获取位置的呢？
        //首先获取队列的长度。
        //index表示现在处理到了队列中的第几个元素，如果i大于index,则表明当前这个队列并没有处理完。
        //下面需要从后往前，取到队列中的每个watcher对象，然后判断id是否大于watcher.id,如果大于正在处理的这个watcher的id,那么这个位置就是插入watcher的位置
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
       //下面就是把待处理的watcher放到队列的合适位置。
      queue.splice(i + 1, 0, watcher)
        
        //上面的代码其实就是把当前将要处理的watcher对象放到队列中。
        //下面就开始执行队列中的watcher对象。
    }
    // queue the flush
      //下面判断的含义就是判断一下当前的队列是否正在被执行。
      //如果watiing为false,表明当前队列没有被执行，下面需要将waiting设置为true.
    if (!waiting) {
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
          //开发环境直接调用下面的flushSchedulerQueue方法
          //flushSchedulerQueue方法的作用会遍历队列，然后调用队列中每个watcher的run方法。
        flushSchedulerQueue()
        return
      }
        //生产环境会将flushSchedulerQueue函数传递到nextTick函数中，后面再来讲解nextTick的应用。
      nextTick(flushSchedulerQueue)
    }
  }
}

```

下面我们来看一下`flushSchedulerQueue`方法内部的代码。

```js
function flushSchedulerQueue () {
  currentFlushTimestamp = getNow()
  flushing = true//将flushing设置为true,表明正在处理队列
  let watcher, id

  // Sort queue before flush.
  // This ensures that:
  //组件的更新顺序是从父组件到子组件，因为先创建了父组件后创建了子组件
  // 1. Components are updated from parent to child. (because parent is always
  //    created before the child)
  //组件的用户watcher,要在渲染watcher之前运行，因为用户watcher是在渲染watcehr之前创建的。
  // 2. A component's user watchers are run before its render watcher (because
  //    user watchers are created before the render watcher)
  // 如果一个组件，在父组件执行前被销毁了，那么对应的watcher应该跳过。
  // 3. If a component is destroyed during a parent component's watcher run,
  //    its watchers can be skipped.
  
  //对队列中的watcher进行排序，排序的方式是根据对应id，从小到大的顺序 进行排序。也就是按照watcher的创建顺序进行排列。
  //为什么要进行排序呢？上面的注释已经给出了三点的说明
  queue.sort((a, b) => a.id - b.id)

  // do not cache length because more watchers might be pushed
  // as we run existing watchers
    //以上注释的含义：不要缓存length,因为watcher在执行的过程中，还会向队列中放入新的watcher.
  for (index = 0; index < queue.length; index++) {
      //对队列进行遍历，然后取出当前要处理的watcher.
    watcher = queue[index]
    if (watcher.before) {
        //判断是否有before这个函数，该函数是在渲染watcher中具有的一个函数。其作用就是触发beforeupdate这个钩子函数。
        //也就是说走到这个位置beforeupate这个钩子函数被触发了。
      watcher.before()
    }
    id = watcher.id//获取watcher的id
    has[id] = null//将has[id]的值设为null,表明当前的watcher已经被处理过了。
    watcher.run()//执行watcher中的run方法。下面看一下run方法中的源码。
    // in dev build, check and stop circular updates.
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > MAX_UPDATE_COUNT) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}"`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }

 
```

下面是`run`方法的实现代码：

```js
run () {
    //标记当前的watcher对象是否为存活的状态。active默认值为true,表明可以对watcher进行处理。
    if (this.active) {
      const value = this.get()//调用watcher对象中的get方法，在get方法中会进行判断，如果是渲染watcher会调用updatecomponent方法，来渲染组件，更新视图
      //对于渲染watcher来说，对应的updateComponent方法是没有返回值，所以常量value的值为undefined.所以下面的代码不在执行，但是是用户 watcher,那么会调用其对应的回调函数，我们创建侦听器的时候，指定了回调函数。
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }
```

以上就是数据更新后，`watcher`的执行过程。

我们总结一下：当数据发生了变化后，会调用`Dep`中的`notify`方法去通知watcher,首先会将`watcher`放入到一个队列中，然后遍历队列，调用`Watcher`对象的`run`方法，在`run`方法中调用了渲染`watcher`的`updatecomponet`这个函数来渲染组件，更新视图，以上就是整个的处理过程。

# 20、总结响应式处理的过程

响应式是从`Vue`实例的`initState`方法开始的，在`initState`中完成了`Vue`实例状态的初始化，在`initState`方法的内部调用了`initData`方法，该方法的作用就是把`data`属性注入到了`Vue`的实例中，并且在其内部调用了`observe`方法，在`observe`方法中把`data`对象转换成响应式对象。所以说`observe`就是响应式的入口，下面我们来看一下在`observe`方法中做了哪些事情：

`observe`方法所在的位置`src/core/observer/index.js`.

在调用`observe`方法的时候，会传递一个参数`value`,所以在`observe`方法中首先会判断一下传递过来的参数`value`是否为对象，如果不是对象直接返回。

然后判断`value`对象是否有`__ob__`属性,如果有说明之前已经对其做过响应式的处理，所以直接返回。

如果没有`__ob__`属性，在创建`observer`对象，最后将`observer`对象返回。

那在创建`observer`对象的时候又做了哪些事情呢？

`observer`对象是有`Observer`类创建的(位置:`src/core/observer/index.js`),在`Observer`类的构造函数中，给`value`对象定义不可枚举的`__ob__`属性，并且通过该数据记录当前的`observer`对象。接下来进行了数组的响应式处理与对象的响应式处理。

在对数组进行响应式处理的时候，主要是设置了数组常用的方法，例如`push`,`pop`等。这些方法会改变原数组，所以当这些方法调用的时候，会发送通知。

在发送通知的时候，找到数组对应的`__ob__`属性,也就是`observer`对象，再找到`observer`对象的`dep`,然后调用`dep`中的`notify`方法。

更改了这些数组的方法后，下面就开始遍历数组中的每个成员，对每一个成员再去调用`observer`,如果这个成员是对象，也会将这个对象转换成响应式。

这就是关于数组的响应式处理。

下面我们再来看一下关于对象的响应式处理。关于对象的响应式处理，调用的是`walk`方法。

在`walk`方法内部会遍历对象中的所有属性，对每一个属性调用`defineReactive`方法（位置：`src/core/observer/index.js`），在`defineReactive`方法的内部会为每一个属性创建`dep`对象。让`dep`收集依赖。如果当前对象的属性值是对象，则会调用`observe`,也就是说如果当前对象的属性是对象，调用`observe`方法的目录就是把这个对象也转换成响应式对象。

在`defineReactive`方法的内部定义了`getter`和`setter`.

在`getter`中收集依赖，当然在收集依赖的时候，会为每一个属性收集依赖。如果属性的值为对象，也要收集依赖。`getter`方法最终会返回属性的值。

下面看一下`setter`,在`setter`方法中会保存新值，如果新值是对象也会调用`observe`,把这个新设置的对象也转换为响应式的对象。在`setter`方法椎间盘每个。数据发生变化，所以会派发通知，调用`dep.notify`方法。

下面我们再来看一下关于`收集`依赖的过程。在收集依赖的过程中，首先会调用`watcher`对象的`get`方法，在`get`方法中调用了`pushTarget`,在该方法中会将当前的`watcher`对象记录到`Dep.target`属性中。

在访问`data`中的成员的时候收集依赖，当访问属性的值时候会触发`defineReactive`中的`getter`方法来收集依赖。这时候会把属性对应的`watcher`对象添加到`dep`的`subs`数组中。如果属性的值也是对象，这时会创建一个`childOb`对象，为子对象收集依赖，目的就是在子对添加或者是删除成员的时候发送通知。

下面我们再来看一下`Watcher`,当数据发生变化的时候会调用`dep.notify`方法发送通知，同时在内容调用了`update`方法，在该方法中又调用了`queueWatcher`方法，在`queueWatcher`方法中会判断当前的`watcher`是否被处理了。如果没有处理，在添加到`queue`队列中，并调用了`flushSchedulerQueue()`方法，在该方法中触发了`beforeUpdate`这个钩子函数，然后调用了`watcher.run`方法，在该方法中最终调用了`updateComponent`方法（当前是渲染`watcher`）.这时已经将数据更新到了视图中，那么我们在页面中看到了最新的数据。最后触发了`actived`钩子函数和`updated`钩子函数。

# 21、动态添加一个响应式属性

在这里我们考虑一个问题，就是给一个响应式对象，动态增加一个属性，那么这个属性是否为响应式的呢？

下面我们通过如下代码来演示。

```html
<body>
    

<div id="app">
    {{obj.title}}
    <hr>
    {{obj.name}}
    <hr>
    {{arr}}
</div>
<script src="./vue.js"></script>
<srcipt>
	const vm=new Vue({
     el:'#app',
     data:{
    	obj:{
    	title:'Hello World'
   		 },
    	arr:[2,2,3]
    }	
    })

</srcipt>
    </body>
```

在上面的代码中，`data`中定义了一个对象`obj`,并且`obj`中有一个属性`title`,并且展示在了视图中，

同时在视图中还展示了`obj`对象中的`name`属性，但是我们并没有在`obj`对象中创建`name`属性。下面我们会动态的向

`obj`对象中动态的增加一个属性`name`,看一下是否会渲染视图。

下面我们在浏览器中打开上面的页面，会展示`title`与`arr`数组中的内容。

但是由于没有`name`，所以不会展示。

打开浏览器的控制台，在`Console`中输入如下代码：

```
vm.obj.name="abc"
```

执行完上面的代码后，并没有看到对应的视图的变化，所以动态添加的`name`属性并不是响应式的。

如果我们这里有这个需求，可以通过`vm.$set`或者是`Vue.set`来解决(这两个方法是一样的)。

如下代码：

```
vm.$set(vm.obj,'name','zhangsan')
```

上面的代码，通过`Vue`的实例调用了`$set`方法，给`obj`对象添加了`name`属性，值为`zhangsan`.

通过以上方式添加属性为响应式的。

下面我们修改一下`arr`数组中的第一项内容，看一下是否为响应式的。

如果是`arr[0]=100`这种修改方式，并不是响应式的，关于这一点我们在前面也讲解过。

我么可以使用`slice`函数来修改，或者可以使用`vm.$set`方法也是可以的。

```
vm.$set(vm.arr,0,100)
```

把`arr`数组中的下标为`0`的这一项的值修改为`100`/

以上代码实现的操作为响应式的。

关于`vm.$set`的使用也可以参考官方文档。

```
https://cn.vuejs.org/v2/api/#Vue-set
```

向响应式对象中添加一个 `property`，并确保这个新 `property `同样是响应式的，且触发视图更新。它必须用于向响应式对象上添加新 `property`，因为 `Vue`无法探测普通的新增` property` (比如 `this.myObject.newProperty = 'hi'`)

```
注意对象不能是 Vue 实例，或者 Vue 实例的根数据对象
```

以下代码是错误的。

```js
vm.$set(vm,'abc','a')
```

以上代码是想`vue`的实例添加属性`abc`,以上写法是错误的。

```js
vm.$set(vm.$data,'abc','12')
```

以上代码是向`data`中添加属性，以上代码也是错误的。表明不能向`Vue`实例的根数据对象中动态添加属性。

以上就是关于`set`方法的使用的方式。下一小节，我们来查看`set`的源代码。

