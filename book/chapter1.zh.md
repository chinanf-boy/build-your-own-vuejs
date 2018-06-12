
## 第1章: Vuejs概述

Vuejs是一个简单但功能强大的MVVM库. 它有助于我们为网络构建一个现代化的用户界面. 

在撰写本文时,Vuejs在Github上有36,312颗星. 每月下载230,250次. Vuejs 2.0为渲染层带来了一个轻量级的虚拟DOM实现. 这解开了更多的可能性,例如服务器端渲染和本地组件渲染. 

Vuejs声称是一个渐进的JavaScript框架. 虽然Vuejs的核心库很小. Vuejs有许多随附的工具和支持库. 因此,您可以使用Vuejs生态系统构建大型应用程序. 

### Vuejs内部组件

让我们了解一下Vuejs内部的核心组件. Vue内部属于仆役部分: 

![Vue internal](https://occc3ev3l.qnssl.com/Vue%20source%20overview.png)

#### 实例生命周期

一个新的Vue实例将经历几个阶段. 比如观察数据,初始化事件,编译模板和渲染. 你可以注册将在特定阶段被调用的生命周期钩子. 

#### 反应性系统

所谓的_反应性系统_是vue的数据视图绑定魔法来自哪里. 设置vue实例的数据时,视图会相应更新,反之亦然. 

Vue使用`Object.defineProperty`使数据对象的属性反应. 随着着名的_观察者模式_将数据更改和视图渲染链接在一起. 

#### 虚拟DOM

虚拟DOM是实际DOM树的树形表示,它以JavaScript对象的形式存在于内存中. 

当数据发生变化时,vue将渲染一个全新的虚拟DOM树,并保留旧的树. 虚拟DOM模块区分两棵树并将更改修补到实际的DOM树中. 

Vue使用[snabbdom](https://github.com/snabbdom/snabbdom)作为其虚拟DOM实现的基础. 并修改一下,使其与Vue的其他组件一起工作. 

#### 编译器

编译器的工作是将模板编译为渲染函数 (AST) . 它将Vue指令 (Vue指令只是普通的HTML属性) 和其他实体解析为一棵树. 它还检测最大静态子树 (没有动态绑定的子树) 并将它们从渲染中提取出来. Vue使用的HTML解析器最初是由[约翰Resig](http://ejohn.org). 

> 本书不会涵盖编译器的实现细节. 由于我们可以使用构建工具在构建时将vue模板编译为渲染函数,因此Compiler不是vue运行时的一部分. 而且我们甚至可以直接编写渲染函数,因此编译器不是了解vue内部的重要部分. 

### 建立开发环境

在我们开始构建自己的Vue.js之前,我们需要设置一些东西. 包括模块打包器和测试工具,因为我们将使用测试驱动的工作流程. 

由于这是一个JavaScript项目,而且我们使用一些奇特的工具,所以首先要做的就是运行`npm init`并设置了一些关于这个项目的信息. 

#### 为模块绑定设置汇总

我们将使用Rollup进行模块捆绑. [卷起](http://rollupjs.org)是一个JavaScript模块捆绑器. 它允许您将应用程序或库编写为一组模块 - 使用现代ES2015导入/导出语法. Vuejs也使用Rollup进行模块捆绑. 

我们必须为Rollup编写一个配置以使其工作. 在根目录下,触摸`rollup.conf.js`: 

    export default {
      input: 'src/instance/index.js',
      output: {
        name: 'Vue',
        file: 'dist/vue.js',
        format: 'iife'
      },
    };

别忘了跑步`npm安装汇总汇总表--save-dev`. 

#### 设置Karma和Jasmine进行测试

测试将需要相当多的包,运行: 

    npm install karma jasmine karma-jasmine karma-chrome-launcher
     karma-rollup-plugin karma-rollup-preprocessor buble  rollup-plugin-buble --save-dev

在根目录下,触摸`karma.conf.js`: 

    module.exports = function(config) {
      config.set({
        files: [{ pattern: 'test/**/*.spec.js', watched: false }],
        frameworks: ['jasmine'],
        browsers: ['Chrome'],
        preprocessors: {
          './test/**/*.js': ['rollup']
        },
        rollupPreprocessor: {
          plugins: [
            require('rollup-plugin-buble')(),
          ],
          output: {
            format: 'iife',
            name: 'Vue',
            sourcemap: 'inline'
          }
        }
      })
    }

#### 目录结构

    - package.json
    - rollup.conf.js
    - node_modules
    - dist
    - test
    - src
    	- observer
    	- instance
    	- util
    	- vdom

### 引导

为了方便,我们将添加一些npm脚本. 

_的package.json_

    "scripts": {
       "build": "rollup -c",
       "watch": "rollup -c -w",
       "test": "karma start"
    }

为了引导我们自己的Vuejs,让我们编写我们的第一个测试用例. 

_测试/选项/ options.spec.js_

    import Vue from "../src/instance/index";

    describe('Proxy test', function() {
      it('should proxy vm._data.a = vm.a', function() {
      	var vm = new Vue({
      		data:{
      			a:2
      		}
      	})
        expect(vm.a).toEqual(2);
      });
    });

这个测试用例测试道具是否与vm的数据类似`vm._data.a`代理虚拟机本身,就像`vm.a`. 这是Vue的小技巧之一. 

所以我们现在可以编写我们的第一行真实代码

_SRC /实例/ index.js_

    import { initMixin } from './init'

    function Vue (options) {
      this._init(options)
    }

    initMixin(Vue)

    export default Vue

这并不令人兴奋,只是Vue的构造函数调用`this._init`. 那么让我们来看看如何`initMixin`工作: 

_SRC /实例/ init.js_

    import { initState } from './state'

    export function initMixin (Vue) {
      Vue.prototype._init = function (options) {
      	var vm = this
      	vm.$options = options
      	initState(vm)
      }
    }

Vue类的实例方法是使用mixin模式注入的. 稍后编写Vuejs的实例方法时,我们会发现这种mixin模式很常见. Mixin只是一个接受构造函数的函数,在其原型中添加一些方法并返回构造函数. 

所以`initMixin`加`_在里面`方法`Vue.prototype`. 这个方法调用`initState`从`state.js`: 

_SRC /实例/ state.js_

    export function initState(vm) {
      initData(vm)
    }

    function initData(vm) {
      var data = vm.$options.data
      vm._data = data
      // proxy data on instance
      var keys = Object.keys(data)

      var i = keys.length
      while (i--) {
        proxy(vm, keys[i])
      }
    }

    function proxy(vm, key) {
        Object.defineProperty(vm, key, {
          configurable: true,
          enumerable: true,
          get: function proxyGetter() {
            return vm._data[key]
          },
          set: function proxySetter(val) {
            vm._data[key] = val
          }
        })
    }

最后,我们到了代理发生的地方. `initState`电话`initData`,和`initData`迭代所有的键`vm._data`,电话`代理`对每个值. 

`代理`定义一个属性`vm`使用相同的密钥,并且此属性具有getter和setter,它们实际上从中获取/设置数据`vm._data`. 

那就是这样`vm.a`代理到`vm._data.a`. 

跑`npm运行构建`和`npm运行测试`. 你应该看到这样的东西: 

![success](http://cdn4.snapgram.co/images/2016/12/11/ScreenShot2016-12-12at2.02.17AM.png)

好样的!你成功引导你自己的Vuejs!继续工作!
