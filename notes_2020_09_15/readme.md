# 循环引用的对象执行`JSON.stringify()`会报错的问题研究

## 从一个诡异bug说起

保存antv g6的edges数据, 由于未对原始数据做过滤, 当快速切换节点时, 调用axios 去调用save接口, 发现http请求没有发出去被提前返回了.

再三定位后发现, g6的edges数据中存在循环引用(可能是快速切换节点导致..), axios 以application/json格式发送请求时, 必然要先执行JSON.stringify(), 然后JSON.stringify由于循环引用报错, 没有正确捕获异常, 导致上述的奇怪现象.

## 循环引用例子

```js
let a = {}
let b = {}
a.child = b
b.child = a
```

还有隐式的循环引用:
```js
const div = document.querySelector('.demo')
div.onclick = function () {
    doSomething(div)
}
```
div 的onClick函数里隐式保存了对div的引用

在早期的ie6, 7 这样写代码导致`div` 变量无法被`GC`, 导致内存泄漏

## 早期的引用计数法

算法含义: 把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

该算法无法处理循环引用的场景, 导致问题

## 改进的标记-清除算法

算法含义: 把"对象是否不再需要"简化定义为"对象是否可以获得". 改进的关键点在于: ***因为“有零引用的对象”总是不可获得的，但是相反却不一定***

大致思路: 把变量构建成一颗树, 根节点可以看做全局对象(js 中即window), 定时的去更新这棵树, 如果节点可访问, 则标记. 如果不可访问则表示***不再需要***, 可以被GC

看起来和webpack的`tree shaking`算法有点类似, 有时间再专门研究一下

## JSON.stringify 处理循环应用

```js
// 遇到循环引用直接跳过
var seen = []
function replacer(key, value) {
    if (typeof value === 'object' && value !== null) {
        if (seen.indexOf(value) !== -1) {
            return
        }
        seen.push(value)
    }
    return value
}

JSON.stringify(obj, replacer)
```



