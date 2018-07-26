---
title: GIT01
date: 2017-08-4 10:33:30
categories: GIT
---

> 以下内容为GIT的学习笔记，教程内容来自[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)与[Pro Git](http://git.oschina.net/progit/)

<!-- more -->

## 基础操作
* **创建仓库**： 使用 `git init` 在需要创建仓库的文件夹下创建仓库，仓库创建完成后，该文件夹下会出现一个名为 .git 的隐藏文件夹。
* **创建版本**： 在仓库中创建新的文件用于测试`touch test`;随意输入数据后，使用`git add test`将文件添加到暂存区，此时文件还没正式提交到版本库，使用`git status`命令可以查看仓库当前状态：
```
    On branch master
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)
    
    	modified:   test
```
* 这意味这test已经被修改但还没提交，此时使用`git commit`命令即可提交，提交时需要输入此次提交的备注信息，git使用nano打开，编辑完成后使用ctrl+x退出，并选择y即可；此时再次使用`git status`查看状态，得到如下信息，说明提交成功
```
    On branch master
    nothing to commit, working directory clean
```
* 使用`git log`命令可以查看提交的历史记录，加上`--pretty=oneline`作为参数可以显示简略信息：
```
    02a7c37149d512f9988c69c8ae2aa63a1d2e2446 yse
    555100819ac480d45be770a9c590f71176c7e0f3 test02
    a011dbcf3f42b9c75ef1a0bca265a354f3658274 test01
```
* 前面的一串字符为`commit id`,是利用所有数据内容计算出的校验和（checksum），此结果作为数据的唯一标识和索引，用于标识版本；在 git 里，一旦提交快照之后就完全不用担心丢失数据，特别是养成定期推送到其他仓库的习惯的话。
* **版本回退**：在git中`HEAD`表示当前版本，上一个版本可以用`HEAD^`表示，上上版本为`HEAD^^`,上N个版本用`HEAD~N`表示；比如在上面的几个版本中，`HEAD`版本为`02a7c37149d512f9988c69c8ae2aa63a1d2e2446`，`HEAD^`为`555100819ac480d45be770a9c590f71176c7e0f3`。如果在编程过程中需要回退版本，则可以使用`git reset`命令，效果如下：
```
    $ git reset --hard  HEAD^
    HEAD is now at 5551008 test02
```
* 此时调用`git log`命令会发现yse版本被撤回：
```
    $ git log --pretty=oneline
    555100819ac480d45be770a9c590f71176c7e0f3 test02
    a011dbcf3f42b9c75ef1a0bca265a354f3658274 test01
```
* 如果再需要恢复yse这个版本，需要执行命令`git reset --hard 02a7c3`，其中02a7c3是yse版本的头几位，git会根据这几位自动寻找对应的版本，这个版本号输入的时候没有固定长度要求，只要能够满足查找到唯一确定的版本即可。效果如下
```
    git reset --hard  02a7c
    HEAD is now at 02a7c37 yse
    
    git log --pretty=oneline
    02a7c37149d512f9988c69c8ae2aa63a1d2e2446 yse
    555100819ac480d45be770a9c590f71176c7e0f3 test02
    a011dbcf3f42b9c75ef1a0bca265a354f3658274 test01
```
* 可以看到已经成功恢复版本。Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从指向待恢复版本。
* git提供了一个命令`git reflog`用来记录你的每一次命令，可以利用这个指令来恢复到可追溯到的任意版本。
* `git add`指令是将文件提交到暂存区，`git commit`是将暂存区文件提交到master分支。如果需要将文件提交到master分支，需要先add到暂存区，再commit到master分支，没有add到暂存区的文件不会commit到master分支。
## 核心概念
* 所有保存在 Git 数据库中的东西都是用此哈希值来作索引的，而不是靠文件名。

* 对于任何一个文件，在 Git 内都只有三种状态：已提交（committed），已修改（modified）和已暂存（staged）。已提交表示该文件已经被安全地保存在本地数据库中了；已修改表示修改了某个文件，但还没有提交到暂存区；已暂存表示把已修改的文件存至暂存区。

   ![git_3_stages](\images\GIT01\git_3_stages.png)

  