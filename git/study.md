# 2024年10月10日 

## Git分布式版本控制工具

### 常用命令

```
ls/ll（ls -al）  查看当前目录
cat  查看文件内容
touch  创建文件
vi   vi编译器（使用vi编译器是为了方便展示效果，也可以用其他编译器）
```



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



### 获取本地仓库

1、在电脑任意位置创建空目录，代表我们的本地仓库

2、进入目录中，打开Git bash窗口

3、执行命令git init

4、创建成功后可以在文件夹下看到隐藏的.git目录



### 基础操作指令

1、git add             (工作区 -->  暂存区)
2、git commit          (暂存区 -->  本地仓库)

**查看修改的状态（status）**
命令形式: git status

**添加工作区到暂存区（add）**
命令形式: git add  单个文件名|通配符
         git add .  (将所有修改加入暂存区)
         
**提交暂存区到本地仓库（commit）**
作用：提交暂存区内容到本地仓库的当前分支
命令形式：git commit -m "注释内容"

**查看提交日志**
（配置了别名：
#用于输出git提交日志
alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'）
命令形式：git log[option]
⚪option
     --all 显示所有分支
     --pretty=oneline 将提交信息显示为一行
     --abbrev-commit 使得输出的commit更简短
     --graph  以图的形式显示
     
**版本回退**

命令形式：git reset --hard commitID

​                   commitID可以使用git-log或git log指令查看

⚪如何查看已经删除的记录

git reflog

可以查看已经删除的提交记录



### 分支

**1、查看本地分支**

命令：git branch

**2、创建本地分支**

命令：git branch 分支名

**3、切换分支（checkout）**

命令: git checkout  分支名

还可以直接切换一个不存在的分支（创建并切换）

命令：git checkout -b 分支名

**4、合并分支（merge）**

一个分支上的提交可以合并到另一个分支

命令：git merge 分支名称

**5、删除分支**

**不能删除当前分支，只能删除其他分支**

git branch -d b1 删除分支时，需要做各种检查

git branch -D b1 不做任何检查，强制删除

# 2024年10月11日

### 操作远程仓库

**1、添加远程仓库**

命令：git remote add <远端名称> <仓库路径>

**2、查看远程仓库**

命令：git remote

**3、推送到远程仓库**

命令：：git push [-f] [--set-upstream]   [远端名称 [本地分支名][:远端分支名] ]

注解：[--set-upstream]  推送到远端的同时建立起和远端分支的关联

​            [-f] 强制覆盖（不建议使用）

**4、从远程仓库克隆**

命令：git clone <仓库路径> [本地目录]（可以省略，会自动生成一个目录）

**5、从远程仓库抓取和拉取**

抓取命令：git fetch [remote name] [branch name]

拉取命令：git pull [remote name] [branch name]

注释：如果不指定远端名称和分支名，则抓取所有分支。

### 常用命令总结

##### 基本操作类：

######        git init

​                  初始化仓库

######        git-log

​                  查看日志

######        git add <文件名|.>

​                   添加到暂存区

######        git commit -m '注释'

​                   提交到仓库

######        git merge <分支名>

​                  合并指定分支到当前活跃分支

##### 分支切换类：

######        git checkout <分支名>

​                                  切换到某个分支

######        git checkout -b <分支名>

​                                   创建并切换到某个分支（分支原来不存在时）

##### 远程操作：

######        git clone <远程地址> [本地文件夹]

​                       clone远程仓库到本地

######        git pull

​                       拉取远端仓库的修改并合并

######        git push [--set-upstream] origin 分支名

​                       推送本地修改到远端分支

​                       --set-upstream表示和远端分支绑定关联关系，只有第一次推送才需要此参数



# 2024年10月12日

### WSL安装

##### windows11安装wsl

搜索栏搜索启用或关闭windows功能，然后打开两个功能：一、适用于Linux的Windows子系统 。 二、虚拟机平台。

微软商城下载Ubuntu20.04.6，然后powershell里面输入 命令wsl --update  成功安装。



### ant design学习

照着官网，把快速入手里面的代码在记事本中敲了一遍，每一行代码都百度了一下，搞清楚了每行代码的具体含义。



# 2024年10月13日

