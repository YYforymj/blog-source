---
title: GIT03
date: 2019-04-10 16:27:00
categories: GIT
---

> 以下内容为GIT的学习笔记，教程内容来自 [网络博客](http://www.cnblogs.com/cnblogsfans/p/5075073.html)与[GitHub Guide](https://guides.github.com/activities/forking/)。

<!-- more -->

## gitflow工作流
* **Gitflow工作流**：gitflow工作流起源于[这里](https://nvie.com/posts/a-successful-git-branching-model/)，是一种非常清晰的流程规范，通常在私有的项目中应用。大体的流程如下：


![](\images\GIT03\git-flow-origin.png)

- **Git Flow常用的分支**：

  - Production 分支

  也就是我们经常使用的Master分支，这个分支最近发布到生产环境的代码，最近发布的Release， 这个分支只能从其他分支合并，不能在这个分支直接修改。

  - Develop 分支

  这个分支是我们是我们的主开发分支，包含所有要发布到下一个Release的代码，这个主要合并与其他分支，比如Feature分支。

  - Feature 分支

  这个分支主要是用来开发一个新的功能，一旦开发完成，我们合并回Develop分支进入下一个Release。

  - Release分支

  当你需要一个发布一个新Release的时候，我们基于Develop分支创建一个Release分支，完成Release后，我们合并到Master和Develop分支。

  - Hotfix分支

  当我们在Production发现新的Bug时候，我们需要创建一个Hotfix, 完成Hotfix后，我们合并回Master和Develop分支，所以Hotfix的改动会进入下一个Release。

- **Git Flow工作流程解析**：

  - 初始分支

    ![](\images\GIT03\o_git-workflow-release-cycle-1historical.png)

  - Feature 分支：

    当需要开发新的功能时，从`develop`分支的最新commit创建新的分支，并以`feature/*`的格式命名。当该分支开发完成后再合并回`develop`分支，合并后`feature` 分支可以保留以也可以删除。

    ![](\images\GIT03\o_git-workflow-release-cycle-2feature.png)

  - Release分支：

    分支名 `release/*`。Release分支基于Develop分支创建，打完Release分支后，我们可以在这个Release分支上测试，修改Bug等。同时，其它开发人员可以基于开发新的Feature (记住：一旦打了Release分支之后不要从Develop分支上合并新的改动到Release分支)。发布Release分支时，合并Release到Master和Develop， 同时在Master分支上打个Tag记住Release版本号，然后可以删除Release分支了。

    ![](\images\GIT03\o_git-workflow-release-cycle-3release.png)

  - 维护分支 Hotfix：

    分支名 `hotfix/*`。hotfix分支基于Master分支创建，开发完后需要合并回Master和Develop分支，同时在Master上打一个tag。

    ![](\images\GIT03\o_git-workflow-release-cycle-4maintenance.png)

    以上便是gitflow的大致工作流程。

- **工具**：如果仅仅使用命令行处理工作流会稍显麻烦，所以现在有很多的辅助工具来实现gitflow工作流
  - SourceTree：可以从[官网](https://www.sourcetreeapp.com/)下载，同时在使用过程中可能会需要翻墙。大部分的git与gitflow功能都可以通过点点鼠标来实现，少部分功能仍需使用命令行处理。
  - IDEA plugin：Git Flow Integration，可以方便地与IDEA集成使用gitflow。

## GitHub工作流（Fork工作流）

- **GitHub工作流**：GitHub工作流（有时也成为fork工作流），是GitHub上针对开源项目的工作流程，最大的优势是可以处理不信任贡献者（`contributor`）的提交，所以十分适合于开源项目。
- **GitHub工作流程分析**：如果开发者想要为某个项目修改代码或添加特性，可以：
  1. 首先`Fork`该仓库到自己的仓库中，此时fork后的仓库中存有一份原仓库代码的拷贝；
  2. 开发者将fork后的仓库 `clone` 或者 `download` 到本地，然后在本地的仓库中进行修改，直至认为新添加的代码可以合并到原仓库后推送到自己仓库中 fork 出来的拷贝；
  3. 如果想要将修改的代码合并到原仓库，需要使用 GitHub 的 `Pull Request`功能，向原仓库发起合并请求；
  4. 原仓库的作者或者维护人员在发现 `Pull Request` 后进行代码审阅，如果代码可以通过审阅则由维护人员将代码合并至原仓库，从而实现对项目的修改。 