## Git分布式版本控制工具

### 常用命令

```
ls/ll（ls -al）  查看当前目录
cat  查看文件内容
touch  创建文件
vi   vi编译器（使用vi编译器是为了方便展示效果，也可以用其他编译器）
```

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

