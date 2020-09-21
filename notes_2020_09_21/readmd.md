# git 忽略规则(.gitignore配置)不生效的原因和解决办法

如果文件在git中有缓存, 即它已经被纳入了版本管理中, 这时如果再在.gitignore中声明该文件, 也是不起作用的

> .gitignore只能忽略那些原来没有被track的文件

## 解决办法

先清除该文件的缓存, 设置成untrack状态

```shell
git rm -r --cached $filepath
```

然后正常`add`, `commit`, 会自动忽略该文件

```shell
git add .
git commit -m "ignore the file"
```