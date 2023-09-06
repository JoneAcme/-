Git 笔记



[TOC]







## 常用命令

| 命令                | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| git status          | 查看当前状态                                                 |
| git log             | 查看提交log                                                  |
| git add             | 添加文件 后接文件名 或者. （添加目录下所有文件）             |
| git restore --staged <文件>...       | 取消暂存 |
| git reset --hard    | 清除缓存区的操作                                             |
|git reset --soft HEAD^|撤回上一次commit                                           |
| git commit -m'描述' | 提交文件到本地仓库                                           |
| git branch          | 查看本地分支  -v  -av                                        |
| git checkout XXX    | 取出、切换到XXX分支、（-b nameA） 创建新分支A、（-b nameA nameB） 基于分支B创建分支A |
| git diff            | git diff <commitIdA> <commitIdB> 两次提交的不同              |
| git pull --rebase | 把远程分支的内容拉取到本地，接着再自动本地commit放在最上面。如有冲突，解决冲突后使用git rebase --continue来继续rebase操作 |
| git branch -vv      | 查看本地分支对应的远程分支              |
| git stash           | 将当前的修改暂存起来，工作区与暂存区恢复到HEAD               |
| git stash apply     | 取出stash 最上层的一条恢复到工作区，但是不会从stash list移除对应记录 |
| git stash pop       | 取出stash 最上层的一条恢复到工作区，并移除 stash list中对应记录 |
| git branch --set-upstream-to origin/xxx       | 将当前分支关联到远程xxx分支 |


### 日志常用操作

| 命令      | 描述                |
| --------- | ------------------- |
| --oneline | 简洁版本            |
| -nA       | 查看最近A次提交记录 |
| --all     | 查看所有分支Log     |
| --graph   | 图形化Log           |
|           |                     |



## 基本操作

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
- -r ：查看远程分支
- -a：查看所有分支
- -d ：删除分支
- -D：强制删除分支
- git push origin --delete XXX ： 删除远程分支


### commit

`git commit`

- -m ：添加描述提交
- --amend ：修改最近一次提交描述

### rebase

`git rebase`

**如果你的当前分支落后于远程分支，并且你有了自己新的commit，使用git pull会产生一个merge的commit信息。如果使用git pull --rebase，那么会git会将你的commit先放一边，然后把远程分支的内容拉取到你的本地，接着再自动把你的commit放在最上面。中途发生冲突，在解决冲突后使用git rebase --continue来继续rebase操作**

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

### remote

| 命令                           | 描述                                                      |
| ------------------------------ | --------------------------------------------------------- |
| git remote                     | 列出所有远程主机 （使用`-v`选项，可以参看远程主机的网址） |
| git remote show <主机名>       | 查看该主机的详细信息                                      |
| git remote add <主机名> <网址> | 添加远程主机                                              |
| git remote rm <主机名>         | 删除远程主机                                              |
| git remote rename              | 远程主机改名                                              |

### fetch

`git fetch <远程主机名>` ： 将某个远程主机的更新，全部取回本地

然后使用merge 或者 rebase 合并到当前分支

```
git merge <远程主机名>/<分支名>  
或者
git rebase <远程主机名>/<分支名>  
```



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



## 常见合并等操作

### 合并commit

1. git rebase -i head~3   编辑最新的3条提交记录

2. 此时会进入一个编辑界面, 显示最近3条记录, 默认是 pick 操作 ，改为 squash 操作

3. 如果合并成功会打开另外一个文件文件，在这里我们输入这次合并时的提交记录信息。

   如果合并有冲突，在解决冲突后需要输入：

   git add .

   如果不想合并了，放弃合并的指令是：

   git rebase --abort

4. 到这一步，本地commit合并已完成

5. 会有提示pull 一次，不可pull ！！ 否则合并无效，还会产生新的commit。

6. 合并完成后，如果需要同步到远程，远程的也合并， -f 强制提交，注意：可能会覆盖其他人代码

   执行：git push -f   /  push到远程dev_test 分支： git push origin dev_test -f 

7. 查看log ，本地远程commit 都已经合并



### rebease 中的操作及其含义

```
	# 命令:
# p, pick <提交> = 使用提交
# r, reword <提交> = 使用提交，但修改提交说明
# e, edit <提交> = 使用提交，进入 shell 以便进行提交修补
# s, squash <提交> = 使用提交，但融合到前一个提交
# f, fixup <提交> = 类似于 "squash"，但丢弃提交说明日志
# x, exec <命令> = 使用 shell 运行命令（此行剩余部分）
# b, break = 在此处停止（使用 'git rebase --continue' 继续变基）
# d, drop <提交> = 删除提交
# l, label <label> = 为当前 HEAD 打上标记
# t, reset <label> = 重置 HEAD 到该标记
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       创建一个合并提交，并使用原始的合并提交说明（如果没有指定
# .       原始提交，使用注释部分的 oneline 作为提交说明）。使用
# .       -c <提交> 可以编辑提交说明。
#
# 可以对这些行重新排序，将从上至下执行。
#
# 如果您在这里删除一行，对应的提交将会丢失。
```



### 合并一个commit 到当前分支

```shell
git cherry-pick <commitId>
```

 git cherry-pick [commitID] 提取一个commit 

 git cherry-pick [start-commitID]..[end-commitID] 提取一个commit到另一个commit之间的所以commit，不包括start-commitID，包括end-commitID。 

 git cherry-pick start-commitID]^..[end-commitID] 提取一个commit到另一个commit之间的所有commit，包括start-commitID，包括end-commitID。

### git push 多个commit中的一个

git push [remote] [commit-id]:[branch]

### 将自己分支以未跟踪状态合并到新分支

通常我们会在自己分支开发功能，可能会产生大量的commit，当需要合并到dev时，把自己分支所有改动一次性全部以未跟踪的方式抓到dev可以使用。先切到dev分支并更新。

```
git merge [your branch] --squash
```

### 删除远程分支
git push -d origin branchName





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





