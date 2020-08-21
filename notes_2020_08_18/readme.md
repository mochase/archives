# 关于`line-height` 和 `vertical-align`的计算(内联格式化上下文)

`content-area(由字体度量决定, 因字体而异)/virtual-area(line-height)`

`vertical-align: middle`:  `baseline + 1/2 * x-Height`对齐. 完全垂直居中似乎很难

`line-box`的高度: `line-height`的最高点 - 最低点. 

计算`line-box`的高度时不可见元素`strut`的影响, 导致奇怪的现象..

```css
p {
    line-height: 200px;
}
span {
    font-size: 100px;
    font-family: Catamaran;
}

```
```html
<p>
    <span>bar</span>
</p>
```
看不见的`strut`(默认可能是`serif`字体), 不同的字体baseline对齐 + span继承的line-height, 导致line-box 的高度可能大于200px..

`vertical-align: top | bottom`: 和`line-box`的顶部或底部对齐.

`vertical-align: text-top | text-bottom`: 和`content-area`的顶部或底部对齐