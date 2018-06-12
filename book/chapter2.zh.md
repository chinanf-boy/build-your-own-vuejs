
## 第2章: 反应性系统

Vue的反应系统使模型和视图之间的数据绑定变得简单而直观. 数据被定义为一个普通的JavaScript对象. 数据更改时,视图会自动更新以反映最新状态. 它像一个魅力. 

引擎盖下,Vuejs将遍历所有数据的属性并将它们转换为使用的getter / setters`Object.defineProperty`. 

数据中的每个原始键值对都有一个`观察`实例. 观察者将向早先订阅价值变化事件的观察者发送信号. 

每个`Vue公司`实例有一个`守望者`实例将组件呈现期间"触及"的任何属性记录为依赖关系. 当数据改变时,观察者将重新收集依赖关系并运行观察者初始化时传递的回调. 

那么观察者如何通知观察者进行数据更改?观察者模式来拯救!我们定义一个叫做的新类`德普`,意思是"依赖",作为调解人. 观察者实例对于数据更改时需要通知的所有分支都有一个参考. 每个dep实例都知道它需要更新哪个观察者. 

这基本上是从10万英尺的角度来看反应系统的工作原理. 在接下来的几节中,我们将仔细研究反应性系统的实现细节. 

### 2.1部门

实现`德普`是战略性的. 每个dep实例都有一个用于识别的uid. 该`潜艇`数组记录所有观察者订阅此dep实例. `Dep.prototype.notify`调用每个订户的更新方法`潜艇`阵列. `Dep.prototype.depend`用于观察者重新评估期间的依赖收集. 我们稍后会来看看. 现在你应该只知道这一点`Dep.target`是目前正在重新评估的观察者实例. 由于这个属性是静态的,所以`Dep.target`在全球范围内开展工作,并一次指向一名观察员. 

_SRC /观察者/ dep.js_

    var  uid = 0

    // Dep contructor
    export default function Dep(argument) {
      this.id = uid++
      this.subs = []
    }

    Dep.prototype.addSub = function(sub) {
      this.subs.push(sub)
    }

    Dep.prototype.removeSub = function(sub) {
      remove(this.subs, sub)
    }

    Dep.prototype.depend = function() {
      if (Dep.target) {
        Dep.target.addDep(this)
      }
    }

    Dep.prototype.notify = function() {
      var subs = this.subs.slice()
      for (var i = 0, l = subs.length; i < l; i++) {
        subs[i].update()
      }
    }

    Dep.target = null

### 2.2观察者基础

我们从这样的样板开始: 

_SRC /观察者/ index.js_

    // Observer constructor
    export function Observer(value) {

    }

    // API for observe value
    export function observe (value){

    }

在我们实施之前`观察`,我们会先写一个测试. 

_测试/观察者/ observer.spec.js_

    import {
      Observer,
      observe
    } from "../../src/observer/index"
    import Dep from '../../src/observer/dep'

    describe('Observer test', function() {
      it('observing object prop change', function() {
      	const obj = { a:1, b:{a:1}, c:NaN}
        observe(obj)
        // mock a watcher!
        const watcher = {
          deps: [],
          addDep (dep) {
            this.deps.push(dep)
            dep.addSub(this)
          },
          update: jasmine.createSpy()
        }
        // observing primitive value
        Dep.target = watcher
        obj.a
        Dep.target = null
        expect(watcher.deps.length).toBe(1) // obj.a
        obj.a = 3
        expect(watcher.update.calls.count()).toBe(1)
        watcher.deps = []
      });

    });

首先,我们定义一个普通的JavaScript对象`obj`作为数据. 然后我们使用`守`函数使数据产生反应. 由于我们还没有实现监视器,我们需要模拟观察者. 观察者有一个`deps`数组为依赖簿记. 该`更新`方法将在数据更改时调用. 欢迎来到`addDep`稍后部分. 

在这里,我们使用茉莉花的间谍功能作为占位符. 间谍功能没有真正的功能. 它保存了多少次被调用的信息以及被调用时传入的参数. 

