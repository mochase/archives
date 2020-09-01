## `text-transform: capitalize`

将首字母转换成大写, ***其余字符不变***(可以检测到空格)

即 `delete` => `Delete`, `DELETE` => `DELETE`, `some item` => `Some Item`

另外lodash的`capitalize`方法, 将字符串的首字母转换成大写, 其余不变(不检测空格)

即`some item` => `Some item` 


## antd 主题的动态切换

基于css变量来做antd主题切换不可行, 把css变量赋值给less变量会报错, 因为css变量(自定义属性)需要浏览器解析, less在编译阶段无法读取.

需要依赖less的动态编译(引入less.js文件), 目前的方案感觉不太优雅, 暂时没有更好的解决方案(抛开antd组件库的话, 直接css变量搞定)

index.html:

```html
<script>
    window.less = {
        async: false,
        env: 'production'//production  development
    };
</script>
<div id="root"></div>
<script src="https://cdn.bootcss.com/less.js/2.7.3/less.min.js">
```

切换:
```js
window.less.modifyVars(
        {
            '@primary-color': '#e64e14',
            '@btn-primary-bg': '#5d72cc',
        }
    )
    .then(() => {console.log('success')})
    .catch(error => {
        console.log(error);
    });
```