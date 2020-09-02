---
title: Vue computed 实现原理
date: 2020-06-29 16:31:45
tags: vue
---

Q：Vue 的 computed 对象中的值，什么时候重算？缓存机制是如何实现的？

结论

首先 vue 初始化过程中，会执行 initComputed，遍历 computed 对象中的每个 key，对每个属性进行 new Watcher 操作，这里的 new Watcher 与 data 中的 new Watcher 传参不同，这里传入 { lazy: true } 参数，将 Watcher 内部的 dirty 设成 true，然后将 computed 的属性定义到 vm 上（也就是 this 上），computed 对应属性的值进行 getter 处理，获取对应的 watcher，判断 watcher 上 dirty 值是否为 true，如果为 true，执行 watcher 的 evaluate 重新计算，执行 evaluate 后会将 dirty 设成 false，后续读取 computed 中的值则不会重新计算。如果 computed 中的依赖项发生变化，其中的依赖项会触发 Object.definedProperty 的 setter，进而触发 notify， notify 触发 watcher 的 update，执行 update 后会将 dirty 设成 ture，因此，再次读取 computed 中属性值时会重新计算。

```javascript
/*---------------------------------------- Dep ------------------------------------*/

// 标识当前的 Dep id
let uidep = 0
// Dep 类 用于处理依赖相关
class Dep {
  constructor() {
    this.id = uidep++
    // 存放所有的监听 watcher
    this.subs = []
  }

  // 添加一个观察者对象
  addSub(Watcher) {
    this.subs.push(Watcher)
  }

  // 依赖收集
  depend() {
    // Dep.target 作用只有需要的才会收集依赖
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  // 调用依赖收集的 Watcher 更新
  notify() {
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// 当前的 target watcher 将被执行，
// 全局唯一且同一时间只能执行同一个 watcher
Dep.target = null
const targetStack = []

// 为 Dep.target 赋值
function pushTarget(Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = Watcher
}
function popTarget() {
  Dep.target = targetStack.pop()
}

/*--------------------------------------- Watcher -----------------------------------*/

// 去重 防止重复收集
let uid = 0
class Watcher {
  constructor(vm, expOrFn, cb, options) {
    // 传进来的 vue 实例，例如 Vue 组件
    this.vm = vm
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
    } else {
      this.deep = this.user = this.lazy = false
    }
    // dirty 后面会用于判断 computed 是否重算
    this.dirty = this.lazy
    // 在 Vue 中 cb 是更新视图的核心，调用 diff 并更新视图的过程
    this.cb = cb
    this.id = ++uid
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    if (typeof expOrFn === 'function') {
      // data 依赖收集走此处
      this.getter = expOrFn
    } else {
      // watch 依赖走此处
      this.getter = this.parsePath(expOrFn)
    }
    // 设置Dep.target 的值，依赖收集时的 watcher 对象
    this.value = this.lazy ? undefined : this.get()
  }

  get() {
    // 设置 Dep.target 值，用以依赖收集
    pushTarget(this)
    const vm = this.vm
    // 此处会进行依赖收集 会调用 data 数据的 get
    let value = this.getter.call(vm, vm)
    popTarget()
    return value
  }

  // 添加依赖
  addDep(dep) {
    // 去重
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        // 收集 watcher 每次 data 数据
        // set 时会遍历收集的 watcher 依赖进行相应视图更新或执行watch监听函数等操作
        dep.addSub(this)
      }
    }
  }

  // 更新
  update() {
    // 在 初始化 computed 时 lazy 必为 true，也就是 dirty 必为 true
    if (this.lazy) {
      this.dirty = true
    } else {
      this.run()
    }
  }

  // 更新视图
  run() {
    console.log(`这里会去执行Vue的diff相关方法，进而更新数据`)
    const value = this.get()
    const oldValue = this.value
    this.value = value
    if (this.user) {
      // watch 监听走此处
      this.cb.call(this.vm, value, oldValue)
    } else {
      // data 监听走此处
      // 这里只做简单的 console.log 处理，在 Vue 中会调用 diff 过程从而更新视图
      this.cb.call(this.vm, value, oldValue)
    }
  }

  // 如果计算熟悉依赖的 data 值发生变化时会调用
  // 案例中 当 data.name 值发生变化时会执行此方法
  evaluate() {
    this.value = this.get()
    this.dirty = false
  }
  //收集依赖
  depend() {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }

  // 此方法获得每个 watch 中 key 在 data 中对应的 value 值
  // 使用 split('.') 是为了得到 像 'a.b.c' 这样的监听值
  parsePath(path) {
    const bailRE = /[^w.$]/
    if (bailRE.test(path)) return
    const segments = path.split('.')
    return function (obj) {
      for (let i = 0; i < segments.length; i++) {
        if (!obj) return
        // 此处为了兼容我的代码做了一点修改
        // 此处使用新获得的值覆盖传入的值 因此能够处理 'a.b.c'这样的监听方式
        if (i === 0) {
          obj = obj.data[segments[i]]
        } else {
          obj = obj[segments[i]]
        }
      }
      return obj
    }
  }
}

/*--------------------------------------- Observer -----------------------------------*/
class Observer {
  constructor(value) {
    this.value = value
    // 增加 dep 属性（处理数组时可以直接调用）
    this.dep = new Dep()
    // 将 Observer 实例绑定到 data 的 __ob__ 属性上面去，后期如果 oberve 时直接使用，不需要重新 Observer,
    // 处理数组是也可直接获取Observer对象
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      // 这里只测试对象
    } else {
      // 处理对象
      this.walk(value)
    }
  }

  walk(obj) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      // 此处我做了拦截处理，防止死循环，Vue 中在 oberve 函数中进行的处理
      if (keys[i] == '__ob__') return
      defineReactive(obj, keys[i], obj[keys[i]])
    }
  }
}
// 数据重复 Observer
function observe(value) {
  if (typeof value != 'object') return
  let ob = new Observer(value)
  return ob
}
// 把对象属性改为 getter/setter，并收集依赖
function defineReactive(obj, key, val) {
  const dep = new Dep()
  // 处理 children
  let childOb = observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      console.log(`调用get获取值，值为${val}`)
      const value = val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
        }
      }
      return value
    },
    set: function reactiveSetter(newVal) {
      console.log(`调用了set，值为${newVal}`)
      const value = val
      val = newVal
      // 对新值进行observe
      childOb = observe(newVal)
      // 通知 dep 调用,循环调用手机的Watcher依赖，进行视图的更新
      dep.notify()
    },
  })
}
// 辅助方法
function def(obj, key, val) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: true,
    writable: true,
    configurable: true,
  })
}
/*----------------------------------------初始化watch------------------------------------*/
// 空函数
const noop = () => {}
// computed初始化的Watcher传入lazy: true就会触发Watcher中的dirty值为true
const computedWatcherOptions = { lazy: true }
//Object.defineProperty 默认value参数
const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop,
}
// 初始化computed
class initComputed {
  constructor(vm, computed) {
    //新建存储watcher对象，挂载在vm对象执行
    const watchers = (vm._computedWatchers = Object.create(null))
    //遍历computed
    for (const key in computed) {
      const userDef = computed[key]
      //getter值为computed中key的监听函数或对象的get值
      let getter = typeof userDef === 'function' ? userDef : userDef.get
      //新建computed的 watcher
      watchers[key] = new Watcher(vm, getter, noop, computedWatcherOptions)
      if (!(key in vm)) {
        /*定义计算属性*/
        this.defineComputed(vm, key, userDef)
      }
    }
  }
  // 把计算属性的key挂载到vm对象下，并使用Object.defineProperty进行处理
  // 因此调用vm.somecomputed 就会触发get函数
  defineComputed(target, key, userDef) {
    if (typeof userDef === 'function') {
      sharedPropertyDefinition.get = this.createComputedGetter(key)
      sharedPropertyDefinition.set = noop
    } else {
      sharedPropertyDefinition.get = userDef.get
        ? userDef.cache !== false
          ? this.createComputedGetter(key)
          : userDef.get
        : noop
      // 如果有设置set方法则直接使用，否则赋值空函数
      sharedPropertyDefinition.set = userDef.set ? userDef.set : noop
    }
    Object.defineProperty(target, key, sharedPropertyDefinition)
  }

  // 计算属性的 getter 获取计算属性的值时会调用
  createComputedGetter(key) {
    return function computedGetter() {
      // 获取到相应的 watcher
      const watcher = this._computedWatchers && this._computedWatchers[key]
      if (watcher) {
        // watcher.dirty 参数决定了计算属性值是否需要重新计算，默认值为 true，即第一次时会调用一次
        if (watcher.dirty) {
          // 每次执行之后 watcher.dirty 会设置为 false，只要依赖的 data 值改变时才会触发
          // watcher.dirty 为 true,从而获取值时从新计算
          watcher.evaluate()
        }
        // 获取依赖
        if (Dep.target) {
          watcher.depend()
        }
        // 返回计算属性的值
        return watcher.value
      }
    }
  }
}




// 1、首先来创建一个Vue构造函数：
function Vue() {}
// 2、设置data和computed的值：
let data = {
  name: 'Hello',
}
let computed = {
  getfullname: function () {
    console.log('-----走了computed 之 getfullname------')
    console.log('新的值为：' + data.name + ' - world')
    return data.name + ' - world'
  },
}
// 3、实例化Vue并把data挂载到Vue上
let vue = new Vue()
vue.data = data
// 4、创建Watcher对象
let updateComponent = (vm) => {
  // 收集依赖
  data.name
}
let watcher1 = new Watcher(vue, updateComponent, () => {})
// 5、初始化Data并收集依赖
observe(data)
// 6、初始化computed
let watcher2 = new initComputed(vue, computed)

```
