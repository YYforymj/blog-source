---
title: GIT02
date: 2019-04-08 14:47:30
categories: GIT
---

> 以下内容为GIT的学习笔记，教程内容来自 [Pro Git](http://git.oschina.net/progit/)

<!-- more -->

## 其他的一些基础操作
* **对比修改**：  `git diff` 会使用文件补丁的格式显示具体添加和删除的行。当不带参数使用时，此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。若要看已经暂存起来的文件和上次提交时的快照之间的差异，可以用 `git diff --cached` 命令（或者是`git diff --staged` 命令）。

* **移除文件**： `git rm` 命令可以用来从暂存区中移除指定的文件，同时也会从工作目录中删除指定的文件。要移除跟踪但不删除文件，用 --cached 选项即可`git rm --cached readme.txt`。

* **移动文件**：正如同 `linux` 下的移动命令，`git mv file_from file_to` 可以执行移动或者重命名操作。

* **修改最后一次提交**：`git commit --amend` 此命令将使用当前的暂存区域快照提交。如果刚才提交完没有作任何改动，直接运行此命令的话，相当于有机会重新编辑提交说明，但将要提交的文件快照和之前的一样。如果刚才提交时忘了暂存某些修改，可以先补上暂存操作，然后再运行 `--amend` 提交：

  ```
  $ git commit -m 'initial commit'
      $ git add forgotten_file
      $ git commit --amend
  ```
- **取消单一文件的暂存**：`git reset HEAD <file>...` 可以将指定的文件恢复到未暂存状态。
- **取消单一文件的修改**：`git checkout -- <file>..."`可以将文件恢复到未修改的状态，对于误操作分支的时候这个命令就可以用得上了。
- **打标签**：标签实际上就是一个保存着对应提交对象的校验和信息的文件。创建一个标签可以使用`git tag tag-name` 命令。而如果想要对以前的某次提交打标签，只要在打标签的时候跟上对应提交对象的校验和即可：`$ git tag v1.0 9fceb02`。
- **显示已有标签**：使用命令`git tag` 即可展示全部标签。
- **代码回滚**：对于某次或多次已经commit的提交，可以使用`git revert`进行代码回滚。这个命令一般用于修复错误的提交。例如`git revert HEAD~3` 将会将当前分支上最后3次的提交撤销并产生一个新的提交并注明撤销了这3次提价对代码的改动。
- **选择某次提交合并**：`git cherry-pick`命令可以将某次提价的改动合并到当前分支上。
- **临时存储**：如果当前工作区有尚未完成不适合提交的代码，但临时需要切换到其他分支处理问题时，可以利用`git stash`命令将当前工作区的内容暂时存储起来，然后在完成其他分支的工作后使用`git unstash`来恢复之前未完成的代码到工作区，从而避免由于工作区未提交导致的代码丢失问题。

## 远程仓库

* **查看远程仓库**：`git remote` 命令会列出每个远程库的简短名字。加上 -v 选项（译注：此为 --verbose 的简写，取首字母），会显示对应的克隆地址。


- **添加远程仓库**：`git remote add [shortname] [url]` 用于为当前仓库添加远程仓库，同时`shortname`代指远程仓库地址。
- **抓取数据**：`git fetch [remote-name]` 用于将远程仓库中有而本地仓库没有的数据拉取到本地（包含分支），fetch 命令只是将远端的数据拉到本地仓库，并不自动合并到当前工作分支。
- **拉取与推送数据**：这两个无疑是最常用的指令了。`git pull [remote-name] [branch-name]` 用于将指定远程仓库的分支拉取到本地；`git push [remote-name] [branch-name]` 用于将本地仓库的当前分支推送到远程仓库的指定分支。

- **远程仓库删除与重命名**：修改某个远程仓库在本地的简称可以使用`git remote rename origin-name new-name`；删除指向远程仓库，使用`git remote rm some-repository`，执行此命令后，本地仓库不在同该远程仓库存在关联。

## 分支

- git 最出众、区分于其他版本管理工具的功能莫过于分支功能了。

- **新建分支**：`git branch branch-name`用于新建一个分支，但是不切换到该分支。`git checkout -b branch-anme`用于新建分支并切换到改分支。注意新建的分支以当前 head 指针指向的commit为基础。

- **切换分支**：`git checkout branch-name` 用于切换到指定分支。

- **删除分支**：`git branch -d branch-name`用于删除指定分支。

- **合并分支**：git 的分支合并是基于当前分支的。使用`git merge need-merge`命令将 `need-merge`分支上的代码合并到当前分支。

  - 冲突处理：当合并的代码发生冲突时（同时修改了某个文件导致 git 无法决定保留哪个文件中的改动），需要由人工手动处理冲突文件，然后重新进行提交。例如

    ```
    $ git merge iss53
        Auto-merging index.html
        CONFLICT (content): Merge conflict in index.html
        Automatic merge failed; fix conflicts and then commit the result.
    ```

    想要查看冲突，可以使用`git status`命令

    ````
    [master*]$ git status
        index.html: needs merge
        # On branch master
        # Changes not staged for commit:
        # (use "git add <file>..." to update what will be committed)
        # (use "git checkout -- <file>..." to discard changes in working directory)
        #
        # unmerged: index.html
        #
    ````

    对于冲突的文件，git 会将冲突的部分标出，然后由人工处理

    ```
    <<<<<<< HEAD:index.html
        <div id="footer">contact : email.support@github.com</div>
        =======
        <div id="footer">
        please contact us at support@github.com
        </div>
        >>>>>>> iss53:index.html
    ```

    ======= 隔开的上半部分，是 HEAD（即当前分支，也就是运行 merge 命令时所切换到的分支）中的内容，下半部分是在 iss53 分支中的内容。解决冲突的办法就是将此处修改为正确的代码：

    ``` 
    <div id="footer">
        please contact us at email.support@github.com
        </div>
    ```

    冲突解决后，使用`git add`、`git commit` 将冲突文件再次提交即可完成冲突处理。如果不想使用命令行可以使用可视化工具或者ide自带功能，也可以进行冲突处理。

- **列出所有分支**：使用`git branch`命令可以列出所有的分支，例如：

  ```
  $ git branch
      iss53
      * master
      testing
  ```

  其中前面带`*`的表示当前分支。对于所有的分支，可以使用`git branch --no-merged` 查看所有尚未合并到当前分支的分支，例如

  ```
  $ git branch --no-merged
      testing
  ```

  当使用`git branch -d `删除该分支时，会提示错误，因为那样做会丢失数据：

  ```
  $ git branch -d testing
      error: The branch 'testing' is not an ancestor of your current HEAD.
      If you are sure you want to delete it, run 'git branch -D testing'.
  ```


- **跟踪远程分支**：从远程分支 checkout 出来的本地分支，称为 **跟踪分支 (tracking branch)**。跟踪分支是一种和某个远程分支有直接联系的本地分支。在跟踪分支里输入 `git push`，Git 会自行推断应该向哪个服务器的哪个分支推送数据。同样，在这些分支里运行 git pull 会获取所有远程索引，并把它们的数据都合并到本地分支中来。

- **删除远程分支**：`git push [远程名] [本地分支]:[远程分支]`这条命令原本是用来推送本地仓库到远程仓库，如果省略 `[本地分支]`，那就等于是在说“在这里提取空白然后把它变成`[远程分支]`”，进而删除了远程分支。