然后我们设定全球`Dep.target`至`守望者`,并得到`obj.a.b`. 如果数据是反应性的,则将调用观察者的更新方法. 

所以让我们来看看`守`首先提醒. 代码如下所示. 它首先检查值是否是一个对象. 如果是这样,它会检查这个值是否已经有一个`观察`通过检查其实例来检查实例`__ob__`属性. 

如果没有存在的话`观察`例如,它会启动一个新的`观察`实例与值并返回它. 

_SRC /观察者/ index.js_

    import {
      hasOwn,
      isObject
    }
    from '../util/index'

    export function observe (value){
      if (!isObject(value)) {
        return
      }
      var ob
      if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
        ob = value.__ob__
      } else {
        ob = new Observer(value)
      }
      return ob
    }

在这里,我们需要一点实用功能`hasOwn`,这是一个简单的warpper`Object.prototype.hasOwnProperty`: 

_SRC / UTIL / index.js_

    var hasOwnProperty = Object.prototype.hasOwnProperty
    export function hasOwn (obj, key) {
      return hasOwnProperty.call(obj, key)
    }

另一个实用功能`则IsObject`: 

_SRC / UTIL / index.js_

    ···
    export function isObject (obj) {
      return obj !== null && typeof obj === 'object'
    }

现在是时候看看`观察`构造函数. 它会启动一个`德普`实例,它会调用`步行`与价值. 它附加观察员`值`如`__ob__`属性. 

_SRC /观察者/ index.js_

    import {
      def, //new
      hasOwn,
      isObject
    }
    from '../util/index'

    export function Observer(value) {
      this.value = value
      this.dep = new Dep()
      this.walk(value)
      def(value, '__ob__', this)
    }

`高清`这里是一个新的用于定义对象键使用属性的效用函数`Object.defineProperty () `API. 

_SRC / UTIL / index.js_

    ···
    export function def (obj, key, val, enumerable) {
      Object.defineProperty(obj, key, {
        value: val,
        enumerable: !!enumerable,
        writable: true,
        configurable: true
      })
    }

该`步行`方法只是遍历对象,用每个值调用`defineReactive`. 

_SRC /观察者/ index.js_

    Observer.prototype.walk = function(obj) {
      var keys = Object.keys(obj)
      for (var i = 0; i < keys.length; i++) {
          defineReactive(obj, keys[i], obj[keys[i]])
      }
    }

`defineReactive`在哪`Object.defineProperty`发挥作用. 

_SRC /观察者/ index.js_

    export function defineReactive (obj, key, val) {
      var dep = new Dep()
      Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: function reactiveGetter () {
          var value = val
          if (Dep.target) {
            dep.depend()
          }
          return value
        },
        set: function reactiveSetter (newVal) {
          var value =  val
          if (newVal === value || (newVal !== newVal && value !== value)) {
            return
          }
    	   val = newVal
          dep.notify()
        }
      })
    }

该`reactiveGetter`函数检查是否`Dep.target`存在,这意味着getter在观察者依赖性重新收集期间被触发. 发生这种情况时,我们通过调用来添加依赖关系`dep.depend () `. `dep.depend () `实际上呼叫`Dep.target.addDep (DEP) `. 以来`Dep.target`是一名观察员,是平等的`watcher.addDep (DEP) `. 让我们看看是什么`addDep`做: 

    addDep (dep) {
       this.deps.push(dep)
       dep.addSub(this)
    }

它推动`dep`观察者的`deps`阵列. 它也将目标观察者推到了dep处`潜艇`阵列. 这就是跟踪依赖的方式. 

该`reactiveSetter`函数只需设置新值,如果新值与旧值不一样. 它通知观察者通过呼叫进行更新`dep.notify () `. 我们来回顾一下Dep部分: 

    Dep.prototype.notify = function() {
      var subs = this.subs.slice()
      for (var i = 0, l = subs.length; i < l; i++) {
        subs[i].update()
      }
    }

`Dep.prototype.notify`调用每个观察者的`更新`方法`潜艇`阵列. 那么,是的,观察者是被推进的观察者`潜艇`数组期间`Dep.target.addDep (DEP) `. 所有事情都连接起来了. 

