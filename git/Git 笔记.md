Git 笔记



[toc]







## 常用命令

| 命令                | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| git status          | 查看当前状态                                                 |
| git log             | 查看提交log                                                  |
| git add             | 添加文件 后接文件名 或者. （添加目录下所有文件）             |
| git reset --hard    | 清除缓存区的操作                                             |
| git commit -m'描述' | 提交文件到本地仓库                                           |
| git branch          | 查看本地分支  -v  -av                                        |
| git checkout XXX    | 取出、切换到XXX分支、（-b nameA） 创建新分支A、（-b nameA nameB） 基于分支B创建分支A |
| git diff            | git diff <commitIdA> <commitIdB> 两次提交的不同              |
| git stash           | 将当前的修改暂存起来，工作区与暂存区恢复到HEAD               |
| git stash apply     | 取出stash 最上层的一条恢复到工作区，但是不会从stash list移除对应记录 |
| git stash pop       | 取出stash 最上层的一条恢复到工作区，并移除 stash list中对应记录 |



### 日志常用操作

| 命令      | 描述                |
| --------- | ------------------- |
| --oneline | 简洁版本            |
| -nA       | 查看最近A次提交记录 |
| --all     | 查看所有分支Log     |
| --graph   | 图形化Log           |
|           |                     |



## 一些常见操作

### 配置



```shell
git config --global user.name 'name'
git config --global user.email 'email'
```

- git config --local 只对某个仓库有效
- git config --global 对当前用户所有仓库有效
- git config --system 对系统所有登录的用户有效

查看配置 

-  git config --local  --list
- git config  --global  --list
- git config  --system  --list

### 新建仓库

1. 把一有的项目纳入git

   ```shell
   cd 目录
   git init
   ```

   

2. 新建的项目直接用git

   ```shell
   cd 目录
   git init your_project #会在当前路径下创建与项目名称相同的文件夹
   cd your_project
   ```





### 新建分支

`git checkout XXX`

- 取出、切换到XXX分支
- -b nameA：创建新分支A
- -b nameA nameB：基于分支B创建分支A

### branch

`git branch`

- --v ：查看分支，详细信息，仅本地
- -av  ：查看分支，详细信息，本地+远程
- -d ：删除分支
- -D：强制删除分支



### commit

`git commit`

- -m ：添加描述提交
- --amend ：修改最近一次提交描述

### rebase

`git rebase`

- -i  <commitId>： 基于要修改的commit 父亲去修改，

  修改历史提交描述：

  1. 其中交互信息 ，将要修改的描述前 `pick` 策略修改为`reword` 
  2. 保存退出后，会自动弹出要修改的描述信息，更改其描述，保存退出即可

  合并commit：[参考链接](https://segmentfault.com/a/1190000007748862)

  1.  `pick` 策略修改为`squash` ，提交保留，合并到之前提交中
  2. 保存退出后，会自动弹出要修改的描述信息，更改其描述，保存退出即可

  > 选取最近的3个版本  git rebase -i HEAD~3
  >
  > 指明要合并的版本之前的版本号 git rebase -i <版本号>
  >
  > 
  >
  > 修改策略时候，最新的提交在最下面，如果要合并间隔commit，需要把要合并的commit 放在一起，然后修改策略为squash

### reset（暂存区修改）

`git reset`

`git reset HEAD`：移除所有更改，所有文件恢复到HEAD

`git reset HEAD <file>`：移除暂存区中对file更改

`git reset --hart <commitId>`：将工作区和暂存区回退到<commitId>，也就是移除上方的commit

### checkout （工作区修改）

`git checkout`

恢复暂存区文件到工作区：

git checkout -- <文件名>



### 对文件重命名

方法1.

```shell
mv readme readme.md
git add readme.md
git rm readme
```

方法2.

```shell
git mv readme readme.md
```



### 分离头指针

checkout <commitId>

注：分离出来的并不属于任何一个分支，独立于分支之外，如果切换回其他分支时候，且没有保存当前分离出来的头指针，那么期间的操作将都会被丢弃。

保留分离的头指针中所作的修改，需要给这个头指针绑定一个分支， git branch <name> <commitId>



### HEAD/diff

当前最新的Commit

如，比较最新两次的提交有什么区别：

1. git diff HEAD HEAD~1  
2. git diff HEAD HEAD^1   
3.   git diff  <commitIdA> <commitIdB> 

- HEAD^1、HEAD^  ：HEAD 的父级
- HEAD^^  ：HEAD 父级的父级
-  HEAD~1：HEAD 的父级
-  HEAD~2：HEAD 父级的父级

 两个分支（指定文件）的差异：

- `git diff <branch1> <branch2>（ -- 文件名）` 
- `git diff <branch1 HEAD > <branch2 HEAD>（ -- 文件名）`

`git diff` 工作区和暂存区的区别

`git diff --cached`  暂存区与HEAD 的区别

> -- 文件名 ：具体文件差别，可添加多个，



### stash（暂存）

作用：可将当前的修改保留到其他位置，当前工作区与暂存区恢复到HEAD，在修改完其他需求后，再从其他位置恢复回来

- `git stash`  将当前的修改暂存起来，工作区与暂存区恢复到HEAD  
- `git stash apply`  取出stash 最上层的一条恢复到工作区，但是不会从stash list移除对应记录  
- `git stash pop`   取出stash 最上层的一条恢复到工作区，并移除 stash list中对应记录

### 恢复文件



## 常用Linux 命令 todo

| 命令    | 作用                                                |
| ------- | --------------------------------------------------- |
| cd      | 切换目录  cd XXX 打开XXX目录 、cd .. 返回上册才能够 |
| ls      | 查看当前文件列表 -al 包括修改用户、时间等信息       |
| mv      | move 移动文件、目录、更改名称                       |
| rm      | remove 删除文件或目录                               |
| cat     | 查看文本文件内容，后接要查看的文件名                |
| vi、vim | 打开文件                                            |
| :wq     | 退出编辑                                            |
| i       | 编辑文本                                            |
| dd      | 删除行                                              |
| pwd     | 当前目录的全路径                                    |

