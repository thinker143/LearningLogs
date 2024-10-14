## Git分布式版本控制工具

### 解决GitBash乱码问题

1、打开GitBash执行下面命令

```
git config --global core.quotepath false
```

2、${git_home}/etc/bash.bashrc 文件最后加入下面两行

```
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"
```

