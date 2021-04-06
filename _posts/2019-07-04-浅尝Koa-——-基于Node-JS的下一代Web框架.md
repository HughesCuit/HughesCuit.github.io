---
layout: post
title: 浅尝Koa —— 基于Node.JS的下一代Web框架
tags:
  - Web
  - Koa
  - Node.JS
categories:
  - 开发笔记
date: 2019-07-04 09:36:00
---

## What is Koa?

`Koa` 是 Node.JS 平台下的一款 Web 服务器框架，它由原`Express`团队开发，区别于`Express`大量使用`回调函数`的形式，`Koa`采用最新的`async/await`模式，使用异步函数的方式来处理 HTTP 请求

## Why Koa？

相比于`Express`是一个高度集成化的 Web 框架，`Koa`显得更加的**小而美**，
它本身并不集成任何的中间件(包括路由)，但得益于它优秀的理念，以及便捷的模式，我们可以通过大量的第三方库对其功能进行扩充。

从业务上来说，使用`Koa`可以很容易地控制项目的规模。

从开发上来说，`Express`使用 callback 的方式处理异步，而在`Koa`中，`Koa1`采用 generator，`Koa2`使用`async/await`的方式来处理异步流程控制。

例如使用 callback 的方式请求资源：

```javascript
fetchData(resourceID, method, function (data) {
  process(data, function (result) {
    handle(result);
  });
});
```

换成 generator 的模式：

```javascript
function* fetchData() {
  var result = yield fetch(resourceID);
  return result;
}

var data = fetchData();
data.next();
var result = process(data);
result.next();
handle(result);
```

再使用 async/await 模式：

```javascript
async function fetchData() {
  return fetch(resourceID);
}

var data = await fetchData();
var result = await process(data);
handle(result);
```

可以看出，使用 async/await 的方式可以使得异步编程更加简洁易懂,而且最新的 JS 为我们提供了这样方便的模式，我们没有理由不使用更好且更新的东西。

## 使用 Koa2

### 使用 Koa 实现 Hello World

```javascript
const Koa = require("koa");
const app = new Koa();

async function hello(ctx, next) {
  ctx.body = "Hello World!";
  next();
}

async function welcome(ctx, next) {
  await next();
  ctx.body += "Welcome To Koa2";
}
app.use(hello);
app.use(welcome);

app.listen(3000);
```

### Context

与`Express`中不同，`Koa`将`request`和`response`集成到了`Context`中,也就是我们实现中间件函数中的`ctx`参数，可以通过`ctx.request`和`ctx.response`来对请求和响应进行操作。

## 展望

目前`Koa`社区尚且比较小，但其量级够轻，并且更加先进，这应该是一个值得开拓的新技术，对于开发者自身来说，使用 JS 语言的新特性用也可以更加顺手
