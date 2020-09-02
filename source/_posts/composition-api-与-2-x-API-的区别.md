---
title: composition-api 与 2.x API 的区别
date: 2020-07-24 18:22:46
tags: vue
---

最快的学习方法就是通过比较，跟自己熟悉的作比较，快速看出差异，能更好的了解 composition-api。

### 简单的计数器

标准 API
```vue
<template>
  <div>
    Count is {{ count }}, count * 2 is {{ double }}
    <button @click="increment">+</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  computed: {
    double() {
      return this.count * 2
    }
  }
}
</script>
```


函数 API
```vue
<template>
  <div>
    Count is {{ count }}, count * 2 is {{ double }}
    <button @click="increment">+</button>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const double = computed(() => count.value * 2)
    const increment = () => { count.value++ }
    return {
      count,
      double,
      increment
    }
  }
}
</script>
```


### 根据传入的值，请求数据


标准 API
```vue
<template>
  <div>
    <template v-if="isLoading">Loading...</template>
    <template v-else>
      <h3>{{ post.title }}</h3>
      <p>{{ post.body }}</p>
    </template>
  </div>
</template>

<script>
import { fetchPost } from './api'

export default {
  props: {
    id: Number
  },
  data() {
    return {
      isLoading: true,
      post: null
    }
  },
  mounted() {
    this.fetchPost()
  },
  watch: {
    id: 'fetchPost'
  },
  methods: {
    async fetchPost() {
      this.isLoading = true
      this.post = await fetchPost(this.id)
      this.isLoading = false
    }
  }
}
</script>
```


函数 API
```vue
<template>
  <div>
    <template v-if="isLoading">Loading...</template>
    <template v-else>
      <h3>{{ post.title }}</h3>
      <p>{{ post.body }}</p>
    </template>
  </div>
</template>

<script>
import { ref, watch } from 'vue'
import { fetchPost } from './api'

export default {
  setup(props) {
    const isLoading = ref(true)
    const post = ref(null)

    watch(() => props.id, async (id) => {
      isLoading.value = true
      post.value = await fetchPost(id)
      isLoading.value = false
    })

    return {
      isLoading,
      post
    }
  }
}
</script>
```


### 多个逻辑
基于前一个获取数据的例子，在此基础上新增展示鼠标的位置功能


标准 API
```vue
<template>
  <div>
    <template v-if="isLoading">Loading...</template>
    <template v-else>
      <h3>{{ post.title }}</h3>
      <p>{{ post.body }}</p>
    </template>
    <div>Mouse is at {{ x }}, {{ y }}</div>
  </div>
</template>

<script>
import { fetchPost } from './api'

export default {
  props: {
    id: Number
  },
  data() {
    return {
      isLoading: true,
      post: null,
      x: 0,
      y: 0
    }
  },
  mounted() {
    this.fetchPost()
    window.addEventListener('mousemove', this.updateMouse)
  },
  watch: {
    id: 'fetchPost'
  },
  destroyed() {
    window.removeEventListener('mousemove', this.updateMouse)
  },
  methods: {
    async fetchPost() {
      this.isLoading = true
      this.post = await fetchPost(this.id)
      this.isLoading = false
    },
    updateMouse(e) {
      this.x = e.pageX
      this.y = e.pageY
    }
  }
}
</script>
```


页面上只有两个逻辑，获取数据和展示鼠标位置，但是这两个逻辑相关的函数分散的在组件的选项中。


使用函数 API
```vue
<template>
  <div>
    <template v-if="isLoading">Loading...</template>
    <template v-else>
      <h3>{{ post.title }}</h3>
      <p>{{ post.body }}</p>
    </template>
    <div>Mouse is at {{ x }}, {{ y }}</div>
  </div>
</template>

<script>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import { fetchPost } from './api'

function useFetch(props) {
  const isLoading = ref(true)
  const post = ref(null)

  watch(() => props.id, async (id) => {
    isLoading.value = true
    post.value = await fetchPost(id)
    isLoading.value = false
  })

  return {
    isLoading,
    post
  }
}

function useMouse() {
  const x = value(0)
  const y = value(0)
  const update = e => {
    x.value = e.pageX
    y.value = e.pageY
  }
  onMounted(() => {
    window.addEventListener('mousemove', update)
  })
  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })
  return { x, y }
}

export default {
  setup(props) {
    return {
      ...useFetch(props),
      ...useMouse()
    }
  }
}
</script>
```


使用 函数 API，代码逻辑相对于选项来说组织的更清晰。


更多例子查看 [this gist](https://gist.github.com/yyx990803/762ec427882a61be3e4affe02f8af555).


#### 两种 API 混合使用
可以在 `setup` 中封装一个逻辑，然后在标准 API 中使用，下面例子中，鼠标相关的逻辑是通过函数 API 实现的，获取数据逻辑是通过标准 API 实现的。


_当 _`_setup_`  与 标准 API 一起使用时， `setup` 首先会先执行，如果与标准 API 中 data 里的属性有冲突，会覆盖。


```vue
<template>
  <div>
    <template v-if="isLoading">Loading...</template>
    <template v-else>
      <h3>{{ post.title }}</h3>
      <p>{{ post.body }}</p>
    </template>
    <div>Mouse is at {{ x }}, {{ y }}</div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'
import { fetchPost } from './api'

function useMouse() {
  const x = ref(0)
  const y = ref(0)
  const update = e => {
    x.value = e.pageX
    y.value = e.pageY
  }
  onMounted(() => {
    window.addEventListener('mousemove', update)
  })
  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })
  return { x, y }
}

export default {
  props: {
    id: Number
  },
  setup(props) {
    return {
      ...useMouse()
    }
  },
  data() {
    return {
      isLoading: true,
      post: null,
    }
  },
  mounted() {
    this.fetchPost()
  },
  watch: {
    id: 'fetchPost'
  },
  methods: {
    async fetchPost() {
      this.isLoading = true
      this.post = await fetchPost(this.id)
      this.isLoading = false
    }
  }
}
</script>
```


> vue rfcs: comparison with 2.x API
> [https://github.com/vuejs/rfcs/blob/function-apis/active-rfcs/0000-function-api.md#comparison-with-2x-api](https://github.com/vuejs/rfcs/blob/function-apis/active-rfcs/0000-function-api.md#comparison-with-2x-api)
