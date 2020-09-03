---
title: Fetch 实现 abort
date: 2020-04-05 01:02:57
tags: fetch
---

![cover](https://user-images.githubusercontent.com/10891613/92090784-721bef00-ee02-11ea-8cb7-e057c081451c.png)

中断一个 fetch 请求，主要使用 `AbortController` 实现。


目前，浏览器端原生获取数据，发请求的主要是 `XMLHttpRequest` 和 `fetch`，`XMLHttpRequest` 作为元老级的存在， `fetch` 是在 ES6 中推出的更现代的 API。


`XMLHttpRequest` 本身是支持请求中断的（abort），以下是 XHR 中断的示例


```javascript
const xhr = new XMLHttpRequest()
xhr.method = 'GET'
xhr.url = 'https://slowmo.glitch.me/5000'
xhr.open(method, url, true)
xhr.send()

// 中断请求
abortButton.addEventListener('click', () => xhr.abort())
```


`fetch` 设计之初是不支持中断的，开发者 2015 年在 GitHub 上提的 [issue](https://github.com/whatwg/fetch/issues/27) 还是 open 状态，以至于开发者们尝试在 `fetch` 规范外兼容或解决这个问题，其中就包括 [cancelable-promises](https://github.com/tc39/proposal-cancelable-promises) 及其他 [hack](https://github.com/whatwg/fetch/issues/27#issuecomment-267483591) 方法。


但是，现在有了通用的 [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) 和 `AbortSignal` API，这些 API 是由 [DOM 标准规范](https://dom.spec.whatwg.org/#aborting-ongoing-activities) 提供的，而不是由语言本身提供的。


## 什么是 AbortController？


![image.png](https://cdn.nlark.com/yuque/0/2020/png/168491/1586014073330-3e5c59ae-6258-40ff-be63-13f3377fde51.png#align=left&display=inline&height=311&name=image.png&originHeight=504&originWidth=978&size=75306&status=done&style=none&width=603)


正如 [DOM 规范文档中所说](https://dom.spec.whatwg.org/#aborting-ongoing-activities)


> Though promises do not have a built-in aborting mechanism, many APIs using them require abort semantics. AbortController is meant to support these requirements by providing an abort() method that toggles the state of a corresponding AbortSignal object. The API which wishes to support aborting can accept an AbortSignal object, and use its state to determine how to proceed.
>

> 尽管 Promise 没有内置的中止机制，但是使用它们的许多 API 都需要中止语义。 AbortController 旨在通过提供可以切换相应 AbortSignal 对象状态的 abort() 方法来支持这些需求。 希望支持中止的 API 可以接受 AbortSignal 对象，并使用其状态来确定如何进行。



以下是 `AbortController`  的基本用法
```javascript
// 创建 AbortController 的实例
const controller = new AbortController()
const signal = controller.signal

// 监听 abort 事件，在 controller.abort() 执行后执行回调打印
signal.addEventListener('abort', () => {
	console.log(signal.aborted) // true
})

// 触发中断
controller.abort()
```


## 怎么使用 AbortController 中断 fetch 请求？


`fetch` 接受 `AbortSignal` 作为第二个参数的一部分。


```javascript
const controller = new AbortController()
const signal = controller.signal

// API 5s 后返回相应
// https://slowmo.glitch.me/5000 5000 代表 5s 后返回相应值

fetch('https://slowmo.glitch.me/5000', { signal })
	.then(r => r.json())
	.then(response => console.log(response))
	.catch(err => {
		if (err.name === 'AbortError') {
    	console.log('Fetch was aborted')
    } else {
    	console.log('Error', err)
    }
	})

// 在 2s 后中断请求，将触发 'AbortError'
setTimeout(() => controller.abort(), 2000)
```


触发 `controller.abort()`  会中断 `fetch`  的 request 和 response。相同的 `AbortSignal` （以上示例中的 signal ）可用于中止多个获取请求。 `AbortController`  不仅适用于 `fetch` ，可以适用到中止任意异步事件，比如可以[实现一个可中断的promise](https://egghead.io/lessons/react-cancel-a-promise-using-abortcontroller)。


## 浏览器支持情况
除了 IE 以外，支持性还是可以的
[https://caniuse.com/#feat=abortcontroller](https://caniuse.com/#feat=abortcontroller)
[https://developer.mozilla.org/zh-CN/docs/Web/API/FetchController#Browser_compatibility](https://developer.mozilla.org/zh-CN/docs/Web/API/FetchController#Browser_compatibility)

## polyfill
[https://www.npmjs.com/package/abort-controller](https://www.npmjs.com/package/abort-controller)
[https://www.npmjs.com/package/abortcontroller-polyfill](https://www.npmjs.com/package/abortcontroller-polyfill)


## 参考:
> [https://developers.google.com/web/updates/2017/09/abortable-fetch](https://developers.google.com/web/updates/2017/09/abortable-fetch)
> [https://github.com/whatwg/fetch/issues/27](https://github.com/whatwg/fetch/issues/27)
