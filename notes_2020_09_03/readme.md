# `less` 使用`@import`导入时重复编译的问题

假设有一个公共的mixins.less文件, 被业务组件里的多个component.less引用, 会导致mixins.less里的内容被***重复编译输出***

这个时候`webpack`的各种插件/`optimization.commonChunks`解决不了css去重的问题(webpack解决的是通过js import时去重的问题)

需要依赖less编译时的优化, 解决方案:
```less
@import (reference) '../../../themes/mixins.less';
```

这时mixins.less文件不会被重复编译;

## less编译时的一些细节

下面两种写法:

mixins.less: 
```less
.btn {
    border-radius: 4px;
    background: blue
}
```
写法一:
```less
.btn-custom {
    .btn;
    color: #fff;
}
```
编译输出:
```css
.btn {
    border-radius: 4px;
    background: blue
}
.btn-custom {
    border-radius: 4px;
    background: blue;
    color: #fff;
}
```

写法二:
```less
.btn-custom {
    &:extend(.btn);
    color: #fff;
}
```
编译输出:
```css
.btn, .btn-custom {
    border-radius: 4px;
    background: blue
}
.btn-custom {
    color: #fff;
}
```

可以看出, 写法二`extend`的语法输出结果更优雅, 节省空间.

