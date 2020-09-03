---
title: 面向未来编程，如何在 Vue2 中使用 Vue3 的语法[实践篇]
date: 2019-07-10 13:34:07
tags: vue
---
面向未来编程(Future-Oriented Programming)，[vue-function-api](https://github.com/vuejs/vue-function-api) 提供开发者在 Vue2.x 中使用 Vue3 的语法逻辑开发应用。(为方便以下以 Vue2 表示)

本文不对文档 api 对过多说明，仅讨论在项目实践中遇到的问题。比较两者的区别是对 Vue3 写法最快的了解，下面通过对比同一个功能在 Vue2 与 Vue3 的区别。

场景是这样的，一个列表页，上部分是条件筛选，下部分是 table，table 行内会有删除，修改（以 dialog 形式展示）功能，template 部分无区别，这里不做赘述。

### 准备工作

安装 vue-function-api

```shell
npm install vue-function-api --save

# 或者
yarn add vue-function-api
```

在项目中引入 main.js

```javascript
import Vue from 'vue'
import Element from 'element-ui'
import { plugin } from 'vue-function-api' // <--- 引入 vue-function-api

import App from './App'
import router from './router'
import store from './store'

Vue.use(Element)
Vue.use(plugin) // <--- 这样就可以在单文件组件中使用函数式 API了

Vue.config.productionTip = false

new Vue({
  router,
  store,
  render: h => h(App),
}).$mount('#app')
```

准备工作后，就可以安安静静的做比较了。

### data
在 Vue2 中，首先要定义 **data**, **data** 中我们定义一个 query 对象用于条件筛选，citys 用于条件筛选中城市（动态获取下拉），statusMap 用于筛选条件中状态的下拉，list 用于 table。

```javascript
export default {
  data() {
    return {
      query: {
        city: null,
        name: null,
        status: null
      },
      citys: [],
      list: [],
      statusMap:  [{
        value: 1,
        label: '启用'
      }, {
        value: 2,
        label: '禁用'
      }]
    }
  }
```

Vue3 中是这样定义

```javascript
import { value } from 'vue-function-api'

export default {
  setup() {
    const query = value({
      city: null,
      name: null,
      status: null
    })
    const citys = value([])
    const list = value([])
    const statusMap = value([{
      value: 1,
      label: '启用'
    }, {
      value: 2,
      label: '禁用'
    }])

    return {
      query,
      citys,
      list,
      statusMap
    }
  }
}
```
引入 **setup**, 在其中做 **data** 做的事情，细心的同学会发现在 **setup** 后需要 return 所定义的变量，其中也并非要将所有的变量都放入 return 中，放入 return 中是为了能够在 template 中使用，即只需 return template 中依赖的变量

### computed
上文中说到 table 中有删除，编辑功能，正常的系统中这类功能是需要有权限的人才能操作，这里我们假设已经将 permissions 存入 vuex 中，使用的时需要通过 mapGetters 取出 permissions（此处为举例这样实现，正常的权限要比这个复杂）。

在 Vue2 中是这样写

```javascript
import { mapGetters } from 'vuex'

export default {
  // 此处先省略 data 部分...
  computed: {
    ...mapGetters(['permissions']),
    canUpdate() {
      return this.permissions.includes('update')
    },
    canDelete() {
      return this.permissions.includes('delete')
    },
  }
  // ...
}

```

Vue3：目前 vue-function-api 还不支持 vuex map 的方式 导出 vuex 中的状态，这里我们从 Vue root 中取出 store 中的数据

```javascript
import { computed } from 'vue-function-api'

export default {
  setup(props, ctx) {
    const { $store } = ctx.root
    // 此处先省略 data 部分...

    const permissions = computed(() => $store.getters.permissions)
    const canUpdate = () => permissions.includes('update')
    const canDelete = () => permissions.includes('delete')

    return {
      canUpdate,
      canDelete // <--- 这里只导出 template 的依赖
    }
  }
}
```
在 template 中， 编辑，删除的按钮上 分别加入 `v-if="canUpdate"`, `v-if="canDelete"`，即可实现对应权限的人才能看到对应的按钮，了解更多 [context](https://github.com/vuejs/vue-function-api#context)。

### methods & lifecycle

基于上文中的描述，因为是一个列表页，所以结合 lifecycle 一起做一下对比。

在 Vue2 中
```javascript
export default {
  // ...

  methods: {
    async fetchCity() {
      const response = await fetchCityApi()
      this.citys = response.data
    },

    async fetchList() {
      const response = await fetchListApi(this.query)
      this.list = response.data
    },

    async delete(id) {
      const response = await deleteItem(id)
      const { status, msg } = response.data
      if (status !== 'ok') return this.$message.error(msg)
      this.$message({
        type: 'success',
        message: '删除成功'
      })
    },

    confirm(id) {
      this.$confirm(`确认删除`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.delete(id)
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消'
        })
      })
    },

    detail(id) {
      this.$router.push({
        path: '/detail',
        query: { id }
      })
    }
  }
}

  created() {
    this.fetchCity()
    this.fetchList()
  }
}
```

接下来看 Vue3 中实现

```javascript
import { computed, onCreated } from 'vue-function-api'

export default {
  setup(props, ctx) {
    const { $message, $router, $confirm, $store } = ctx.root // <--- setup() 中不可以使用 this 访问当前组件实例, 可以通过 setup 的第二个参数 context 来访问 vue2.x API 中实例上的属性。
    // 此处先省略 data 部分...

    // method
    const fetchCity = async () => {
      const response = await fetchCityApi()
      citys.value = response.data // <--- 划重点，通过 value() wrap初始化的变量在 setup 内部引用值或者修改值都要加 .value
    },

    const fetchList = async () => {
      const response = await fetchListApi(query.value)
      list.value = response.data
    },

    const delete = async id => {
      const response = await deleteItem(id)
      const { status, msg } = response.data
      if (status !== 'ok') return $message.error(msg)
      $message({
        type: 'success',
        message: '删除成功'
      })
    },

    confirm(id) {
      $confirm(`确认删除`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        delete(id)
      }).catch(() => {
        $message({
          type: 'info',
          message: '已取消'
        })
      })
    },

    detail(id) {
      $router.push({
        path: '/detail',
        query: { id }
      })
    }

    // lifecycle
    onCreated(() => {
      fetchCity()
      fetchList()
    })

    return {
      fetchCity,
      fetchList,
      confirm,
      detail // <--- 这里只导出 template 的依赖
    }
  }
}
```
是不是感觉切换到 Vue3 也没那么难

### watch

watch 的例子在一个列表页中可能没有什么使用场景，上文中说到列表页上面是一个条件筛选，有城市下拉，状态下拉，当然正常情况下，我们都是通过 select 的 change 事件调用 fetchList，这里我就为了举例而这么写，实际项目自己衡量利弊。

在 Vue2 中
```javascript
export default {
  // ...

  watch: {
    query: {
      handler: function(val) {
        this.fetchList()
      },
      { deep: true }
    }
  }

  // 或者不嫌麻烦这样写...
  watch: {
    'query.city': function(val) {
      this.fetchList()
    },
    'query.name': function(val) {
      this.fetchList()
    },
    'query.status': function(val) {
      this.fetchList()
    }
  }

  // ...
}
```

在 Vue3 中
```javascript
import { watch } from 'vue-function-api'

export default {
  setup() {
    // ...

    watch(
      query,
      val => {
        fetchList()
      },
      { deep: true }
    )

    // 或者

    watch(
      () => query.value,
      val => {
        fetchList()
      },
      { deep: true }
    )

    // ...
  }
}
```

通过以上对比，你应该对 Vue3 写法有一定理解了，因为是以一个列表页为 demo对比，没有用到 **provide**、 **inject**，想了解的同学可以到 _[`provide, inject`](https://github.com/vuejs/vue-function-api#provide-inject)，_ 还有 **state** 相当于 _[`Vue.observable`](https://vuejs.org/v2/api/index.html#Vue-observable)_.

对比完整代码

**Vue2**

```javascript
export default {
  data() {
    return {
      query: {
        city: null,
        name: null,
        status: null
      },
      citys: [],
      list: [],
      statusMap:  [{
        value: 1,
        label: '启用'
      }, {
        value: 2,
        label: '禁用'
      }]
    }
  },
  computed: {
    ...mapGetters(['permissions']),
    canUpdate() {
      return this.permissions.includes('update')
    },
    canDelete() {
      return this.permissions.includes('delete')
    },
  },
  methods: {
    async fetchCity() {
      const response = await fetchCityApi()
      this.citys = response.data
    },

    async fetchList() {
      const response = await fetchListApi(this.query)
      this.list = response.data
    },

    async delete(id) {
      const response = await deleteItem(id)
      const { status, msg } = response.data
      if (status !== 'ok') return this.$message.error(msg)
      this.$message({
        type: 'success',
        message: '删除成功'
      })
    },

    confirm(id) {
      this.$confirm(`确认删除`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.delete(id)
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消'
        })
      })
    },

    detail(id) {
      this.$router.push({
        path: '/detail',
        query: { id }
      })
    }
  },
  created() {
    this.fetchCity()
    this.fetchList()
  },
  watch: {
    query: {
      handler: function(val) {
        this.fetchList()
      },
      { deep: true }
    }
  }

```

**Vue3**

```javascript
import { value, computed, onCreated } from 'vue-function-api'

export default {
  setup(props, ctx) {
    const { $store, $message, $router, $route } = ctx.root
    // reactive state
    const query = value({
      city: null,
      name: null,
      status: null
    })
    const citys = value([])
    const list = value([])
    const statusMap = value([{
      value: 1,
      label: '启用'
    }, {
      value: 2,
      label: '禁用'
    }])

    // computed
    const permissions = computed(() => $store.getters.permissions)
    const canUpdate = () => permissions.includes('update')
    const canDelete = () => permissions.includes('delete')

    // method
    const fetchCity = async () => {
      const response = await fetchCityApi()
      citys.value = response.data
    },

    const fetchList = async () => {
      const response = await fetchListApi(query.value)
      list.value = response.data
    },

    const delete = async id => {
      const response = await deleteItem(id)
      const { status, msg } = response.data
      if (status !== 'ok') return $message.error(msg)
      $message({
        type: 'success',
        message: '删除成功'
      })
    },

    confirm(id) {
      $confirm(`确认删除`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        delete(id)
      }).catch(() => {
        $message({
          type: 'info',
          message: '已取消'
        })
      })
    },

    detail(id) {
      $router.push({
        path: '/detail',
        query: { id }
      })
    }

    // watch
    watch(
      query,
      val => {
        fetchList()
      },
      { deep: true }
    )

    // lifecycle
    onCreated(() => {
      fetchCity()
      fetchList()
    })

    return {
      query,
      citys,
      list,
      statusMap,
      canUpdate,
      canDelete,
      fetchCity,
      fetchList,
      confirm,
      detail
    }
  }
}
```

### 最后

这里贴一下 **setup** 中的第二个参数 `context`，对象中的属性是 2.x 中的 vue 实例属性的一个子集。完整的属性列表：

- parent
- root
- refs
- slots
- attrs
- emit

像其他库 vuex，vue-router，ElementUI 的一些 $方法可以从 root 中取得

```javascript
export default {
  setup(props, ctx) {
    const { $store, $router, $route, $message, $confirm } = ctx.root
  }
}
```

### 最后的最后
笔者已用到正式环境，目前来看还没有什么问题，引用 vue-function-api 官方的说明
> vue-function-api 会一直保持与 Vue3.x 的兼容性，当 3.0 发布时，您可以无缝替换掉本库。
> vue-function-api 的实现只依赖 Vue2.x 本身，不论 Vue3.x 的发布与否，都不会影响您正常使用本库。
> 由于 Vue2.x 的公共 API 限制，vue-function-api 无法避免的会产生一些额外的内存负载。如果您的应用并不工作在极端内存环境下，无需关心此项。

[Vue Function API](https://github.com/vuejs/vue-function-api)
[通过基于函数的 API 来复用组件逻辑](https://zhuanlan.zhihu.com/p/68477600)