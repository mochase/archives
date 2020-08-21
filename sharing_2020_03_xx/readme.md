# 对html打标签的前端实现总结

## 基本思路

打标签的实质是给获取到的文本外面包一层span 标签,然后对该span标签着色高亮;

我们可以通过window.selection()来获取到选中的文本,目标是找到一种数据结构,描述标签的位置信息并记录;下次加载时,通过标签的位置信息,再拼接回来,实现着色高亮,以及添加各种事件.

这两个过程我们分别称之为`序列化`与`反序列化`.

## 具体的数据结构

```js
{
  
    startMeta: {
        parentTagName: "DIV",
        parentIndex: 84,
        textOffset: 70
 
    },
 
    endMeta: {
        parentTagName: 'SPAN',
        parentIndex: 80,
        textOffset: 70
 
    },
    text: "张三",
    id: 'xxxxxxx'
}
```

startMeta, endMeta分别描述标签的起始位置和结束位置.

parentTagName表示当前位置的父节点name, parentIndex则表示该父节点在整个dom树中的同类节点序号,形如 document.getElementsByTagName('div')[parentIndex]
text 则记录了选中文本的内容.

这些信息唯一描述了一个选中文本的位置.因为要考虑选中文本跨dom节点的问题,形如 `<p>张三</p><div>李四</div>`, 如果选中了"三李",最后需要转换成`<p>张<span>三</span></p><div><span>李</span>四</div>`; 反序列化的工作有一定难度,在时间紧张的情况下;好在社区已经有实现该功能的库了,拿来用即可.

## 嵌入网页到项目中的方案

一般来说,嵌入网页第一时间想到的是用iframe;iframe天然具有隔离性, 但是由于这里内嵌网页的富交互性,需要对网页生成的标签添加拖拽等自定义事件,与父容器交互;需要给iframe注入第三方package,父子容器通信的不方便,在一番尝试无果后遂放弃;

可不可以直接innerHTML插入到当前页面中呢?三方脚本以及xss注入的问题我们可以通过正则或三方package来过滤;但是css嵌入的问题似乎无法解决.由于css语法天然没有"局部变量"的概念,始终全局生效,第三方网页的css会直接污染我们页面中的其他模块,造成不可预知的样式错乱. 现在的目标是找到一种方案,来把三方网页的css变成"局部变量",和页面中的其他组件隔离开.

通过查阅文档,最终我们找到了一种dom结构: "shadow dom", 它内部的css,生效范围是局限在shadow dom内的,知道这一点后,我们就知道目标可以实现了.

我们把抓取的三方网页插入到构造的shadow dom中, 用它来隔离css,并用相关库来过滤脚本和危险代码;

然而,事情不是一蹴而就的.在调试的时候发现,大部分浏览器(包括chrome),获取选中文本的关键api: window.getSelection()在处理shadow dom的时候,无法正确返回.这时候唯一能正确识别的只有Firefox ... 一脸辛酸泪

好在,社区上已经有相关api的polyfill可用了,拿来即用~ 题外话,关于这个bug,看社区上在2016年就已经有标准相关人员在讨论了,马上就2020年了...

## 其他的一些插曲

1. 由于我们的网页要支持ner 自动打标签,所以同样的序列化与反序列化算法,要由后端同学重新实现一遍. 在实际调试的过程中,发现某些网站用ner分词以后,出现大面积的分词错乱情况.经过反复调试debug,确认是前后端两套算法实现上有细微差异.

查看package源码,定位到是关于parentIndex的计算上.package相关源码:

```js
const $list = $root.getElementsByTagName(tagName);
for (let i = 0; i < $list.length; i++) {
    if ($node === $list[i]) {
        return i;
    }
}
return -1;
```

这里关于parentIndex计算,隐式依赖了我们之前插入其中的标签,即 tagName === "SPAN"的情况,因为标签同样是用SPAN 插入的.

和后端保持一致,这里对集合过滤掉标签dom, 改成:

```js
const $list = $root.querySelectorAll(`${tagName}:not([data-highlight-id])`);
```

反序列化时的逻辑类似. 将package源码修改后重新build,再引入到项目中,大功告成~

附该package地址: [https://github.com/alienzhou/web-highlighter](https://github.com/alienzhou/web-highlighter)

2. 注意对嵌入网页的可执行脚本做过滤.有些门户网站会自己给自己注入脚本,定时检测当前域名,如果不匹配则直接跳转到指定域名.以下是csdn网站源码中的注入实例:

```js

<div style="display:none;">
    <img src="" onerror='setTimeout(function(){if(!/(csdn.net|iteye.com|baiducontent.com|googleusercontent.com|360webcache.com|sogoucdn.com|bingj.com|baidu.com)$/.test(window.location.hostname)){window.location.href="\x68\x74\x74\x70\x73\x3a\x2f\x2f\x77\x77\x77\x2e\x63\x73\x64\x6e\x2e\x6e\x65\x74"}},3000);'>
</div>
```