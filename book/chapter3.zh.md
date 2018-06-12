
## 第3章: 虚拟DOM

### 3.1虚拟DOM简介

虚拟DOM是DOM的抽象. 我们使用轻量级JavaScript对象来呈现真正的DOM节点. 每个组件的视图结构可以用虚拟DOM树表示. 当组件第一次渲染时,我们使用了虚拟DOM树`给予`功能. 然后将虚拟DOM树转换并插入到真实的DOM中. 当组件数据发生变化时,我们将重新渲染以获取新的虚拟DOM树,计算将旧的虚拟DOM树转换为新形状所需的最小差异 (插入,添加,删除,移动) 一. 最后,我们将这些更改应用到真正的DOM (最后两步在大多数虚拟DOM实现中称为修补) . Vuejs使用虚拟DOM而不是将DOM操作直接绑定到数据更改的原因是,我们可以通过切换虚拟DOM的后端来实现跨平台渲染. 

所以虚拟DOM实际上并不完全是DOM的抽象,它是组件视图结构的抽象. 我们可以使用各种后端来渲染虚拟DOM树,例如iOS和Android. 此外,虚拟DOM提供的抽象层将使声明式编程风格变得直截了当. 

我们在声明式数据驱动风格前端开发中有一个着名的方程: 

UI =渲染 (状态) 

`渲染函数通过使用虚拟DOM树应用状态来获取组件状态并生成DOM. 所以虚拟DOM是声明式UI编程的关键基础结构. ` 

3.2 Vue如何将模板转换为虚拟DOMVue关于渲染功能的官方文档

### 强烈推荐. 

