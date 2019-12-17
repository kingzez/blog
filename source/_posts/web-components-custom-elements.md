---
title: 初探 web components
date: 2019-12-13 0:34:07
tags: [webcomponents]
---
通过使用 Custom Elements 创建内联的 CSS 和 JavaScript 的自定义元素。需要说明的是它不是 React， Vue 或者 Angular 的框架的替代方案，它是一个全新的概念。

#### CustomElementRegistry 对象
在 window 全局对象下暴露了 customElements 属性，可以通过此属性访问到 `CustomElementRegistry` 对象。
`CustomElementRegistry` 对象下的几种方法用于注册 Custom Elements和查询 Custom Elements：

- `define()`用于定义新的 Custom Element
- `get()` 用于获取 Custom Element 的 constructor，如果不存在，返回 undefined
- `upgrade()` 用于升级 Custom Element
- `whenDefined()` 用于获取 Custom Element 的 constructor，类似 get()，不同之处是返回值是 promise，有可用值是返回 resolves 状态

#### 如何创建一个 Custom Element
在调用 `window.customElements.define()` 方法前，首先要定义一个新的HTML元素

```javascript
class CustomTitle extends HTMLElement {
    // ToDo...
}
```
在 CustomTitle 类的构造器中使用 **Shadow DOM** 关联自定义 CSS，JavaScript 和 HTML到新的元素，这样就可以在这个新元素中封装各种功能，首先初始化构造器

```javascript
class CustomTitle extends HTMLElement {
    constructor() {
        super()
        //...
    }
}
```


然后调用 attachShadow()，传入参数 `{ mode: 'open' }` ,这个属性设置了 shadow DOM的封装模式，如果设置值为 `open`，可以访问元素的 shadowRoot 属性，如果值为 `close`，则不可以。

```javascript
class CustomTitle extends HTMLElement {
    constructor() {
        super()
        this.attachShadow({ mode: 'open' })
        //...
    }
}
```

接下来设置元素的HTML

```javascript
class CustomTitle extends HTMLElement {
    constructor() {
        super()
        this.attachShadow({ mode: 'open' })
        this.shadowRoot.innerHTML = `
            <h1>Hello WC!</h1>
        `
    }
}
```

> 这里的 innerHTML 可以写入多个 html tag

定义了内置元素后，即可使用 window.customElements

```javascript
window.customElements.define('custom-title', CustomTitle)
```

以上即可在页面中使用 <custom-title></custom-title>

#### 加入CSS

```javascript
class CustomTitle extends HTMLElement {
  constructor() {
    super()
    this.attachShadow({ mode: 'open' })
    this.shadowRoot.innerHTML = `
      <style>
        h1 {
          font-size: 40px;
          color: #000;
        }
      </style>
      <h1>Hello WC!</h1>
    `
  }
}
```

#### 简写语法

```javascript
window.customElements.define('custom-title', class extends HTMLElement {
  constructor() {
    ...
  }
})
```

#### 加入 JavaScript
在加入JavaScript处理方式上与 CSS 上略有区别，不能再模板字符串中直接写入，需要加入 addEventListener处理

```javascript
class CustomTitle extends HTMLElement {
    constructor() {
        super()
        this.attachShadow({ mode: 'open' })
        this.shadowRoot.innerHTML = `
        <style>
            h1 {
              font-size: 40px;
              color: #000;
            }
        </style>
        <h1>Hello WC!</h1>
        `
    this.addEventListener('click', e => {
    	console.log('clicked')
    })
  }
}
```

