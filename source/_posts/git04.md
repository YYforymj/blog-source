---
title: GIT04
date: 2019-04-12 17:55:00
categories: GIT

---

> 以下内容为GIT的学习笔记，教程内容来自 [Pro Git](<https://gitee.com/progit/7-%E8%87%AA%E5%AE%9A%E4%B9%89-Git.html>)与[网络博客](<https://zhaohuabing.com/post/2019-01-21-git/>)。

<!-- more -->

## git配置与hooks

- **设置姓名和邮箱**：使用`git config` 命令可以设置git用户的姓名与邮箱，例如：

  ```
  $ git config --global user.name "John Doe"
  $ git config --global user.email johndoe@example.com
  ```

- **配置文件**：Git 使用一系列的配置文件来存储你定义的偏好，它首先会查找`/etc/gitconfig`文件，该文件含有 对系统上所有用户及他们所拥有的仓库都生效的配置值（gitconfig是全局配置文件）， 如果传递--system选项给`git config`命令， Git 会读写这个文件。

  接下来 Git 会查找每个用户的`~/.gitconfig`文件，你能传递`--global`选项让 Git读写该文件。

  最后 Git 会查找由用户定义的各个库中 Git 目录下的配置文件（`.git/config`），该文件中的值只对当前库有效。 以上阐述的三层配置从一般到特殊层层推进，如果定义的值有冲突，以靠后层中定义的为准，例如：如果同时配置了`.git/config`和`/etc/gitconfig`， 则Git会使用`.git/config`文件中的配置。

  - **core.autocrlf**：当开发环境（windows）与生产环境（linux）的换行符不同时，可以将`core.autocrlf`设置为`true`，这样当检出代码时，LF会被转换成CRLF：

- **git hooks**：git hooks 的功能很类似于spring中aop的概念，当某些对git执行某些操作时，可以通过git hooks 触发一些脚本，实现提交检查、自动化部署等功能。通常来说， git hooks 可以分为两种：客户端和服务器端。客户端挂钩用于客户端的操作，如提交和合并。服务器端挂钩用于 Git 服务器端的操作，如接收被推送的提交。

  - 挂钩都被存储在 Git 目录下的`hooks`子目录中，即大部分项目中的`.git/hooks`。 默认情况下，这里会存储一些 sample 脚本。git hooks支持多种脚本语言：shell、python、Ruby都可以执行，但是必须要以正确的后缀结尾。

## git 内部原理

- Git的本质是一个文件系统，其工作目录中的所有文件的历史版本以及提交记录(Commit)都是以文件对象的方式保存在.git目录中的。

- 当新增一个commit时，git会在`.git/objects`目录下增加文件，Git Object目录中存储了三种对象：Commit， tree和blob。Git为对象生成一个文件，并根据文件信息生成一个 SHA-1 哈希值作为文件内容的校验和，创建以该校验和前两个字符为名称的子目录，并以 (校验和) 剩下 38 个字符为文件命名 ，将该文件保存至子目录下。通过 git cat-file命令可以查看Git Object中存储的内容及对象类型，命令参数为Git Object的SHA-1哈希值，即目录名+文件名。当前分支的对象引用保存在HEAD文件中，可以查看该文件得到当前HEAD对应的branch，并通过branch查到对应的commit对象。

  ```
  $ cat .git/HEAD
  ref: refs/heads/master
  cat .git/refs/heads/master
  b767d7115ef57666c9d279c7acc955f86f298a8d
  ```

  使用 -p 参数可以查看文件内容：

  ```
  $ git cat-file -p b767d7
  tree ca964f37599d41e285d1a71d11495ddc486b6c3b
  author Huabing Zhao <zhaohuabing@gmail.com> 1548055516 +0800
  committer Huabing Zhao <zhaohuabing@gmail.com> 1548055516 +0800
  
  init commit
  
  Signed-off-by: Huabing Zhao <zhaohuabing@gmail.com>
  ```

  可以看出这是一个commit对象，commit对象中保存了commit的作者，commit的描述信息，签名信息以及该commit中包含哪些tree对象和blob对象。
  
  b767d7这个commit中保存了一个tree对象，可以把该tree对象看成这次提交相关的所有文件的根目录。让我们来看看该tree对象中的内容。
  
  ```
  $ git cat-file -p ca964f
  100644 blob 065bcad11008c5e958ff743f2445551e05561f59    README
  040000 tree 82424451ac502bd69712561a524e2d97fd932c69    src
  ```
  
  可以看到该tree对象中包含了一个blob对象，即README文件；和一个tree对象，即src目录。 分别查看该blob对象和tree对象，其内容如下：
  
  ```
  $ git cat-file -p 065bca
  my project
  $ git cat-file -p 824244
  100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    file1.txt
  ```
  
  查看file1.txt的内容。
  
  ```
  $ git cat-file -p 3b18e51
  hello world
  ```
  
  
  从上面的实验我们可以得知，git中存储了三种类型的对象，commit，tree和blob。分别对应git commit，此commit中的目录和文件。这些对象之间的关系如下图所示。
  
  ```
  HEAD---> refs/heads/master--> b767d7(commit)
                                      +
                                      |
                                      v
                                  ca964f(tree)
                                      +
                                      |
                            +---------+----------+
                            |                    |
                            v                    v
                       065bca(blob)         824244(tree)
                            README              src
                                                 +
                                                 |
                                                 v
                                            3b18e5(blob)
                                               file1.txt    
  ```
  
  从refs/heads/master的内容可以看到，branch是一个指向commit的指针，master branch实际是指向了b767d7这个commit。
  
  ```
  $ git checkout -b work
  Switched to a new branch 'work'
  $ tree .git/refs/
  .git/refs/
  ├── heads
  │   ├── master
  │   └── work
  └── tags
  $ cat .git/refs/heads/work .git/refs/heads/master
  b767d7115ef57666c9d279c7acc955f86f298a8d
  b767d7115ef57666c9d279c7acc955f86f298a8d
  ```
  
  
  上面的命令创建了一个work branch。从其内容可以看到，该branch并没有创建任何新的版本文件，和master一样指向了b767d7这个commit。
  
  从上面的实验可以看出，一个branch其实只是一个commit对象的应用，Git并不会为每个branch存储一份拷贝，因此在git中创建branch几乎没有任何代价。
  
  Git会为每次commit时修改的目录/文件生成一个新的版本的tree/blob对象，如果文件没有修改，则会指向老版本的tree/blob对象。而branch则只是指向某一个commit的一个指针。即Git中整个工作目录的version是以commit对象的形式存在的，可以认为一个commit就是一个version，而不同version可以指向相同或者不同的tree和blob对象，对应到不同版本的子目录和文件。如果某一个子目录/文件在版本间没有变化，则不会为该子目录/文件生成新的tree/blob对象，不同version的commit对象会指向同一个tree/object对象。
  
  Tag和branch类似，也是指向某个commit的指针。不同的是tag创建后其指向的commit不能变化，而branch创建后，其指针会在提交新的commit后向前移动。