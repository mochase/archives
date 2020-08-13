# 地图组件性能优化

## 主要的优化点
1. 优化代码逻辑, 大幅减少计算量.
2. 不需要即时返回的逻辑放入异步队列, 避免阻塞主线程
3. 尽可能避免对象深拷贝, 该操作较耗性能
4. 将大量的点数据从react 组件的props中去掉(不使用redux的数据管理方式), 减少react组件update时的计算开销, 使用订阅者模式.

## 具体操作

1. 常见的性能陷阱写法:

```js
    self.nodes.forEach(v => {
      if (actIds.includes(v.id)) {
        v.active = true
      }
    })
```
优化后
```js

  let mapping = {}
  self.nodes.forEach(v => {
    mapping[v.id] = v
  })
  actIds.forEach(m => {
    mapping[m].active = true
  })

```
优化前使用了隐式的双重for循环, 计算量o(n^2), 优化后变成了o(n). 当n = 10000时, 耗时差距在300ms左右

2. 将不需要即时返回的函数fn放入定时器, setTimeout(fn, 0). 提前响应UI.
3. 深拷贝(lodash.cloneDeep)在处理大量数据时, 很耗性能, 实际测试结果: 1万个node对象, 拷贝一次需要500ms.需要尽量避免.
4. 将大量的点数据从react 组件的props中去掉(不使用redux的数据管理方式), 减少react组件update时的计算开销(开销可能是: 框架判断是否需要更新组件时需要对props数据做diff,数据量大时计算量很大);使用订阅者模式,其他业务组件有需要使用时自行订阅.

## 其他的一些测试数据与评估
openlayers 框架读写一次的时间大概在80ms(一万个点), 优化计算与业务逻辑后, 地图组件流畅渲染的极限在5万个点左右.