[demo 链接](https://codepen.io/kingzez/pen/NWPbOrO)

#### 使用 template
可以使用 `template`，通过id指定到

```html
<template id="custom-title-template">
  <style>
    h1 {
        font-size: 40px;
        color: #000;
    }
  </style>
  <h1>Hello WC!</h1>
</template>

<custom-title></custom-title>
```

然后在 Custom Element 构造器中引用到 shadow DOM 上

```javascript
class CustomTitle extends HTMLElement {
    constructor() {
        super()
        this.attachShadow({ mode: 'open' })
        const tmpl = ducument.querySelector('#custom-title-template')
        this.shadowRoot.appendChild(tmpl.content.cloneNode(true))
  }
}

window.customElements.define('custom-title', CustomTitle)
```

[demo 链接](https://codepen.io/kingzez/pen/xxbRyYg)

#### 生命周期
除 constructor 外，还可以定义生命周期函数，用于在特定时期执行特定的方法

- `connectedCallback` 当元素插入到DOM中时执行
- `disconnectedCallback` 当DOM中删除元素时执行
- `attributeChangedCallback` 当检测到属性发生更改或添加或删除时执行（传入3个参数）
  - attrName: 变化了的属性名
  - oldVal: 属性改变前的值
  - newVal: 属性改变后的值
- `adoptedCallback` 当元素已[移至新文档时](https://developer.mozilla.org/en-US/docs/Web/API/Document/adoptNode)

```javascript
class CustomTitle extends HTMLElement {
    constructor() {
      //...
    }

    connectedCallback() {
      //...
    }

    disconnectedCallback() {
      //...
    }

    attributeChangedCallback(attrName, oldVal, newVal) {
      //...
    }
}
```

上面提到`attributeChangedCallback` 会监听属性值的变化，这里要定义监听属性值的范围，必须定义一个静态方法用于返回监听属性的数组

```javascript
class CustomTitle extends HTMLELement {
    constructor() {
      //...
    }

    static get observedAttributes() {
      return ['diabled']
    }

    attributeChangedCallback() {
      //...
    }
}
```

这里我定义了 `disabled` 属性用于被监听，当它的值变化了，比如它的值为 `true`

```javascript
document.querySelector('custom-title').disabled = true
```

此时，`attributeChangedCallback` 被触发，三个参数值分别为 `'disabled', false, true`

#### 定义自定义属性
在自定义元素上定义自定义属性，通过添加 getter 和 setter 函数向自定义元素上定义自定义属性

```javascript
class CustomTitle extends HTMLElement {
    static get observedAttributes() {
        return ['nbattribute']
    }

    get nbattribute() {
        return this.getAttribute('nbattribute')
    }

    set nbattribute(value) {
        this.setAttribute('nbattribute', value)
    }
}
```

如果是定义类似 `disabled` 这样的布尔属性值，如果值为 `true` 时展示这个属性值，`false` 时不展示

```javascript
class CustomTitle extends HTMLElement {
    static get observedAttributes() {
        return ['boolattribute']
    }

    get boolattribute() {
        return this.hasAttribute('boolattribute')
    }

    set boolattribute(value) {
        if (value) {
            this.setAttribute('boolattribute', '')
        } else {
            this.removeAttribute('boolattribute')
        }
    }
}
```

#### 设置尚未定义的自定义元素的样式
在页面加载过程中，JavaScript的加载需要时间，此时自定义元素还未定义，当页面加载完成后，页面的的重新布局过程可能不是那么友好，为了解决这个问题，加入 `:not(:defined)` 伪类，设置一个大致的高度和渐变效果

```css
custom-title:not(:defined) {
    display: block;
    height: 400px;
    opacity: 0;
    transition: opacity 0.5s ease-in-out;
}
```

#### 浏览器支持情况 [caniuse](https://caniuse.com/#search=Custom%20Elements)
最新版的 FireFox，Safari，Chrome，使用 Chromium内核 重写的Edge，IE（就别想了），另外可以加入 [polyfill](https://github.com/webcomponents/polyfills/tree/master/packages/custom-elements) 兼容老版本的浏览器

#### Web Components 相关的库
- [Hybrids](https://github.com/hybridsjs/hybrids) is a UI library for creating Web Components with simple and functional API.
- [LitElement](https://github.com/Polymer/lit-element) uses [lit-html](https://github.com/Polymer/lit-html) to render into the element's Shadow DOM and adds API to help manage element properties and attributes.
Polymer provides a set of features for creating custom elements.
- [Slim.js](http://slimjs.com/) is an opensource lightweight web component library that provides data-binding and extended capabilities for components, using es6 native class inheritance.
- [Stencil](https://stenciljs.com/) is an opensource compiler that generates standards-compliant web components.