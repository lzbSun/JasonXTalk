# 如何串行/并行处理多个 async/await 
> 引子：
> 思考一个问题：这里有多张图片需要加载，有两种方式一种方式是一次性加载完，另外一种是一张一张依次的加载，那么应该怎么实现这个功能？

在我们在线教学的课堂的实践中，课堂中的教材是由多张图片组成的，为了在课中实现图片的快速加载，我们会在初始化进入教室的时候给所有的教材图片做预加载（当然不是最佳的实践方案，但是本篇旨在讨论 async/await 串并行问题），然后利用 HTTP 的缓存做到快速获取图片。

那么第一种我们要最快的速度获取到所有的图片，那么一定选择并行的方案，我们来看并行的方案

## 并行方案
```javascript

// 统一加载图片
const getImage = url => {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => {
      resolve(`${url} image load success`)
    }
    img.onerror = () => {
      resolve(`${url} image load error`)
    }
    img.src = url
  })
}

```

```javascript

// 并行
const getImages = images => {
  images.forEach(async item => {
    await getImage(item)
  })
}

```

那么我们来看为什么这里是并行的？

当我们的 js 遇到 forEach 函数的时候其实是同步执行的，也就是说一股脑的把所有的异步函数放到了 microtask 的队列中了，当我们的 forEach 执行完后开始执行我们的 microtask 队列中所有的单项。


## 串行方案

当我们想做串行方案的时候，就是说需要等待一张图片加载完或者报错后再加载另一张图片，那么我们来看串行的方案

```javascript

// 串行
const getImages = async images => {
  for (const item of images) {
      await getImage(item)
  }
}

```
 那么我们来看为什么这里是串行的？

我们把代码分解下
for 循环这块就相当于
```javascript
 await getImage(item1)
 await getImage(item2)
 await getImage(item3)
 ...
 await getImage(itemN)
```
那么在执行中，item2 就是要等待 item1 的执行结果执行完成后才执行的, item3 依赖 item3，依次类推。

##  区别
那我们来看这两种方案的区别是什么呢？

我们看到 getImages 函数一个是同步的函数，一个是异步的函数仅此而已。

这块其实跟 JavaScript 中的时间循环有关系，如果看不懂可以参考这个[链接](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/7?from=groupmessage&isappinstalled=0) 学习，我就不再次废话了，这位同学写的很清楚了。