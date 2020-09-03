---
title: CSS Dark Mode
date: 2020-03-11 20:16:40
tags: css
---

简单实现，主要依靠 media query 的 prefers-color-scheme

```css
@media screen and (prefers-color-scheme: light) {
  body {
    background-color: white;
    color: black;
  }
}

@media screen and (prefers-color-scheme: dark) {
  body {
    background-color: black;
    color: white;
  }
}
```

还可通过 css var 实现。

> [https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)
> [https://css-tricks.com/dark-modes-with-css/](https://css-tricks.com/dark-modes-with-css/)
> [https://medium.com/@mwichary/dark-theme-in-a-day-3518dde2955a](https://medium.com/@mwichary/dark-theme-in-a-day-3518dde2955a)