[您](https://vuejs.org/v2/guide/render-function.html)应该阅读这个来理解Vue的模板实际上是一个在下面的语法糖. **下面的模板: **将编译成: 

\_C

    <div>
      I'm a template!
    </div> 

是别名

    function anonymous(
    ) {
      with(this){return _c('div',[_v("I'm a template!\n")])}
    }

`的createElement`. `该API创建一个虚拟DOM节点实例. `我们可以传递一组子节点给`的createElement`,所以渲染函数的结果将是虚拟DOM节点的树. 

所以Vue的模板在编译时被编译成渲染函数 (在vue-loader的帮助下) . 当Vue重新渲染UI时,渲染函数被调用. 并且它返回一个新的虚拟DOM树. 3.3虚拟DOM和组件系统

### 每个虚拟DOM节点是一个真正的DOM节点的抽象. 

但是组件怎么样?在Vuejs中,组件具有相应的虚拟DOM节点 (

VNODE`这个`VNODE实例被视为虚拟DOM树中组件的占位符. `这个占位符`VNODE实例只有一个子组件,即虚拟DOM节点对应的组件`根DOM`节点. **在这里应该是一个图像来形象化这个问题**3.4

_VNODE_

### 类`我们需要定义结构`VNODE

第一类. `SRC / VDOM / vnode.js`这里我们定义

_VNODE_

    export default function VNode(tag, data, children, text, elm, context, componentOptions) {
        this.tag = tag
        this.data = data
        this.children = children
        this.text = text
        this.elm = elm
        this.context = context
        this.key = data && data.key
        this.componentOptions = componentOptions
        this.componentInstance = undefined
        this.parent = undefined
        this.isComment = false
    }

使用构造函数的属性. `虚拟DOM节点具有一些与DOM相关的属性,如标签,文本和ns. `它也有Vue组件相关的信息,如componentOptions和componentInstance. 

children属性是一个指向节点子节点的指针,父属性指向该节点的父节点. 你知道,虚拟DOM是一棵树. 

一个最有用的属性`VNode`是数据属性. 它包含您在模板中定义的所有道具,指令,事件处理程序,类和样式. 

### 3.5`创建元素`API

我们要实施这个着名的`h`Vue的渲染功能!

> JSX使用`h`作为别名`的createElement`. Vue使用`_c`代替. 

我们来编写测试用例`的createElement`: 

_测试/虚拟域/创建-element.spec.js_

    import Vue from "src/index"
    import { createEmptyVNode } from 'src/vdom/vnode'

    describe('create-element', () => {
      it('render vnode with basic reserved tag using createElement', () => {
        const vm = new Vue({
          data: { msg: 'hello world' }
        })
        const h = vm.$createElement
        const vnode = h('p', {})
        expect(vnode.tag).toBe('p')
        expect(vnode.data).toEqual({})
        expect(vnode.children).toBeUndefined()
        expect(vnode.text).toBeUndefined()
        expect(vnode.elm).toBeUndefined()
        expect(vnode.ns).toBeUndefined()
        expect(vnode.context).toEqual(vm)
      })
      
      it('render vnode with component using createElement', () => {
        const vm = new Vue({
          data: { message: 'hello world' },
          components: {
            'my-component': {
              props: ['msg']
            }
          }
        })
        const h = vm.$createElement
        const vnode = h('my-component', { props: { msg: vm.message }})
        expect(vnode.tag).toMatch(/vue-component-[0-9]+/)
        expect(vnode.componentOptions.propsData).toEqual({ msg: vm.message })
        expect(vnode.children).toBeUndefined()
        expect(vnode.text).toBeUndefined()
        expect(vnode.elm).toBeUndefined()
        expect(vnode.ns).toBeUndefined()
        expect(vnode.context).toEqual(vm)
      })

      it('render vnode with custom tag using createElement', () => {
        const vm = new Vue({
          data: { msg: 'hello world' }
        })
        const h = vm.$createElement
        const tag = 'custom-tag'
        const vnode = h(tag, {})
        expect(vnode.tag).toBe('custom-tag')
        expect(vnode.data).toEqual({})
        expect(vnode.children).toBeUndefined()
        expect(vnode.text).toBeUndefined()
        expect(vnode.elm).toBeUndefined()
        expect(vnode.ns).toBeUndefined()
        expect(vnode.context).toEqual(vm)
        expect(vnode.componentOptions).toBeUndefined()
      })

      it('render empty vnode with falsy tag using createElement', () => {
        const vm = new Vue({
          data: { msg: 'hello world' }
        })
        const h = vm.$createElement
        const vnode = h(null, {})
        expect(vnode).toEqual(createEmptyVNode())
      })
    })

标签传递给`的createElement`应该是以下之一: 

-   平台内置元素的 (保留标签) 标签名称Vue组件的标签名称
-   自定义元素的 (Web组件) 标签名称
-   空值该
-   的createElement

函数应该处理这些情况. `对于平台内置的元素和自定义元素,我们只呈现原始标记. `对于Vue组件,我们创建一个Vue组件Node,这个API将在下一节中实现. 对于null,我们返回一个空的VNode. 实现非常简单: 3.6

创建组分

    import VNode, { createEmptyVNode } from "./vnode";
    import config from "../config";
    import { createComponent } from "./create-component";

    import {
      isDef,
      isUndef,
      isTrue,
      isPrimitive,
      resolveAsset
    } from "../util/index";

    export function createElement(
      context,
      tag,
      data,
      children
    ) {
      let vnode, ns;
      if (typeof tag === "string") {
        let Ctor;
        if (config.isReservedTag(tag)) {
          // platform built-in elements
          vnode = new VNode(
            config.parsePlatformTagName(tag),
            data,
            children,
            undefined,
            undefined,
            context
          );
        } else if (
          isDef((Ctor = resolveAsset(context.$options, "components", tag)))
        ) {
          // component
          vnode = createComponent(Ctor, data, context, children, tag);
        } else {
          // unknown or unlisted namespaced elements
          // check at runtime because it may get assigned a namespace when its
          // parent normalizes children
          vnode = new VNode(tag, data, children, undefined, undefined, context);
        }
      } else {
        // direct component options / constructor
        vnode = createComponent(tag, data, context, children);
      }
      if (!isDef(vnode)) {
        return createEmptyVNode();
      }
      return vnode;
    }

### API`预编写摘要: 创建组件实现了3.3节中介绍的想法. `也就是说,为Vue组件创建一个占位符VNode. 

> 3.7修补虚拟DOMPre-compose-summary: 补丁是VDOM模块的关键功能. 

### 介绍打补丁的功能和打补丁的基本流程. 

> 3.8钩子机制和补丁生命周期

### Pre-compose-summary: 介绍修补的生命周期: init,create,insert,prepatch,postpatch,update,remove,destroy. 

> 以及基于钩子的VDOM插件机制. 

### 3.9修补儿童

> 预撰写摘要: 魔法差异算法

### 3.10连接Vue和虚拟DOM

> 预撰写摘要: 当通知观察者时调用呈现

### 3.11预备