咱们试试吧`npm运行测试`. 我们写得更好的测试用例应该全部通过. 

### 2.3观察嵌套对象

我们现在只能观察具有原始值的简单对象. 因此,在本节中,我们将添加对观察非原始值 (如对象) 的支持. 

首先我们要修改测试案例: 

_测试/观察者/ observer.spec.js_

    describe('Observer test', function() {
      it('observing object prop change', function() {
    	···
        // observing non-primitive value
        Dep.target = watcher
        obj.b.a
        Dep.target = null
        expect(watcher.deps.length).toBe(3) // obj.b + b + b.a
        obj.b.a = 3
        expect(watcher.update.calls.count()).toBe(1)
        watcher.deps = []
      });

`obj.b`本身就是一个对象. 所以我们检查一下值是否改变`obj.b`被通知以查看是否支持非原始值观测. 

解决方案非常简单,我们将递归调用`观察者`功能`val`. 如果`val`不是一个对象,`观察者`将返回. 所以当我们使用`defineReactive`观察一个键值对,我们保持呼叫`守`保留并保留返回值`childOb`. 

_SRC /观察者/ index.js_

    export function defineReactive (obj, key, val) {
      var dep = new Dep()
      var childOb = observe(val) // new
      Object.defineProperty(obj, key, {
        ···
      })
    }

我们需要保留子观察者的参考的原因是我们需要在调用getter时重新收集子对象的依赖关系: 

_SRC /观察者/ index.js_

    ···
    get: function reactiveGetter () {
          var value = val
          if (Dep.target) {
            dep.depend()
            // re-collect for childOb
            if (childOb) {
              childOb.dep.depend()
            }
          }
          return value
        }
    ···

当调用setter时,我们还需要重新观察孩子的价值: 

_SRC /观察者/ index.js_

    ···
    set: function reactiveSetter (newVal) {
          var value =  val
          if (newVal === value || (newVal !== newVal && value !== value)) {
            return
          }
    	  val = newVal
          childOb = observe(newVal) //new
          dep.notify()
        }
    ···

### 2.4观察数据的设置/删除

Vue对观察数据变化有一些警告. Vue无法检测属性**加成**要么**缺失**由于Vue处理数据更改的方式. 只有在调用getter或setter时才会检测到数据更改,但设置/删除数据将不会调用getter或setter. 

但是,可以使用该方法将反应性属性添加到嵌套对象`Vue.set (对象,键,值) `方法. 使用. 删除反应性属性`Vue.delete (object,key,value) `方法. 

我们一如既往地为此写一个测试用例: 

_测试/观察者/ observer.spec.js_

    import {
      Observer,
      observe,
      set as setProp, //new
      del as delProp  //new
    }
    from "../../src/observer/index"
    import {
      hasOwn,
      isObject
    }
    from '../util/index' //new

    describe('Observer test', function() {
      // new test case
      it('observing set/delete', function() {
        const obj1 = {
          a: 1
        }
        // should notify set/delete data
        const ob1 = observe(obj1)
        const dep1 = ob1.dep
        spyOn(dep1, 'notify')
        setProp(obj1, 'b', 2)
        expect(obj1.b).toBe(2)
        expect(dep1.notify.calls.count()).toBe(1)
        delProp(obj1, 'a')
        expect(hasOwn(obj1, 'a')).toBe(false)
        expect(dep1.notify.calls.count()).toBe(2)
        // set existing key, should be a plain set and not
        // trigger own ob's notify
        setProp(obj1, 'b', 3)
        expect(obj1.b).toBe(3)
        expect(dep1.notify.calls.count()).toBe(2)
        // should ignore deleting non-existing key
        delProp(obj1, 'a')
        expect(dep1.notify.calls.count()).toBe(3)
      });
      ···
    }

我们添加一个新的测试用例`观察设置/删除`在`观察员测试`. 

现在我们可以实现这两种方法: 

_SRC /观察者/ index.js_

    export function set (obj, key, val) {
      if (hasOwn(obj, key)) {
        obj[key] = val
        return
      }
      const ob = obj.__ob__
      if (!ob) {
        obj[key] = val
        return
      }
      defineReactive(ob.value, key, val)
      ob.dep.notify()
      return val
    }

    export function del (obj, key) {
      const ob = obj.__ob__
      if (!hasOwn(obj, key)) {
        return
      }
      delete obj[key]
      if (!ob) {
        return
      }
      ob.dep.notify()
    }

功能`组`将首先检查密钥是否存在. 如果密钥存在,我们只需给它一个新的值并返回. 然后我们将检查这个对象是否被使用`OBJ .__ ob__`如果没有,我们会回来的. 如果密钥尚未存在,我们将使这个键值对无效使用`defineReactive`,并打电话`ob.dep.notify () `通知obj的值已更改. 

功能`德尔`几乎是相同的期望它删除价值使用`删除`运营商. 

### 2.5观测阵列

我们的实现还有一个缺陷,它不能观察阵列变异. 由于使用subscrpt语法访问数组元素不会触发getter. 所以老派的getter / setter不适合阵列变化检测. 

为了观察数组的变化,我们需要劫持一些数组方法`Array.prototype.pop () `和`Array.prototype.shift () `. 而不是使用subscrpt语法来设置数组值,我们将使用`Vue.set`在最后一个secion中实现的API. 

这里是我们使用时观察数组变异的测试用例`排列`将导致突变的API,将会观察到这种变化. 并且每个数组的元素也会被观察到. 

_测试/观察者/ observer.spec.js_

    describe('Observer test', function() {
    	// new
    	it('observing array mutation', () => {
        const arr = []
        const ob = observe(arr)
        const dep = ob.dep
        spyOn(dep, 'notify')
        const objs = [{}, {}, {}]
        arr.push(objs[0])
        arr.pop()
        arr.unshift(objs[1])
        arr.shift()
        arr.splice(0, 0, objs[2])
        arr.sort()
        arr.reverse()
        expect(dep.notify.calls.count()).toBe(7)
        // inserted elements should be observed
        objs.forEach(obj => {
          expect(obj.__ob__ instanceof Observer).toBe(true)
        })
      });
      ···
    }

第一步是处理数组`观察`: 

_SRC /观察者/ index.js_

    export function Observer(value) {
      this.value = value
      this.dep = new Dep()
      //this.walk(value) //deleted
      // new
      if(Array.isArray(value)){
        this.observeArray(value)
      }else{
        this.walk(value)
      }
      def(value, '__ob__', this)
    }

`observeArray`只是迭代数组并调用`守`在每个项目上. 

_SRC /观察者/ index.js_

    ···
    Observer.prototype.observeArray = function(items) {
      for (let i = 0, l = items.length; i < l; i++) {
        observe(items[i])
      }
    }

接下来我们要扭曲原文`排列`方法通过修改原型链. 

首先,我们创建一个拥有所有数组变体方法的单例. 这些数组方法由其他处理变化检测的逻辑进行处理. 

_SRC /观察者/ array.js_

    import { def } from '../util/index'

    const arrayProto = Array.prototype
    export const arrayMethods = Object.create(arrayProto)

    /**
     * Intercept mutating methods and emit events
     */
    ;[
      'push',
      'pop',
      'shift',
      'unshift',
      'splice',
      'sort',
      'reverse'
    ]
    .forEach(function (method) {
      // cache original method
      const original = arrayProto[method]
      def(arrayMethods, method, function mutator () {
        let i = arguments.length
        const args = new Array(i)
        while (i--) {
          args[i] = arguments[i]
        }
        const result = original.apply(this, args)
        const ob = this.__ob__
        let inserted
        switch (method) {
          case 'push':
            inserted = args
            break
          case 'unshift':
            inserted = args
            break
          case 'splice':
            inserted = args.slice(2)
            break
        }
        if (inserted) ob.observeArray(inserted)
        // notify change
        ob.dep.notify()
        return result
      })
    })

`arrayMethods`是具有全部数组变异方法的单例. 

对于数组中的所有方法: 

    ['push','pop','shift','unshift','splice','sort','reverse']

我们定义一个`突变`函数会扭曲原始方法. 

在里面`突变`函数,我们首先获取参数作为数组. 接下来,我们将原始数组方法应用于arguments数组并保留结果. 

对于向数组添加新项目的情况,我们称之为`observeArray`在新的数组项目上. 

最后,我们使用通知变更`ob.dep.notify () `,并返回结果. 

其次,我们需要将这个单例添加到原型链中. 

如果我们可以使用`__proto__`在当前的浏览器中,我们直接将数组的原型指向我们最近创建的单例. 

如果不是这样,我们会混合`arrayMethods`单身进入观察数组. 

所以我们需要一些辅助功能: 

_SRC /观察者/ index.js_

    // helpers
    /**
     * Augment an target Object or Array by intercepting
     * the prototype chain using __proto__
     */
    function protoAugment (target, src) {
      target.__proto__ = src
    }

    /**
     * Augment an target Object or Array by defining
     * properties.
     */
    function copyAugment (target, src, keys) {
      for (let i = 0, l = keys.length; i < l; i++) {
        var key = keys[i]
        def(target, key, src[key])
      }
    }

在`观察`功能,我们使用`protoAugment`要么`copyAugment`取决于我们是否可以使用`__proto__`或不,以增加原始数组: 

_SRC /观察者/ index.js_

    import {
      def,
      hasOwn,
      hasProto, //new
      isObject
    }
    from '../util/index'

    export function Observer(value) {
      this.value = value
      this.dep = new Dep()
      if(Array.isArray(value)){
        //new
        var augment = hasProto
            ? protoAugment
            : copyAugment
          augment(value, arrayMethods, arrayKeys)
        this.observeArray(value)
      }else{
        this.walk(value)
      }
      def(value, '__ob__', this)
    }

的定义`hasProto`是繁琐的: 

_SRC / UTIL / index.js_

    ···
    export var hasProto = '__proto__' in {}

这应该足以通过`观察阵列突变`测试. 

//关于dependsArray (value) 的一些信息

### 2.6看守

我们嘲笑了`守望者`在之前的测试中是这样的: 

    const watcher = {
    	deps: [],
    	addDep (dep) {
    		this.deps.push(dep)
    		dep.addSub(this)
        },
        update: jasmine.createSpy()
    }

所以这里的观察者基本上是一个有一个对象`deps`记录这个观察者的所有依赖关系的属性,它也有一个`addDep`添加依赖关系的方法,以及a`更新`当观看数据发生变化时将调用该方法. 

我们来看看Watcher构造函数签名: 

    constructor (
        vm: Component,
        expOrFn: string | Function,
        cb: Function,
        options?: Object
      )

所以看守者的构造函数需要一个`expOrFn`参数和回调`cb`. 该`expOrFn`是初始化观察者时评估的表达式或函数. 当观察者需要运行时调用回调. 

下面的测试应该阐明观察者的工作方式. 

_测试/观察者/ watcher.spec.js_

    import Vue from "../../src/instance/index";
    import Watcher from "../../src/observer/watcher";

    describe('Wathcer test', function() {
      it('should call callback when simple data change', function() {
      	var vm = new Vue({
      		data:{
      			a:2
      		}
      	})
      	var cb = jasmine.createSpy('callback');
      	var watcher = new Watcher(vm, function(){
      		var a = vm.a
      	}, cb)
      	vm.a = 5;
        expect(cb).toHaveBeenCalled();
      });
    });

该`expOrFn`被评估,以便调用vm的数据特定的无功吸气器 (在这种情况下,`vm.a`的吸气剂) . 观察者将自己设置为目前的目标. 所以`vm.a`的dep将推动这个观察者实例`潜艇`阵列. 观察者将推动`vm.a`这是对的`deps`阵列. 什么时候`vm.a`叫做二传手,`vm.a`的dep`潜艇`数组将被迭代并且每个观察者都进入`潜艇`数组`更新`方法将被调用. 最后,观察者的回调将被调用. 

现在我们可以开始实施Watcher类: 

**SRC /观察者/ watcher.js**

    let uid = 0

    export default function Watcher(vm, expOrFn, cb, options) {
      options = options ? options : {}
      this.vm = vm
      vm._watchers.push(this)
      this.cb = cb
      this.id = ++uid
    	
      // options
      this.deps = []
      this.newDeps = []
      this.depIds = new Set()
      this.newDepIds = new Set()
      this.getter = expOrFn
      this.value = this.get()
    }

该`守望者`类初始化一些属性. 每`守望者`实例具有唯一的ID以供进一步使用. 这是通过设置`this.id = ++ uid`. `this.deps`和`this.newDeps`是deps对象的数组,这些数组用于Deps簿记. 我们将看到为什么我们需要两个数组来实现. `this.depIds`和`this.newDepIds`是相应deps数组的id集. 我们可以通过这些集合来查找deps数组中是否存在特定的dep实例. 

最后两行评估传入的表达式/函数. 这一步是发生依赖关系收集的地方. 接下来我们需要实施Watcher.prototype.get`. `SRC /观察者/ watcher.js

**Watcher.prototype.get**

    Watcher.prototype.get = function() {
      pushTarget(this)
      var value = this.getter.call(this.vm, this.vm)
      popTarget()
      this.cleanupDeps()
      return value
    }

`方法先推送当前`守望者`实例为`Dep.target`. `然后获得通过的价值this.getter.call (this.vm,this.vm) `. `如果吸气剂是一种功能,该值并不重要. 之后,我们需要弹出目标,并清理. 

因为每次都需要清理干净守望者`实例被重新评估,dep-watcher映射的簿记是不同的. `当某些数据发生变化时,我们需要更新dep的子数组和观察者的数组. 

所以这就是为什么我们需要两个数组`守望者`构造函数. 该`newDep`数组和`newDepIds`数组用于新的依赖性集合运行. 最后一次的依赖被保存在`dep`和`depIds`阵列. 什么`cleanupDeps`确实只是移动数据中的数据`newDep`和`newDepIds`数组到`dep`和`depIds`数组,并重置`newDep`和`newDepIds`阵列. 

**SRC /观察者/ watcher.js**

    /**
     * Add a dependency to this directive.
     */
    Watcher.prototype.addDep = function(dep) {
      var id = dep.id
      if (!this.newDepIds.has(id)) {
        this.newDepIds.add(id)
        this.newDeps.push(dep)
        if (!this.depIds.has(id)) {
          dep.addSub(this)
        }
      }
    }

    /**
     * Clean up for dependency collection.
     */
    Watcher.prototype.cleanupDeps = function() {
      var i = this.deps.length
      while (i--) {
        var dep = this.deps[i]
        if (!this.newDepIds.has(dep.id)) {
          dep.removeSub(this)
        }
      }
      var tmp = this.depIds
      this.depIds = this.newDepIds
      this.newDepIds = tmp
      this.newDepIds.clear()
      tmp = this.deps
      this.deps = this.newDeps
      this.newDeps = []
    }

最后,`Watcher.prototype.update`和`Watcher.prototype.run`方法时使用`Wathcher`实例需要重新评估. `Watcher.prototype.update`只需打电话`Watcher.prototype.run` (这里的warpper用于进一步的asnyc批处理机制) . 

`Watcher.prototype.run`电话`this.get`获得新的价值,并调用回调`Wathcher`实例通知用户数据已更改. 

**SRC /观察者/ watcher.js**

    Watcher.prototype.update = function() {
      console.log("update!!")
      this.run()
    }

    Watcher.prototype.run = function() {
      var value = this.get()
      var oldValue = this.value
      this.value = value
      this.cb.call(this.vm, value, oldValue)
    }

### 2.7异步批处理队列

单元测试部分将使用Vue构造函数. 所以这部分应该移到后面的章节. 

简介: 为什么Async批处理队列?

单元测试

**SRC /观察者/ scheduler.js**

队列,flushQueue

```

```

下一个勾号

### 2.8预热

去做
